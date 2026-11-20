# Test Evidence — Audio to MIDI (n8n)

**Date:** 2026-11-21 · **Instance:** local n8n 2.47.7
(`~/workspace/tools/n8n`, DB `~/.n8n/database.sqlite`) ·
**Apify package:** `@apify/n8n-nodes-apify` v0.9.0 ·
**Credential:** `apifytestcred1` (rose_firehouse TEST account —
every actor run below ran on the test account, never Magnova) ·
**Test workflow in the instance:** `audioMidiTest1`
("Audio to MIDI — Convert New Drive Audio Automatically",
inactive)

## What was substituted locally, and why

The local n8n has **no Google OAuth app and no Telegram bot**
(verified: the only credential in the instance is
`apifytestcred1`), so the Google and Telegram legs cannot run
here — the same known wall as the Fare Watch / Slogan Shield
tests. The test copy (builder: `tests/build_test_files.py`)
replaces exactly those legs:

| Submission workflow | Local test copy |
|---|---|
| Google Drive Trigger "Watch Drive folder for new audio" | Manual Trigger + Code node "Test Drive files" returning 4 fake Drive files (below) |
| Google Drive "Share audio file in Drive" | Passthrough Code node |
| Set "Build Apify audio URL" | Code node mapping the fake Drive IDs to real public audio URLs (identical output shape: sourceUrl, driveFileId, driveFileName) |
| Google Drive "Save MIDI to Drive folder" | Code node returning Drive-like output — file name read from the Prepare step's paired item (the same mechanism the submission's Drive node uses for its name expression), fixed fake webViewLink |
| Google Sheets "Log conversion to Sheet" | Passthrough Code node, so log rows land in the execution output |
| Telegram "Send MIDI ready message to Telegram" | Passthrough Code node, so the built message texts land in the execution output — **stubbed, not sent** (no Telegram credential exists locally) |

Everything else is the real thing and ran for real: the audio
IF filter, the Loop Over Items node, the Apify node (real runs
of magnova/audio-to-midi-converter, ID `z5Fhn3SdiTaU5ljpb`, on
the test account), the Prepare Code node, the success IF, the
**real HTTP download of the produced MIDI files**, and the
log-row/message Code node — **byte-identical to the ones in the
submitted `workflow.json`**. The submitted file keeps the real
Google/Telegram nodes enabled, contains no test substitutions
and no credentials, and was separately import-tested (created
in the instance as a check copy, 18 nodes read back cleanly,
then removed).

Test files, in order (the broken file sits mid-list to prove
the loop continues past a failed conversion):

1. `Jazz Violin Solo.ogg` (audio/ogg) — the schema's own example
   file on Wikimedia Commons
2. `Broken Upload.mp3` (audio/mpeg) — URL returns HTTP 404
3. `Violin Take Two.ogg` (audio/ogg) — same public file as #1
   under a different Drive name, proving per-item fields do not
   bleed across loop iterations
4. `liner notes.txt` (text/plain) — must be filtered out

## Execution 26 — full pass

(Execution 25 was orphaned by a service restart seconds after
it started, before any actor run was made; execution 26 is the
clean rerun, status **success**.)

- **Filter:** 4 items in → 3 true / 1 false. The .txt file was
  rejected; all three audio files passed.
- **Loop:** 3 iterations in order. The mid-list failure did not
  stop the loop.
- **Actor runs** (all rose_firehouse, all SUCCEEDED):
  - `HTQJTBMTP5152239m` — file 1: status **ok**, 192 notes,
    key **Bb major**, tempo **99.7 BPM**, 30 s audio,
    1 billed minute, **$0.01**
  - `i5WfaU2x6suoGTxuf` — file 2: status **error**
    ("Download failed: HTTP 404"), 0 billed minutes, **$0.00**
  - `iNL7eEXYZwpN7rU02` — file 3: status **ok**, same 192
    notes / Bb major / 99.7 BPM, **$0.01**
  - Total charged on the test account: **$0.02**. These are
    self-test runs and belong in the self-test ledger, never
    in traction numbers.
- **Prepare + success IF:** both ok rows flagged `ok: 1` and
  routed to delivery; the error row flagged `ok: 0` and routed
  back to the loop. Per-item isolation held: file 3's delivery
  carried its own Drive name ("Violin Take Two.ogg" →
  "Violin Take Two.mid"), not file 1's.
- **MIDI download (real):** the HTTP node fetched both MIDIs
  from the actor's key-value store (mimeType `audio/midi`,
  file name `001-00_Jazz_Violin_Solo.mid`). Byte check outside
  n8n: HTTP 200, **1,941 bytes**, valid `MThd` header
  (format 1, 2 tracks).
- **Log rows + Telegram texts** built for both conversions,
  e.g.:

  > 🎵 MIDI ready: 00_Jazz_Violin_Solo.ogg
  > Source: Violin Take Two.ogg
  > 192 notes · key of Bb major · 99.7 BPM
  > MIDI: https://drive.google.com/file/d/TEST_MIDI_FILE/view

  (the Drive link is the test substitute's fake webViewLink;
  in the submission workflow it is the real uploaded file's
  link, with the actor's own midiUrl as fallback.)

## Defects found

None in the template logic. One environment incident: the
first attempt (execution 25) was killed by a service restart
before reaching the Apify node — no actor runs, no charges;
the rerun is execution 26 above.

## Honest limits

- The Google legs (Drive trigger, share, Drive upload, Sheets
  append) were **not** executed locally — no Google OAuth app
  on this VM, by design, same as every precedent template.
  Their configuration mirrors the proven node shapes from the
  earlier templates; an importer's first run is the first live
  exercise of those four nodes.
- The Telegram node was **not** executed (no bot credential
  exists in the local instance). Its input — the finished
  message text — was produced by the byte-identical builder
  node and captured in the execution output.
- The Drive link sharing model: the workflow shares each audio
  file "anyone with the link" so the converter can fetch it —
  flagged by the red sticky note in the workflow itself.
