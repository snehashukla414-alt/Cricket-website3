# INDIA XI — Cricket Central (v2)

A Cricbuzz-inspired match centre focused on India's men's and women's national cricket teams.

## Features
- Live / upcoming / results match views.
- Men's / women's and format filters.
- Match detail page with match summary, scorecards, batting and bowling tables, extras, fall of wickets and source metadata.
- Player search and profile/career splits when available from the API.
- Refresh button and automatic live-view refresh every 60 seconds.
- No fabricated scores: unavailable data is shown as unavailable.
- Optional Cloudflare Worker proxy so a public GitHub Pages website does not expose its API key.

## Data provider
This build integrates with [OpenCricket API](https://opencricketapi.com/) REST endpoints documented at https://opencricketapi.com/docs:
- `/v1/live` — live scores (Developer plan per current docs)
- `/v1/fixtures` — upcoming fixtures (Developer plan per current docs)
- `/v1/matches` — match archive/search (Starter plan)
- `/v1/matches/{id}` — match summary
- `/v1/matches/{id}/scorecard` — batting/bowling scorecard
- `/v1/search?q=...&type=player` — player search
- `/v1/players/{id}` — player profile and career splits

Plan names and limits can change. Check provider docs and pricing before deploying. Some player fields and match coverage may be unavailable depending on the underlying source.

### About Playing XI
The API documents match summaries, scorecards, player profiles and live match centre data, but does not document a separate guaranteed official playing-XI endpoint for every fixture. The site therefore does not falsely label scorecard participants as the confirmed XI. It explains this in the match detail view and displays published batting/bowling scorecards. If the provider adds confirmed lineup data, this can be added as a dedicated panel.

## Quick start
1. Download and unzip the project.
2. Open `index.html` using VS Code Live Server or any static web server.
3. Click **Data Setup** and enter an OpenCricket API key for personal testing, or a Worker proxy URL for public use.
4. Choose Live / Upcoming / Results.

## Publish on GitHub Pages
1. Create a GitHub repository.
2. Upload the contents of this folder to the repository root.
3. Go to **Settings → Pages**.
4. Choose **Deploy from a branch**, select `main` and `/(root)`, then Save.

## Public deployment security
GitHub Pages is static hosting. Never put an API key in `app.js`, HTML, or a committed config file. Direct-key mode saves the key in the visitor's browser and is for personal testing only. For a public website, deploy `worker/worker.js` to Cloudflare and save the API key as the Worker secret `OPENCRICKET_KEY`. See `worker/README.md`.

## Attribution
Match responses include source/licence metadata. Historical records may come from Cricsheet (ODC-By 1.0); live scores and fixtures may be sourced from ESPN. Display and follow the provider's current attribution and terms for any public deployment. This project is an independent fan project and is not affiliated with BCCI, ICC, Cricbuzz or OpenCricket.

## Multi-model AI analyst (v3 update)
The AI Analyst section can send the selected match context and a question to every configured model in parallel:
- OpenAI / ChatGPT (`OPENAI_API_KEY`)
- Google Gemini (`GEMINI_API_KEY`)
- Anthropic Claude (`ANTHROPIC_API_KEY`)
- Optional Google Programmable Search Engine (`GOOGLE_CSE_KEY` + `GOOGLE_CSE_CX`) for source links and secondary context

Add these as **Cloudflare Worker secrets**, not in `app.js`, HTML, GitHub, or browser local storage. Optional model overrides are `OPENAI_MODEL`, `GEMINI_MODEL`, and `ANTHROPIC_MODEL`. The Worker defaults are set in `worker.js` and can be changed to models available to your provider accounts. Each configured model returns its own answer so you can compare them.

### Configure AI and Google Search
1. Deploy the updated `worker/worker.js` to Cloudflare Workers.
2. In Worker **Settings → Variables and Secrets**, add `OPENCRICKET_KEY` plus whichever AI keys you have: `OPENAI_API_KEY`, `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`.
3. For Google Search links, create a Programmable Search Engine and add `GOOGLE_CSE_KEY` and `GOOGLE_CSE_CX`. Google Search API access, quotas and billing are subject to Google's current terms.
4. Set `ALLOWED_ORIGINS` in `worker.js` to your exact GitHub Pages origin, then redeploy.
5. In the site, open **Data Setup**, paste the Worker URL ending in `/api`, and save.
6. Open **AI Analyst** and ask a question. The site sends match-feed data and the question to every configured AI provider and displays their separate responses plus Google search source links.

### Important data integrity rule
AI models and Google Search are analysis/context layers, not a trusted live score database. Runs, balls, wickets, overs, batting figures and bowling figures must come from the cricket data provider's structured feed and its scorecard. AI answers are labelled as analysis and instructed not to invent missing numbers. Search snippets can be incomplete or stale; use their source links to verify. Add other licensed cricket-data APIs as additional score-feed adapters if you need broader coverage or failover. Do not scrape Google results pages or unofficial score sites.

### Cost and privacy
Each AI question can call every configured model and Google Search, which may incur separate usage charges and quota limits. The current design runs AI only when a visitor asks, not on every 60-second score refresh. Do not send personal information in questions. Add Cloudflare rate limiting before sharing the Worker publicly.

Model defaults are configurable. As checked against current provider documentation in November 2026, the bundle defaults to OpenAI `gpt-6-luna`, Gemini `gemini-3.8-flash`, and Claude `claude-haiku-5-5`. Provider accounts can have different model availability; set the corresponding `*_MODEL` variable to a model enabled for your API account if needed.
