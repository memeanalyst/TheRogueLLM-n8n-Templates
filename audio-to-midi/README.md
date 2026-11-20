# Audio to MIDI — Convert New Drive Audio Automatically (n8n template)

Drop an audio file into a Google Drive folder and get a MIDI
file back — transcribed automatically, saved to Drive, logged
to a spreadsheet, and announced on Telegram.

The transcription runs on the
[Audio to MIDI Converter](https://apify.com/magnova/audio-to-midi-converter)
actor on Apify (Spotify's Basic Pitch model). It works best on
solo instruments and vocals — piano, guitar, violin, flute,
singing, humming.

## How it works

1. **Watch** — the trigger polls your inbox folder for new
   files.
2. **Filter** — only audio files pass (MP3, WAV, M4A, FLAC,
   OGG, OPUS, AIFF…). Anything else is ignored.
3. **Convert** — each file is shared with a public link and
   sent to the converter on Apify, one at a time.
4. **Deliver** — the finished `.mid` is saved to your output
   folder, a row is appended to the log sheet (source file,
   date, note count, key, tempo, Drive link), and a Telegram
   message announces it.

Files that fail to download or contain no detectable notes
are skipped — the converter does not charge for them.

## Setup

You need: an **Apify account**, a **Google account**, and
(optionally) a **Telegram bot**.

1. Import `workflow.json` into n8n.
2. **Apify node** ("Convert audio to MIDI on Apify"): connect
   your own Apify credential. The converter costs **$0.01 per
   minute of audio**, billed to your Apify account; the node
   is capped at $2 per file by default.
3. **Trigger** ("Watch Drive folder for new audio"): connect
   your Google account and pick the inbox folder to watch.
4. **Upload** ("Save MIDI to Drive folder"): pick the output
   folder for finished MIDIs.
5. **Sheet** ("Log conversion to Sheet"): create a Google
   Sheet with a tab named **Conversions** and these headers,
   then point the node at it:
   `sourceFile`, `convertedAt`, `noteCount`, `musicalKey`,
   `tempoBpm`, `midiLink`
6. **Telegram** ("Send MIDI ready message to Telegram"): add
   your bot credential and chat ID — or delete this node if
   you don't want messages.
7. Activate the workflow.

## ⚠️ One thing to know

The "Share audio file in Drive" step sets each audio file to
**anyone with the link** so the converter can fetch it. Use a
dedicated inbox folder for this workflow, not a folder of
private recordings.

## Customization

- **Video files too:** the converter also accepts MP4/MOV/
  WEBM — widen the "Is it an audio file?" filter to let them
  through.
- **Long recordings:** raise the Apify node's
  `maxTotalChargeUsd` (default $2 ≈ 200 minutes of audio).
- **Different notifications:** swap the Telegram node for
  Slack or email — the message text is built in the step
  before it.
- **Transcription tuning:** the actor accepts onset/frame
  thresholds and frequency limits; add them to the Apify
  node's custom body if you convert a lot of one instrument.
