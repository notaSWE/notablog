---
title: "Detecting Acropalypse with Go Because Why Not"
date: 2023-03-25
---
From what I can tell [Simon Aarons](https://twitter.com/ItsSimonTime/status/1636857478263750656) and [David Buchanan](https://www.da.vidbuchanan.co.uk/blog/exploiting-acropalypse.html) did the heavy lifting on this.  But I never let re-inventing the wheel keep me from learning something new.

# Acropalypse

[CVE-2023-21036](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-21036), henceforth referred to as Acropalypse, is a flaw in the cropping mechanism on Android phones that can allow an attacker to see thought-to-be-truncated "cropped" data from an image during the `screenshot -> crop -> save` process.  This is not to be confused with a similar vulnerability in the Windows 10/11 Snipping Tool/Skip & Sketch, but it is quite similar.

[![Simon Aarons Tweet](/notablog/docs/assets/2023_03_27_tweet.JPG "Informative Tweet")](https://twitter.com/ItsSimonTime/status/1636857478263750656)

As part of my research I came across [intobyte's](https://github.com/infobyte/CVE-2023-21036) tool to detect vulnerable images using Python.  As normal, well-adjusted individuals like myself tend to do, I began testing it to see if I could extend functionality.  Rather than using a bash for loop I created a new Python file called `acropalypse_dir.py` and modified it so I could scan an entire directory of relevant `.jpg` and `.png` images:

```python
images_to_check = []
directory_path = sys.argv[1]

# Check if the directory exists
if not os.path.isdir(directory_path):
  print(f"Error: {directory_path} is not a valid directory")
  sys.exit(1)

# Iterate through all files in the directory
for filename in os.listdir(directory_path):
  # Check if the file is a .jpg/.png file
  if fnmatch.fnmatch(filename, "*.png") or fnmatch.fnmatch(filename, "*.jpg"):
    file_path = os.path.join(directory_path, filename)
    images_to_check.append(file_path)
```

Of course, I had to modify the return statements in the functions provided in the infobyte `acropalypse_detection.py` and then loop through my `images_to_check` like so:

```python
if images_to_check:
  for image in images_to_check:
    f_in = open(image, "rb")
    start = f_in.read(2)
    f_in.seek(0,0)

    if start == b"\x89P":
      try:
        parse_png(f_in)
      except:
        continue
    elif start == b"\xFF\xD8":
      try:
        parse_jpeg(f_in)
      except:
        continue
    else:
      print("File doesn't appear to be jpeg or png.")
else:
  print("No images to check; quitting.")
  sys.exit(1)
```

Now at this point I thought to myself: I wonder if I could scan all files on my computer for the above issue...in Go.

![Hmm...](/notablog/docs/assets/2023_03_27_think.gif "Thinking Emoji")

# Gocropalypse

First things first, I needed to figure out how to scan a directory recursively for .jpg/.png files.  Thankfully Go documentation is quite nice and I quickly found [os.ReadDir](https://pkg.go.dev/os#ReadDir).  I needed to not only check to see if files had the right extension, but also needed to check to see if the corresponding [Magic Numbers](https://en.wikipedia.org/wiki/List_of_file_signatures) were found.  PNG has a magic number of `\x89PNG\r\n\x1a\n` so I started with that:

```golang
package main

import (
  "fmt"
  "log"
  "os"
  "strings"
)

// PNG Magic Number
const PNG_MAGIC = "\x89PNG\r\n\x1a\n"

func main() {
  files, err := os.ReadDir(".")
  if err != nil {
    log.Fatal(err)
  }

  for _, file := range files {
    if strings.Contains(strings.ToLower(file.Name()), ".png") {
      f, err := os.Open(file.Name())
      if err == nil {
        buffer := make([]byte, len(PNG_MAGIC))
        _, err = f.Read(buffer)
        if string(buffer) == PNG_MAGIC {
          fmt.Println(file.Name())
        }
      }
      defer f.Close()
    }
  }
}
```

Unfortunately, the above is quite limited as it only scans the directory `gocropalypse.go` resides in.  I wanted to take in a directory to scan recursively as a command-line argument:

```golang
args := os.Args[1:]

if len(args) < 1 {
  fmt.Println("go run gocropalypse.go /path/to/dir")
  return
}

directory := args[0]
```

Additionally, it made sense to create a function called `isConfirmedPngFile` which would take in the file string and return a boolean value.  We couild then append confirmed .png files via another function `appendConfirmedPngFiles`:

```golang
func isConfirmedPngFile(file string) bool {
  if strings.Contains(strings.ToLower(file), ".png") {
    f, err := os.Open(file)
    if err == nil {
      buffer := make([]byte, len(PNG_MAGIC))
      _, err = f.Read(buffer)
      f.Close()
      if string(buffer) == PNG_MAGIC {
        return true
      }
    }
  }
  return false
}

func appendConfirmedPngFiles(path string, confirmedPngFiles *[]string) error {
  files, err := os.ReadDir(path)
  if err != nil {
    return err
  }

  for _, file := range files {
    fullPath := filepath.Join(path, file.Name())
    if file.IsDir() {
      err = appendConfirmedPngFiles(fullPath, confirmedPngFiles)
      if err != nil {
        return err
      }
    } else {
      if isConfirmedPngFile(fullPath) {
        *confirmedPngFiles = append(*confirmedPngFiles, fullPath)
      }
    }
  }

  return nil
}
```

I could then iterate through the list of `confirmedPngFiles` and pass them into a new function called `checkPngForVuln`.  Of course, checking for the vuln is actually kind of hard, so I relied on ChatGPT to help me better understand the Python code in the infobyte tool.  I'm not going to post the verbose interaction with GPT-4 since I'd hate to get sued by an AI model this early in the singularity, but long story short, working code to detect and report potentially vulnerable .jpg and .png files can be found on my Github.  I also re-worked my logic a bit to make sure the Go and Python versions were scanning the same files; more on that in the next section.  

# Performance
After some additional transcription of the logic in infobyte's `acropalypse_detection.py` I was at a point where both their version and my version could recursively search for .png and .jpg files in a directory taken in as a command-line argument.  That said I thought it would be fun to test performance between Go and Python.  I added timers to each script and ran them; results below:

## Go
```bash
$ go run gocropalypse.go /mnt/e/notaswe
Found 197 vulnerable images out of a scanned total of 5958.
Total time to execute: 66.41605 seconds
```

## Python
```bash
$ python3 acropalypse_dir.py /mnt/e/notaswe
Found 197 vulnerable images out of a scanned total of 5958.
Total time to execute: 69.364916 seconds
```

# Conclusion

Well I guess it's official, folks.

![Bye bye](/notablog/docs/assets/2023_03_27_conclusion.gif "Deleting Python")