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

![Discogs to JSON Query](notablog/docs/assets/2023_02_19_question01.JPG "Discogs to JSON Query")

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