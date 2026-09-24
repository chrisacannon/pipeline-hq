# Pipeline HQ

A single-file web tool that screens job postings against a resume, drafts tailored application materials, tracks every application, and surfaces monthly patterns — all in one place, powered by the Claude API. Merged from two separate tools ([job-description-screener](https://github.com/chrisacannon/job-description-screener) and a standalone application tracker) into one, with new cross-referencing and monthly-summary features layered on top.

Live: *(pending — GitHub Pages is being set up to publish from `/docs` in this private repo)*

The app lives at [`docs/index.html`](./docs/index.html) — that's both the file to edit and the one GitHub Pages publishes, so there's no separate copy to keep in sync. Everything else at repo root (this README, the project log) stays off the published site since Pages only serves what's inside `/docs`.

## What it does

1. **Screen a role** — paste a job posting and get a 1–10 match score, a verdict (apply / review / pass), a salary recommendation, fit reasons, gaps, and a plain-language summary. If you've screened a role at this company before, or actually applied there, both show up as banners above the result — prior score/verdict, and real outcome/notes from the Tracker — so a repeat posting doesn't get evaluated in a vacuum.
2. **Tailor (score 5+)** — generates a first-draft tailored resume and cover note based on your resume, the posting, and your own scoring rules. Flags claims worth double-checking before you submit.
3. **Tracker** — log every application: status (Applied, Recruiter screen, Interview, Offer, five rejection-stage variants, Withdrawn, No response), salary details, next actions, and notes. Filter by status, search, click a notes cell to expand it, export/import as CSV.
4. **Summary** — a monthly recap computed directly from tracker data: applications sent, how many reached a screen or further, offers, and the most common rejection stage. Optionally, click "Generate insights" to have Claude look for a specific pattern (currently: 5+ roles at the same company scored 7+ that never advanced) and suggest — in plain language, for you to evaluate — whether something about how you're scoring roles might be worth a second look. Nothing it says is ever written back into your scoring rules automatically.

## Getting started

1. Open the page — it starts blank except for one fabricated example row in the Tracker tab (delete it once you add your own). The first visit walks through setup: paste your resume, optionally answer a set of scoring-rule questions (target roles, hard disqualifiers, skills that shouldn't count against you, language rules for tailored drafts, etc.).
2. Enter your own Anthropic API key (get one at [console.anthropic.com](https://console.anthropic.com/)). Used only to call the API directly from your browser, never saved — you'll re-enter it each visit.
3. Screen a role, or head to the Tracker tab and add/import your applications.

Each screening costs roughly $0.005–0.01 in API usage; tailored materials add another $0.01–0.02. Generating Summary insights is similarly small and only runs when you click the button.

## Your data

* Resume, rule answers, tracker entries, and screening history are all saved in your browser's local storage only, tied to this page's URL. Nothing is uploaded to a server.
* The only network call this page makes is directly from your browser to Anthropic's API, using your own key.
* The Summary tab's insights are generated on demand and not persisted anywhere — export as Markdown or PDF (via your browser's print dialog) if you want a record, or use the "Email myself" link to send it to your own inbox.
* Because storage is per-browser, everyone who opens this page gets their own private setup — nothing is shared between visitors, and nothing here is shared with the two original tools this was merged from.
* This repo is private. A checkpoint CSV and some build-reference snapshots that briefly contained real personal application data (including third-party contact info from recruiters) were removed and purged from the full git history — see the project log for details.

## Project history

This started as a merge of two working, separately-maintained tools, built carefully to avoid touching either live tool during the build (see [pipeline-hq-project-log.md](./pipeline-hq-project-log.md) for the full history, including real bugs caught during development — a storage-isolation bug that briefly had this tool reading the live tracker's real data, and a `window` scoping bug that silently broke two features).

## Tech notes

* Single static HTML file. No build step, no backend, no dependencies beyond the Anthropic API and Google Fonts.
* Model: `claude-sonnet-4-6` via `POST https://api.anthropic.com/v1/messages`, called directly from the browser.

Built by Chris Cannon · [chrisacannon.github.io](https://chrisacannon.github.io/)
