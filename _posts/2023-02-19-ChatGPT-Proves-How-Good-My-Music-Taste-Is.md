---
title: "ChatGPT Proves How Good My Music Taste Is"
date: 2023-02-19
---
![Wall of Records Demo](/notablog/docs/assets/2023_02_19_albums01.JPG "Wall of Records Demo")

What you see above is the result of entirely too many hours interacting with ChatGPT to display my nostalgia-based vinyl collection.  

You might (not) be asking yourself, how'd you accomplish such a feat?  Simple, really.  I have entirely too much free time.  But it does involve an interesting tech stack, so if you're into that kind of thing, read on!

# Enter Discogs

Discogs is basically the Untappd of the vinyl world.  Meaning, you can add a record to your "collection" and share a link with your friends to compare records, trade records, etc.

Long story short, the data just kind of exists.  I thought to myself, wouldn't it be cool if I could query that data somehow, and maybe display it in an interesting way?

# Enter ChatGPT

Full disclosure, I am pretty bad at reading API documentation most of the time.  ChatGPT basically lets me skip that hastle.  So the first thing I looked up was:

![Discogs to JSON Query](/notablog/docs/assets/2023_02_19_question01.JPG "Discogs to JSON Query")

```python
import requests
import json

# Your Discogs username and API key
username = 'your_username'
api_key = 'your_api_key'

# The ID of the folder containing your collection
folder_id = 0

# Base URL for Discogs API
base_url = 'https://api.discogs.com'

# Endpoint for retrieving collection releases
endpoint = f'/users/{username}/collection/folders/{folder_id}/releases'

# Set up authentication headers
headers = {'User-Agent': 'MyDiscogsClient/1.0', 'Authorization': f'Discogs token={api_key}'}

# Make request to API
response = requests.get(base_url + endpoint, headers=headers)

# Convert response to JSON and output to file
with open('my_collection.json', 'w') as f:
    json.dump(response.json(), f)
```

I simply saved that locally as `get_collection.py`, created an API key in my Discogs account, and hardcoded my credentials without a care in the world.  I then executed the script and verified the output to the newly created `my_collection.json` file.  So far, so good!

I needed to parse out the actual URLs of the albums I own so I could then save them and arrange them in a nice grid for the world to see.  That said, ChatGPT is rather verbose in its output so I will just be pasting in the relevant code snippets from here on out.

![Parse Thumbnails Query](/notablog/docs/assets/2023_02_19_question02.JPG "Parse Thumbnails Query")

```python
# Extract album thumbnails from each release
thumbnails = []
for release in response.json()['releases']:
    # Check if release has an image and add it to the list of thumbnails
    if release['basic_information']['thumb']:
        thumbnails.append(release['basic_information']['thumb'])
```

After a bit of `jq` inspection of `my_collection.json` I found that the preferred thumbnail was actually `release['basic_information']['cover_image']` which was a simple enough edit.  If you read on you will notice that not all of ChatGPT's results are direct answers, but they get you *close enough*.  Which brings me to an issue with the above code; pagination.

![Pagination Issue Query](/notablog/docs/assets/2023_02_19_question03.JPG "Pagination Issue Query")

```python
# Make request to API and retrieve all pages of results
page = 1
thumbnails = []
while True:
    response = requests.get(base_url + endpoint, headers=headers, params={'page': page})
    response_data = response.json()
    # Extract album thumbnails from each release
    for release in response_data['releases']:
        # Check if release has an image and add it to the list of thumbnails
        if release['basic_information']['cover_image']:
            thumbnails.append(release['basic_information']['cover_image'])
    # Check if there are more pages of results
    if 'next' in response.links:
        page += 1
    else:
        break
```

The above code resolved the pagination issue, and now I could retrieve all of my collection (79 records at the time of writing).  Now that I had a list of thumbnails called `thumbnails` it was trivial to iterate through them and perform a `wget` operation on each.

![Wget Files Query](/notablog/docs/assets/2023_02_19_question04.JPG "Wget Files Query")

```python
import os
import subprocess

# Set the URL and file name for the file you want to download
url = 'https://example.com/myfile.txt'
filename = 'myfile.txt'

# Set the directory where you want to check for the file
directory = '/path/to/directory'

# Check if the file exists in the directory
file_path = os.path.join(directory, filename)
if os.path.isfile(file_path):
    print(f"{filename} already exists in {directory}")
else:
    # Download the file using wget
    print(f"Downloading {filename} from {url} to {directory}")
    subprocess.run(['wget', '-P', directory, url])
```

With a bit of Frankenstein code with the above snippet and what we saw previously, I was able to come up with this:

```python
for thumbnail in thumbnails:
    filename = thumbnail.split("/")[-1]
    file_path = f"{directory}{filename}"
    if os.path.isfile(file_path):
        print(f"{filename} already exists in {directory}")
    else:
        # Download the file using wget
        print(f"Downloading {filename} from {thumbnail} to {directory}")
        subprocess.run(['wget', '-P', directory, thumbnail])
```

This resulted in a collection of 78 photos in my `thumbs/` directory.  Interestingly, this was one less than my total collection... I believe a single record was missing the required `release['basic_information']['cover_image']`) and thus no thumbnail was downloaded.  

Now was the time to arrange everything in a nice grid using some semblance of html/javascript/css.  My skills in front end development are nonexistent, so...

## Enter More ChatGPT

![Front End HTML/CSS Query](/notablog/docs/assets/2023_02_19_question05.JPG "Front End HTML/CSS Query")

```php
<!DOCTYPE html>
<html>
<head>
  <title>Image Grid Example</title>
  <style>
    .grid-container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      grid-gap: 10px;
    }
  </style>
</head>
<body>
  <div class="grid-container">
  <?php
  $dir = 'path/to/images/';
  $files = scandir($dir);
  foreach ($files as $file) {
    if (is_file($dir . $file)) {
      echo '<img src="' . $dir . $file . '">';
    }
  }
  ?>
  </div>
</body>
</html>
```

Oh no, PHP.  If there's anything I like worse than JavaScript, it's PHP.

![No PHP, Please](/notablog/docs/assets/2023_02_19_question06.JPG "No PHP, Please")

In order to prevent this post from getting too long, I had quite a bit of back and forth with ChatGPT from here on out in terms of the front end design.  Here are a few of my questions before we cut to the final layout:

![Dynamic Layout Query](/notablog/docs/assets/2023_02_19_question07.JPG "Dynamic Layout Query")

![Aspect Ratio Query](/notablog/docs/assets/2023_02_19_question08.JPG "Aspect Ratio Query")

![On Mouseover Query](/notablog/docs/assets/2023_02_19_question09.JPG "On Mouseover Query")

![Maintain Image Size Query](/notablog/docs/assets/2023_02_19_question10.JPG "Maintain Image Size Query")

After modifying my `index.html` file I placed it in my project directory and launched a simple python web server: `python3 -m http.server`

I could then load `127.0.0.1:8000/index.html` in my browser, greeted with my incredible record collection in its full glory:

![Layout GIF](/notablog/docs/assets/2023_02_19_index_gif.gif "Layout GIF")

Now came the part in my project to share the above with the world.  Thankfully I had a barren wasteland of an AWS account at the ready...

## Enter AWS

Without going into insane detail on the infrastructure, I knew that this could be something like:

1. A Lambda function that would:
    * Grab my collection data using the Discogs API
    * Read an S3 bucket subdirectory called `thumbs/` and check for missing thumbnails
    * Download missing thumbnails to `/tmp/` and then upload them to the S3 bucket `thumbs/`
2. The S3 bucket mentioned above would host `index.html`
3. The trigger for the lambda function would a CloudWatch Event running on a cron schedule
4. I wanted to share the above buildout with the world so it all needed to be in terraform

That said, I used ChatGPT for all of it...and it works!  Thanks for reading 😎

[Check it out on my GitHub](https://github.com/notaSWE/wallofrecords)

[Live demo here, for now](https://album-photos-gvqelm.s3.amazonaws.com/index.html)