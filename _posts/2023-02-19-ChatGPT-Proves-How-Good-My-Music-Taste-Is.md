---
title: "ChatGPT Proves How Good My Music Taste Is"
date: 2023-02-19
---
![Wall of Records Demo](/notablog/docs/assets/2023_02_19_albums01.JPG "Wall of Records Demo")

What you see above is the result of entirely too many hours interacting with ChatGPT to display my nostalgia-based vinyl collection.  

You might (not) be asking yourself, how'd you accomplish such a feat?  Simple, really.  I have entirely too much free time.  But it does involve an interesting tech stack, so if you're into that kind of thing, read on!

### Enter Discogs

Discogs is basically the Untappd of the vinyl world.  Meaning, you can add a record to your "collection" and share a link with your friends to compare records, trade records, etc.

Long story short, the data just kind of exists.  I thought to myself, wouldn't it be cool if I could query that data somehow, and maybe display it in an interesting way?

### Enter ChatGPT

Full disclosure, I am pretty bad at reading API documentation most of the time.  ChatGPT basically lets me skip that hastle.  So the first thing I looked up was:

![Discogs to JSON Query](/notablog/docs/assets/2023_02_19_question01.JPG "Discogs to JSON Query")

```
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

```
# Extract album thumbnails from each release
thumbnails = []
for release in response.json()['releases']:
    # Check if release has an image and add it to the list of thumbnails
    if release['basic_information']['thumb']:
        thumbnails.append(release['basic_information']['thumb'])
```

After a bit of `jq` inspection of `my_collection.json` I found that the preferred thumbnail was actually `release['basic_information']['cover_image']` which was a simple enough edit.  If you read on you will notice that not all of ChatGPT's results are direct answers, but they get you *close enough*.  Which brings me to an issue with the above code; pagination.

![Pagination Issue Query](/notablog/docs/assets/2023_02_19_question03.JPG "Pagination Issue Query")

```
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

```
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

```
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