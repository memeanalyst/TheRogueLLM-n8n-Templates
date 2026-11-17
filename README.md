# The Rogue LLM: n8n Templates

Free n8n workflows from The Rogue LLM YouTube channel: https://www.youtube.com/@theroguellm

## YouTube Outlier Finder

`youtube-outlier-finder/YouTube-Outlier-Finder.json`

Every Monday it checks the channels you choose, finds the long videos doing 3x or more of that channel's normal views (views ÷ the median of its recent long-form uploads), and sends you one Telegram message per outlier with why it likely popped plus 3 video ideas for your channel. An outlier is never sent twice, and a quiet week sends nothing.

**Nodes:** Schedule Trigger → Set (your channels & limits) → Apify (YouTube Channel Scraper, Skip Shorts on) → Code (outlier math + dedupe) → IF (any new outliers?) → Basic LLM Chain + any chat model → Telegram. If the scrape itself fails, a side branch sends a failure alert instead of staying silent.

### Setup
1. In n8n: **Workflows → Import from file**, pick the JSON.
2. Install the Apify community node (`@apify/n8n-nodes-apify`) if you don't have it, and add your Apify API token.
3. In **Set your channels & limits**, replace the placeholder channels with the ones you want to track (handles, channel IDs or URLs), tune the multiplier and view floor, and describe your channel in one line.
4. Add a chat model credential (Gemini is set by default; any chat model works).
5. Add your Telegram bot credential and replace `YOUR_TELEGRAM_CHAT_ID` in both Telegram nodes (or swap Telegram for Slack or email).

Disclosure: the YouTube Channel Scraper actor is built by us, and we earn when it runs.

## Slogan Trademark Checker — Etsy & Print-on-Demand Ban Shield

`slogan-trademark-checker/Slogan-Trademark-Checker.json`

Paste your t-shirt, mug and sticker slogans into a Google Sheet and it checks each one against live USPTO trademarks in the print-on-demand classes (clothing 025, housewares 021, paper goods 016, textiles 024). It writes the verdict back into your sheet: RISK — an identical live trademark exists, with the owner and serial number; CHECK — a similar live mark contains your phrase; or No live match found. This is a screening aid, not legal advice.

**Nodes:** Schedule Trigger → Google Sheets (read unchecked slogans) → Loop Over Items → Apify (USPTO Trademark Search) → Code (verdict) → Google Sheets (write verdict)

### Setup
1. In n8n: **Workflows → Import from file**, pick the JSON.
2. Install the Apify community node (`@apify/n8n-nodes-apify`) if you don't have it, and add your Apify API token.
3. Create a Google Sheet with the headers `Slogan | Verdict | Owner | Serial` in a tab named `Slogans`, put one slogan per row, and connect it in both Google Sheets nodes with your Google credential.
4. Run it manually, or let the daily schedule check for new slogans. Each run checks the rows that don't have a verdict yet (up to 10 trademark records per slogan, $0.004 per record on your Apify account).

Disclosure: the USPTO Trademark Search actor is built by us, and we earn when it runs.

## Competitor Stealth Radar — New Trademark Filing Alerts

`competitor-stealth-radar/Competitor-Stealth-Radar.json`

Companies file trademarks months before they announce a product, feature, or sub-brand. Every Monday this workflow checks the competitors on your watchlist against the official USPTO database, spots filings that are new since the last run, has an AI explain in plain English what each filing probably signals, and posts one alert per filing to Slack. Every filing is written to a Log sheet, and that log is the memory: a filing whose serial number is already logged is never alerted twice.

**Nodes:** Schedule Trigger → Set (lookback window & caps) → Google Sheets (read filing log) → Google Sheets (read watchlist) → Loop Over Items → Apify (USPTO Trademark Search, by owner) → Code (keep only new filings) → IF (new filing?) → Basic LLM Chain + Gemini → Code (build alert + log row) → Slack → Google Sheets (append to log)

### Setup
1. In n8n: **Workflows → Import from file**, pick the JSON.
2. Install the Apify community node (`@apify/n8n-nodes-apify`) if you don't have it, and add your Apify API token.
3. Create a Google Sheet with a tab named `Watchlist` (header `ownerName`, one exact USPTO owner name per row, e.g. `Nike, Inc.`) and a tab named `Log` with the headers `serialNumber | wordmark | owner | filedDate | internationalClasses | goodsAndServices | aiSummary | dateLogged`. Connect it in all three Google Sheets nodes. Don't delete Log rows — the Log is the workflow's memory.
4. Add a chat model credential (Gemini is set by default; any chat model works) and connect the Slack node to your workspace and channel.
5. Run it once manually, then activate the Monday schedule. Only filings from the last 14 days are fetched, so weekly runs cost a few cents (the actor charges $0.004 per trademark record returned, on your Apify account).

Disclosure: the USPTO Trademark Search actor is built by us, and we earn when it runs.
