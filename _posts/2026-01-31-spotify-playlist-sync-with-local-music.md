---
layout: post
title: "Syncing Spotify Playlists to Your Local Music Library with Python (and fuzzy matching)"
categories: [python, spotify, mpd, ncmpcpp, music]
---

I like to think I have a diverse and extensive music taste (my wife disagrees).

Either way, I have a local music library that is big enough to make manual playlist management painful:
```bash
mpc update > /dev/null 2>&1 && mpc stats | grep -E '(Artists|Albums|Songs)'

Artists:    871
Albums:    1021
Songs:     4803
```

I use `ncmpcpp` as an mpd client. However, I am also a regular user of Spotify. I wanted a way to:

1. Initially archive my Spotify playlists.
2. If possible, match them to any files that I have locally.

The goal: *take Spotify playlists → match them against my local library → load them into MPD/ncmpcpp.*

With the help of ChatGPT to get fuzzy-matching working, I found a more automated way to do this with python.

This post requires the python [dependencies](https://github.com/sdysch/export-spotify-playlists/blob/main/pyproject.toml):
```
    "eyed3>=0.9.9",
    "mutagen>=1.47.0",
    "pandas>=3.0.0",
    "pyyaml>=6.0.3",
    "rapidfuzz>=3.14.3",
    "spotipy>=2.25.2",
```

---

# 1. Export Spotify playlists

Spotify has an extensive [API](https://developer.spotify.com/documentation/web-api), and I used the [spotipy](https://spotipy.readthedocs.io/en/2.25.2/)
(a python library for interacting with the API) to initially archive my playlists to csv files:

In pseudo-python code:

```python
import spotipy
from spotipy.oauth2 import SpotifyOAuth
import pandas as pd

sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
    client_id='YOUR_ID',
    client_secret='YOUR_SECRET',
    redirect_uri='YOUR_REDIRECT_URI',
    scope='playlist-read-private playlist-read-collaborative'
))

playlists = sp.current_user_playlists(limit=50)

for pl in playlists['items']:
    tracks = sp.playlist_items(pl['id'], limit=100)
    rows = []
    for item in tracks['items']:
        track = item['track']
        rows.append({
            'id': track['id'],
            'name': track['name'],
            'artist': track['artists'][0]['name']
        })

    df = pd.DataFrame(rows)
    df.to_csv(f'data/playlists/{pl["name"].replace("/", "-")}.csv', index=False)

```

---

# 2. Index the local music library
My music is saved under `~/Music/{source1, source2}` etc., with the common formats under each sub-directory as `Artist/Album/Songs`.

We scan everything, including possible audio formats (again python pseudocode):
```python
from pathlib import Path
import mutagen

AUDIO_EXTS = ['.mp3', '.flac', '.m4a', '.aac', '.wav']

library = []

for path in Path('~/Music').expanduser().rglob('*'):
    if path.suffix.lower() not in AUDIO_EXTS:
        continue

    audio = mutagen.File(path, easy=True)
    title = audio.get('title', [path.stem])[0] if audio else path.stem
    artist = audio.get('artist', [path.parent.parent.name])[0] if audio else path.parent.parent.name

    library.append({
        'title': title.lower(),
        'artist': artist.lower(),
        'path': str(path)
    })
```

[mutagen](https://mutagen.readthedocs.io/en/latest/) is a python library to handle audio metadata.

I use file metadata as the first possible source, otherwise the folder structure is the fallback.

---

# 3. The need for fuzzy matching
Exact string matches will fail here. I am not too careful with my tags[^1]. For example, I have many legitimate album variants locally like:
```
A Flash Flood of Colour
A Flash Flood of Colour (Deluxe Version)
```
which may not be the exact version I included in a Spotify playlist.
There are also singles vs album versions of songs. The list goes on.

Fuzzy matching will match _almost_ close cases, with a score based on the changes between two strings. E.g.

```
kitten → sitting = 3 changes
```

```
bat → cat = 1 change
```

---

# 4. Fuzzy matching with RapidFuzz
With the help of ChatGPT, I got a simple fuzzy matching on track and artist working:
```python
from rapidfuzz import fuzz

def match(track, artist, library, threshold=80):
    track = track.lower()
    artist = artist.lower()

    best = None
    best_score = 0

    for song in library:
        score = (fuzz.ratio(track, song['title']) +
                 fuzz.ratio(artist, song['artist'])) / 2
        if score > best_score:
            best_score = score
            best = song

    return best if best_score >= threshold else None
```
This will automatically handle:
* punctuation differences
* capitalisation
* small typos
* inconsistencies in naming

---

# 5. Writing mpd playlists
Once the playlist matching is done, it is very easy to convert the playlists into the `m3u` format for mpd:
```python
with open('~/Music/playlists/my_playlist.m3u', 'w', encoding='utf-8') as f:
    for track in matched_tracks:
        f.write(track['path'] + '\n')
```

---

# 6. Runtime and improvements
With nearly 5000 tracks:
* fuzzy matching is very fast and took a few minuts at most to run
* false positives did happen, but rarely (the bigger unavoidable problem was me noot having the track locally)
* I could include duration matching, but this is almost certainly overkill

And yes, even with 871 artists, my wife still says I always listen to blink-182. She's not wrong.

The code I used for this work is available on [github](https://github.com/sdysch/export-spotify-playlists)


[^1]:And honestly, my collection dates back to the early 2000s, when I was still using iTunes (_shudder_), and a lot of the tags got creative.
