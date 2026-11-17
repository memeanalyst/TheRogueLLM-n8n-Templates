# Test Evidence — Dice Job Alerts (n8n)

**Date:** 2026-11-17 · **Instance:** local n8n 2.47.7
(`~/workspace/tools/n8n`, DB `~/.n8n/database.sqlite`) ·
**Apify package:** `@apify/n8n-nodes-apify` v0.9.0 ·
**Credential:** `apifytestcred1` (rose_firehouse TEST account — every
actor run below ran on the test account, never Magnova) ·
**Test workflows in the instance:** `diceTestA` (empty log),
`diceTestB` (log = Test A's alerts), `diceTestC` (log = every job ID
the actor returned in Test B), all inactive.

## What was substituted locally, and why

The local n8n has **no Google OAuth app, no Gemini credential, and no
Slack workspace**, so five legs cannot run here. The test copies
replace exactly those legs (builder: `tests/build_test_files.py`):

| Submission workflow | Local test copies |
|---|---|
| Google Sheets "Read job watchlist" | Code node returning 2 test searches: `python` (Remote), `cobol` |
| Google Sheets "Read alert log" | Code node returning the test log (A: empty; B: A's 20 alerted IDs; C: all 60 IDs returned in B) |
| LLM chain "Summarize the posting" + Gemini model | Code node (per-item mode) returning a fixed mock summary as `text` |
| Slack "Send alert to Slack" | Code node passing items through so alert texts land in the execution output |
| Google Sheets "Log new job to Sheet" | Code node passing items through so log rows land in the execution output |

Test settings: `max_results_per_search` 30 (submission: 50);
`posted_window_days` 7 and `max_alerts_per_search` 10 as submitted.
A Manual Trigger is added to the test copies (the n8n CLI cannot
start a workflow from a schedule trigger alone).

Everything else is the real thing and ran for real: the Loop Over
Items node, the Apify node (real runs of magnova/dice-jobs-monitor,
ID `YYGe1r58Cxd02CO48`, on the test account), the filter Code node,
the IF node, and the alert/log-row Code node — **byte-identical to
the ones in the submitted `workflow.json`**. The submitted file
keeps the real Sheets, LLM and Slack nodes, contains no test
substitutions and no credentials, and was separately import-tested
(created in the instance as a check copy, 19 nodes read back
cleanly, then removed).

## Three executions — full lifecycle, no defects found

**Execution 14 (Test A, empty log) — pass.**
- Loop processed both watchlist searches and completed.
- Each Apify run returned 31 dataset items (30 postings + the run
  SUMMARY, which the filter correctly drops).
- Filter kept 10 per search (the alert cap), 10 alerts built with
  the final alert text (title, company, location line, pay line,
  mock summary, posting URL), 20 log rows total.

**Execution 15 (Test B, log = Test A's 20 alerts) — pass.**
- 20 further alerts, **zero overlap** with the logged IDs: the
  filter surfaced the next unalerted postings from each search —
  the designed behaviour (postings past the alert cap stay
  unlogged and surface on the next run instead of being lost).

**Execution 16 (Test C, log = all 60 returned job IDs) — pass.**
- **0 alerts.** For both searches the filter emitted the single
  "nothing new" marker, the IF routed it down the false branch,
  and the loop completed. The quiet path works.

## Notes for the submitter

- The workflow inherits the Competitor Stealth Radar skeleton,
  including its hard-won fixes: the Log is read *before* the
  Watchlist (item-flow order), the Log read has Always Output
  Data on (an empty Log must not end the run), and the IF uses
  the `equals` operator.
- Sticky notes follow the house checklist: one yellow overview
  note, white section notes, one red warning note (don't delete
  Log rows).
