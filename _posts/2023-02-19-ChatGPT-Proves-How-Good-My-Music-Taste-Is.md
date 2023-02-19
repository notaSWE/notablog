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

Now that I had a list of thumbnails called `thumbnails` it was trivial to iterate through them and perform a `wget` operation on each.