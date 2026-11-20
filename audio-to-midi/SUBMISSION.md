# n8n Template — Audio to MIDI (Convert New Drive Audio Automatically)

**Status:** built and live-tested locally 2026-11-21 (see
TEST_EVIDENCE.md). Package ready for Hamzah to submit to n8n
himself — this package is the deliverable, nothing has been
submitted yet. Built at Hamzah's ask as the companion to his
Rogue LLM tutorial video.

**Actor:** magnova/audio-to-midi-converter (`z5Fhn3SdiTaU5ljpb`) —
$0.01 per minute of audio converted (Silver $0.0085 / Gold+
$0.007), no start fee. Files that fail or contain no detectable
notes are not charged.

## What it does

The workflow watches a Google Drive folder. Every new audio
file is checked (audio only — anything else is ignored),
shared with a public link so the converter can fetch it, and
transcribed on Apify by the Audio to MIDI Converter actor
(Spotify's Basic Pitch model). The finished .mid file is
downloaded, saved to an output Drive folder, logged to a
Google Sheet (source file, date, note count, key, tempo, Drive
link), and announced with a Telegram message. Failed or
noteless files are skipped quietly and never charged.

This mirrors the gallery-approved Make template "Convert audio
to MIDI and save it to Google Drive" (template 20355) and adds
the Sheet log + Telegram announcement.

## Files

- `workflow.json` — the submission workflow (18 nodes: 13
  functional + 5 sticky notes, real Drive/Sheets/Telegram
  nodes, no credentials).
- `README.md` — importer-facing setup guide.
- `tests/build_test_files.py` — builds the local test copy.
- `TEST_EVIDENCE.md` — the live-test record (filter, loop,
  3 real actor runs, MIDI byte verification, charges).

## Submission checklist (per the house n8n guidelines)

- [x] Yellow overview sticky note (top-left, 246 words,
      How it works / Setup / Customization)
- [x] Three white section notes, each ≤50 words, covering
      their node groups
- [x] One red warning note, over the Share node: sharing sets
      the audio file to anyone-with-the-link
- [x] Every node name descriptive (verb + service/data)
- [x] Live-tested end-to-end locally with the rose_firehouse
      test account before packaging (Google/Telegram legs
      substituted — no Google OAuth app or Telegram bot on
      this VM; see TEST_EVIDENCE.md honest limits)
