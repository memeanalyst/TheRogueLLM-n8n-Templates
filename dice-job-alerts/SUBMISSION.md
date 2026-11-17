# n8n Template — Dice Job Alerts (New Postings from Your Watchlist)

**Status:** built and live-tested locally 2026-11-17 (see
TEST_EVIDENCE.md). Package ready for Hamzah to submit to n8n
himself — this package is the deliverable, nothing has been
submitted yet.

**Actor:** magnova/dice-jobs-monitor (YYGe1r58Cxd02CO48) —
$0.001 per posting returned, no start fee.

## What it does

Every morning, for each search on a Google Sheets watchlist
(query, location, workplace types), the workflow runs the Dice
Jobs Scraper actor on Apify, filters out postings already in
the Log tab, asks Gemini for a one-line summary of each
genuinely new posting, sends one Slack alert per new posting,
then appends it to the Log. Postings past the per-search alert
cap stay unlogged and surface on the next run.

## Files

- `workflow.json` — the submission workflow (19 nodes, real
  Sheets/LLM/Slack nodes, no credentials).
- `tests/build_test_files.py` — builds the local test copies.
- `TEST_EVIDENCE.md` — the three-execution live test record.

## Submission checklist (per the house n8n guidelines)

- [x] Yellow overview sticky note (top-left, ~200 words)
- [x] White section sticky notes for each stage (≤50 words)
- [x] One red warning note (don't delete Log rows)
- [x] Descriptive node names throughout
- [x] Live-tested end-to-end on the rose_firehouse test account
- [x] Import-tested: the submitted file creates cleanly in n8n
