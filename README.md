# The Rogue LLM: n8n Templates

Free n8n workflows from The Rogue LLM YouTube channel: https://www.youtube.com/@theroguellm

## YouTube Outlier Finder

`youtube-outlier-finder/YouTube-Outlier-Finder.json`

Every Monday it checks the channels you choose, finds the long videos doing 3x or more of that channel's normal views (views ÷ the median of its recent long-form uploads), and sends you why each one popped plus 3 video ideas for your channel.

**Nodes:** Schedule Trigger → Apify (YouTube Channel Scraper, Skip Shorts on) → Code (outlier math) → Basic LLM Chain + any chat model → Telegram

### Setup
1. In n8n: **Workflows → Import from file**, pick the JSON.
2. Install the Apify community node (`@apify/n8n-nodes-apify`) if you don't have it, and add your Apify API token.
3. In the Apify node, replace `@YourCompetitor` / `UCxxxxxxxxxxxxxxxxxxxxxx` with the channels you want to track (handles, channel IDs or URLs).
4. Add a chat model credential (Gemini is set by default; any chat model works).
5. In the LLM node, replace `[DESCRIBE YOUR CHANNEL HERE]` with what your channel is about.
6. Add your Telegram bot credential and replace `YOUR_TELEGRAM_CHAT_ID` (or swap Telegram for Slack or email).

Disclosure: the YouTube Channel Scraper actor is built by us, and we earn when it runs.
