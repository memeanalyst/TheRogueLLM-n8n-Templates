# n8n Template — Fare Drop Watch (Flight Price Alerts from Your Watchlist)

**Status:** built and live-tested locally 2026-11-20 (see
TEST_EVIDENCE.md). Package ready for Hamzah to submit to n8n
himself — this package is the deliverable, nothing has been
submitted yet.

**Actor:** magnova/flight-price-watch (eGG0ek0O3cOadXuFf) —
$0.002 per flight record, $0.02 per price-drop alert, no start
fee. Watch-mode checks that find no drop charge $0.

## What it does

Every morning, for each flight on a Google Sheets watchlist
(origin, destination, departure date, optional return date,
target price, cabin class, currency), the workflow runs the
Flight Price Watch actor on Apify in watch mode. The actor
remembers the fares it has seen: the first check stores a
baseline, later checks return only fare drops and target hits.
The workflow filters out everything that is not a genuine,
not-already-alerted drop, sends one Slack alert per drop
(route, dates, airline, old price → new price, drop %,
Google Flights link), then appends the drop to the Alert Log
tab so the same drop is never alerted twice.

## Files

- `workflow.json` — the submission workflow (17 nodes: 11
  functional + 6 sticky notes, real Sheets/Slack nodes, no
  credentials).
- `tests/build_test_files.py` — builds the local test copies.
- `TEST_EVIDENCE.md` — the live-test record (baseline,
  no-change, forced drop, dedupe).

## Submission checklist (per the house n8n guidelines)

- [x] Yellow overview sticky note (top-left, 230 words,
      How it works / Setup / Customization)
- [x] Four white section notes, each ≤50 words, covering
      their node groups
- [x] One red warning note, over the Apify node: the first
      check per watch is baseline-only
- [x] Every node name descriptive (verb + service/data)
- [x] Live-tested end-to-end locally with the rose_firehouse
      test account before packaging
