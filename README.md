# Pipeline HQ

A single-file web tool that screens job postings against a resume, drafts tailored application materials, tracks every application, and surfaces monthly patterns — all in one place, powered by the Claude API. Merged from two separate tools ([job-description-screener](https://github.com/chrisacannon/job-description-screener) and a standalone application tracker) into one, with new cross-referencing and monthly-summary features layered on top.

Live: https://chrisacannon.github.io/pipeline-hq/

The app lives at [`docs/index.html`](./docs/index.html) — that's both the file to edit and the one GitHub Pages publishes, so there's no separate copy to keep in sync. Everything else at repo root (this README, the project log) stays off the published site since Pages only serves what's inside `/docs`.

## What it does

1. **Screen a role** — paste a job posting and get a 1–10 match score, a verdict (apply / review / pass), a salary recommendation, fit reasons, gaps, and a plain-language summary. If you've screened a role at this company before, or actually applied there, both show up as banners above the result — prior score/verdict, and real outcome/notes from the Tracker — so a repeat posting doesn't get evaluated in a vacuum.
2. **Tailor (score 5+)** — generates a first-draft tailored resume and cover note based on your resume, the posting, and your own scoring rules. Flags claims worth double-checking before you submit.
3. **Tracker** — log every application: status (Applied, Recruiter screen, Interview, Offer, five rejection-stage variants, Withdrawn, No response), salary details, next actions, and notes. Each application can carry a dated status timeline ("Recruiter screen 9/18/26 → Interview 9/24/26") that expands under the status badge. Filter by status, search, click a notes cell to expand it, export/import as CSV.
4. **Summary** — a monthly recap computed directly from tracker data: applications sent, how many reached a screen or further, offers, and the most common rejection stage. Optional "pipeline insights" appear only when a deterministic pattern shows up in your data — currently two: 5+ roles at the same company scored 7+ that never advanced (scoring calibration), and 3+ rejections arriving the same or next day after applying (almost always automated screening). Clicking "Generate insights" then has Claude describe, in plain language and for you to evaluate, what the notes suggest. Nothing it says is ever written back into your scoring rules automatically.

## Try it without a key

Click **Try the demo** (next to the API key box, and on the welcome screen), or open the page with `?demo=1` on the end of the address. It loads a fictional candidate's resume, rules, and tracker, plus pre-written sample output for Screen, Tailor, and Generate insights, so you can click through everything with no API key and no setup. Sample output is labelled as such wherever it appears; the demo makes no API calls and keeps its data under separate storage keys, so it never reads or changes a real profile in the same browser. "Reset demo data" and "Exit demo" are in the banner.

## Getting started

1. Open the page — it starts blank except for one fabricated example row in the Tracker tab (delete it once you add your own). The first visit walks through setup: paste your resume, optionally answer a set of scoring-rule questions (target roles, hard disqualifiers, skills that shouldn't count against you, language rules for tailored drafts, etc.). The "See examples" links show a fictional sample candidate. Already have a backup file? "Restore from backup" brings back your resume, rules, and screening history and skips the questions.
2. Enter your own Anthropic API key (get one at [console.anthropic.com](https://console.anthropic.com/)). Used only to call the API directly from your browser, never saved — you'll re-enter it each visit.
3. Screen a role, or head to the Tracker tab and add/import your applications.

Each screening costs roughly $0.005–0.01 in API usage; tailored materials add another $0.01–0.02. Generating Summary insights is similarly small and only runs when you click the button.

## Your data

**Nothing is collected.** There are no accounts, analytics, cookies, or tracking scripts, and no server behind this page. Here is everything it does with your information, including the parts that aren't obvious:

* **Where it lives:** your resume, rule answers, tracker entries, and screening history are saved in your browser's local storage for this site, on this device only. Clearing your browser's site data deletes them, so use **Export backup** (resume, rules, and screening history — company, title, score, verdict, and date for each screen, never job-description text — as one JSON file) and **Export CSV** (tracker) to keep copies. Importing a backup replaces your saved resume and rules but only *adds* to your screening history, so nothing already saved is removed. Backups contain your resume and the companies you've screened — store them somewhere private. If you used the earlier standalone version of this tool in the same browser, Pipeline HQ offers once to add that version's screening history; it only reads it, never changes or deletes it, and only if you say yes.
* **What leaves your browser, and only when you click:** Screen, Generate (tailored materials), and Generate insights send text directly from your browser to Anthropic's API using your own key — your resume, the job description, your scoring rules, and, for insights, the notes from the tracker entries involved. Anthropic's terms and privacy policy govern it from there. The key itself is never stored by this page.
* **Two network requests you don't control:** the page loads its fonts from Google Fonts when it opens (so Google can see your IP address and browser details, as with any site using it), and it's served by GitHub Pages, which, like any host, may keep ordinary server logs this tool has no access to.
* **Summary insights** are generated on demand and not persisted anywhere — export as Markdown or PDF (via your browser's print dialog) if you want a record, or use the "Email myself" link, which only opens your own email app with a short summary pre-filled.
* **Demo mode** makes no API requests (its output is pre-written) and stores its sample data under separate keys.
* Because storage is per-browser, everyone who opens this page gets their own private setup — nothing is shared between visitors. Browsers also keep storage separately per site address, so the live site and a local copy of the file each have their own data; use the exports above to move between them.
* This repo is public. A checkpoint CSV and some build-reference snapshots that briefly contained real personal application data (including third-party contact info from recruiters) were removed and purged from the full git history before it was made public — see the project log for details.

## Project history

This started as a merge of two working, separately-maintained tools, built carefully to avoid touching either live tool during the build (see [pipeline-hq-project-log.md](./pipeline-hq-project-log.md) for the full history, including real bugs caught during development — a storage-isolation bug that briefly had this tool reading the live tracker's real data, and a `window` scoping bug that silently broke two features).

## Tech notes

* Single static HTML file. No build step, no backend, no scripts or libraries loaded from elsewhere — the only external resources are the Anthropic API (when you ask it to) and Google Fonts (stylesheets only).
* Model: `claude-sonnet-4-6` via `POST https://api.anthropic.com/v1/messages`, called directly from the browser.

Built by Chris Cannon · [chrisacannon.github.io](https://chrisacannon.github.io/)
