# Test Evidence — Fare Drop Watch (n8n)

**Date:** 2026-11-20 · **Instance:** local n8n 2.47.7
(`~/workspace/tools/n8n`, DB `~/.n8n/database.sqlite`) ·
**Apify package:** `@apify/n8n-nodes-apify` v0.9.0 ·
**Credential:** `apifytestcred1` (rose_firehouse TEST account —
every actor run below ran on the test account, never Magnova) ·
**Test workflows in the instance:** `fareTestA`, `fareTestB`,
`fareTestC`, `fareTestD`, all inactive.

## What was substituted locally, and why

The local n8n has **no Google OAuth app and no Slack
workspace**, so four legs cannot run here. The test copies
replace exactly those legs (builder: `tests/build_test_files.py`):

| Submission workflow | Local test copies |
|---|---|
| Google Sheets "Read fare watchlist" | Code node returning the test watch (see below) |
| Google Sheets "Read alert log" | Code node returning the test log (A/B/C: empty; D: Test C's 10 log rows) |
| Slack "Send alert to Slack" | Code node passing items through so alert texts land in the execution output |
| Google Sheets "Log alert to Sheet" | Code node passing items through so log rows land in the execution output |

A Manual Trigger is added to the test copies (the n8n CLI
cannot start a workflow from a schedule trigger alone).

Everything else is the real thing and ran for real: the Loop
Over Items node, the Apify node (real runs of
magnova/flight-price-watch, ID `eGG0ek0O3cOadXuFf`, on the
test account), the filter Code node, the IF node, and the
alert/log-row Code node — **byte-identical to the ones in the
submitted `workflow.json`**. The submitted file keeps the real
Sheets and Slack nodes, contains no test substitutions and no
credentials, and was separately import-tested (created in the
instance as a check copy, 17 nodes read back cleanly, then
removed).

Test watches: A/B used YYZ → YVR, 2026-12-12. C/D used
ORD → DEN, 2026-12-19 (a fresh watch, so its baseline was
clean — see the throttle note below). Watch state lives in
the test account's own named KV store
(`flight-price-watch-state`), exactly as it will for any
importer.

## Executions — full watch lifecycle

**Execution 17 (Test A, baseline) — pass.**
Actor returned 20 BASELINE + 2 SUMMARY rows (per-route
summary + run summary; both are dropped by the filter by
design). Filter emitted the single "nothing new" marker, the
IF routed it down the false branch, **0 alerts**.

**Execution 18 (Test B, no change) — pass.**
Second check on the same watch: actor returned only the 2
SUMMARY rows — no itineraries delivered when nothing changed.
**0 alerts.** The quiet path works.

**Execution 22 (Test A rerun on ORD → DEN, baseline) — pass.**
14 BASELINE + 2 SUMMARY, **0 alerts**.

**Execution 23 (Test C, forced drop) — pass.**
The test watch's stored state prices were inflated ×1.5
(lowest $147 → $220) so the next check would see genuine
drops. Actor returned 13 PRICE_DROP + 2 SUMMARY. The filter
kept 10 (the per-watch alert cap), 10 alerts were built and
"sent", 10 log rows written. First alert, verbatim:

> ✈️ Fare drop: ORD → DEN · 2026-12-19
> American · 2026-12-19T22:04 → 2026-12-19T23:59
> was $220, now $147 (-33.2%)
> https://www.google.com/travel/flights?q=Flights+from+ORD+to+DEN+on+2026-12-19&curr=USD

**Execution 24 (Test D, dedupe) — pass.**
Alert Log pre-seeded with Test C's 10 log rows; state
re-inflated ×1.5 so the actor emitted the same 13 drops
again. The filter suppressed all 10 logged drops and
surfaced exactly the 3 that had overflowed Test C's cap —
**zero overlap** between alerted flightKey+price pairs and
the log. The designed behaviour: capped drops are not lost,
they surface on the next run; logged drops never repeat.

## Defects found and fixed during testing

1. **Field name (template bug, fixed).** The first build of
   the filter keyed on `recordType`, but the actor marks
   watch outcomes in **`changeType`** (BASELINE / PRICE_DROP /
   TARGET_HIT) on rows whose recordType stays `FLIGHT`.
   Test A passed by accident (everything was dropped either
   way); the filter, alert builder, log schema and overview
   note were corrected to `changeType` before Tests B–D.
2. **Corrupt-state crash (actor robustness, noted, not
   fixed here).** A test-harness error wrote a malformed
   record into the watch state store; the actor's next run
   crashed reading it (`InvalidResponseBodyError`, exit 91)
   instead of failing safe. In production the state is only
   ever written by the actor itself, so the exposure is
   narrow, but a defensive state-read catch belongs on the
   actor's fix list. The state was rebuilt losslessly from
   Test A's baseline dataset and the test sequence rerun.
3. **Google burst throttling (source behaviour, expected).**
   Two drop-check attempts during rapid repeat searching
   returned ERROR + SUMMARY (Google throttles bursts from
   shared platform IPs; the actor's pacing/backoff could not
   clear a hot window). Both runs SUCCEEDED with $0 charged
   and the workflow stayed quiet — the designed failure
   shape. Spacing the retry by ~3 minutes cleared it. This
   is the actor's known staging behaviour, recorded in its
   BUILD_NOTES.

## Charges on the test account

Baselines charged the `flight` event ($0.002/record); the
drop checks charged the `price-drop` event ($0.02/alert
record delivered by the actor, including the 3 over the
workflow's alert cap — the cap limits alerts sent, not the
actor's delivery). Errored checks charged $0.
