# Podtips

Podtips finds movie and music tips in podcasts. It writes one Markdown file per episode and pushes the files to this GitHub repo. You can then read the tips on a computer or a phone.

## How it works

1. The script reads the RSS feed of each podcast.
2. It gets a transcript. It uses the transcript from the feed if there is one. If not, it downloads the audio and uses Whisper on the Mac GPU.
3. It sends the show notes and the transcript to Claude with the `claude` command. Claude returns the tips.
4. It writes one file for the episode.
5. It commits and pushes the result.

The audio file is deleted after transcription. The transcript stays in `transcripts/`, so a new extraction does not need Whisper again.

## Run it

Run all commands from the project folder.

Add new episodes:

```
.venv/bin/python podtips.py
```

A normal run only takes episodes from 31 October 2026 or later. Change `START_DATE` in `podtips.py` to use another date.

Add an older episode with a word from its title:

```
.venv/bin/python podtips.py --only "Carrie"
```

Add an episode with a pabloheld.com link:

```
.venv/bin/python podtips.py --only "https://pabloheld.com/en/investigates/bill-frisell"
```

If a word matches several episodes, the script processes all of them. The script skips episodes that are already in `done.json`.

To process an episode again, remove its entry from `done.json` and run the script with `--only`. Then the script overwrites the episode file.

## Podcasts

| Podcast | Language | New episodes |
|---|---|---|
| Bio min Bio podden | Swedish | Automatic |
| You'll Hear It | English | Automatic |
| Pablo Held Investigates | English | Only when you name the episode |

To add a podcast, add an entry to `FEEDS` in `podtips.py`. Set `"only_on_request": True` if you want to select the episodes yourself.

## The tips

Each tip has a title, a creator, a year, a short comment and search links. Movies and series link to IMDb and Rotten Tomatoes. Albums and songs link to Spotify. Most tips also have a time in the episode.

- Comments use the language of the podcast.
- Movies and series use the English release title if there is one.
- "Where" tells where the hosts said that you can see the movie. This is true for the episode date and can change later.
- Movies that the hosts review but do not like stay in the list, with a remark.

The rules for what counts as a tip are in `PROMPT` in `podtips.py`.

## Files

| Path | Content | In git |
|---|---|---|
| `podtips.py` | The script | Yes |
| `done.json` | The finished episodes | Yes |
| `<podcast>/<date>-<title>.md` | One file per episode | Yes |
| `transcripts/` | Saved transcripts | No |
| `audio/` | Audio during transcription | No |
| `models/` | Whisper models | No |
| `.venv/` | Python environment | No |

Each episode file starts with a data block with the podcast, title, date and link. Website generators can read this block, so the files are ready for a web page later.

## Setup on a new Mac

You need an Apple Silicon Mac, ffmpeg, `uv`, and the `claude` command with a login. The `git` remote must also be set up. Then run:

```
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python mlx-whisper
.venv/bin/python -c "
from huggingface_hub import snapshot_download
snapshot_download('jegeblad/kb-whisper-large-mlx-q4', local_dir='models/kb-whisper-large-mlx-q4')
snapshot_download('mlx-community/whisper-large-v3-turbo', local_dir='models/whisper-large-v3-turbo')"
```

The Swedish model is about 880 MB, and the English model is about 1.6 GB.

## Limits

- Whisper takes about 5 minutes per hour of audio.
- Names and titles in the tips can have errors. A title that is wrong gives a poor search result.
- The search links are searches, not direct links to the movie or album.
- The list of tips can change between two runs on the same episode.
- The script treats all Pablo Held episodes as English.
