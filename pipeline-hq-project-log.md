# Current Session

**Stage:** Post-launch iteration (v2 work). The app is live and the repo is public; this session added status timelines, a second insight trigger, and a public-facing cleanup pass.

**Footer and naming cleanup:** removed the dev-style build/version lines from the page footer (the project log is the real changelog). The Summary tab's insights section is now "Pipeline insights", and each trigger carries a category tag (existing trigger: "Scoring calibration"; new one: "Automated screening") so non-scoring patterns aren't mislabeled.

**Status timelines (new):** each application can carry dated steps after applying (e.g. Recruiter screen 9/18/26, Interview 9/24/26). The current `status` is always the last step's, so filters/stats/Summary are unchanged; entries logged before this existed have no steps and behave exactly as before. UI: a "Timeline (n)" toggle under the status badge expands a vertical list (collapsed by default, like notes); the edit modal has a "Date of this status change" field (appears when the status dropdown changes) and a timeline editor with "+ Add step" for back-filling/correcting. CSV gained a ninth "Status Timeline" column (`Recruiter screen 2026-09-18; Interview 2026-09-24`); older 8-column files still import, and a round-trip export→import was verified identical. No new table column was needed.

**Second insight trigger — quick rejections:** fires when 3+ applications have a dated `rejected_app`/`rejected_unspecified` step the same or next day after the applied date (dates are day-level, so "same or next day" is the real threshold). Chris's read, adopted in the framing: quick rejections on freshly listed roles are almost always automated (ATS/AI) screening, so the insight treats that as given and instead asks which mechanism the notes point to (knockout form answers such as years of experience, literal keyword/requirement mismatch, level mismatch, or something shared across the roles). Note the tracker has no posting-age field, so "freshly listed" isn't visible to the trigger; anything like "posted 2 days ago" has to go in notes. The real tracker's existing entries have no dated steps, so this trigger sees nothing until timelines accrue. Each trigger now owns its own prompt (`buildPrompt`), and groups carry a `label` instead of assuming one company.

**Root cause of the lingering table side-scroll found and fixed:** it wasn't the overall width — the Edit/Delete buttons in the narrow Action column were `nowrap` and couldn't shrink, pushing the table past its container whenever the window was under ~1900px (the 1px overflow measured earlier at 1920px was the same thing). Buttons now stack; measured 0px overflow at 1280, 1500, and 1920px with timelines expanded. Status column trimmed 16%→14%, notes 17%→19%.

**Also:** About tab no longer advertises the removed dark-mode toggle and now describes the timeline and Summary tab; two modal placeholders that were borrowed from a real employer's name and role title were replaced with neutral ones.

**Backup/privacy/persona pass (built on a separate local branch, `v2-backup-privacy-persona`, not yet merged or pushed — Chris was mid-use of the live tool and asked that nothing be pushed without his go-ahead):**
- **Resume & rules backup:** "Export backup" (Resume & rules tab) writes one JSON file (`type: pipeline-hq-profile`, version, timestamp, resume, all rule answers — no API key, no tracker data). "Import backup" validates the file, asks for confirmation before replacing saved data, saves immediately, and drops unknown keys / fills missing ones with empty strings. "Restore from backup" on the welcome screen lets a returning user on a new browser or origin skip setup. Tracker data keeps its own CSV export. Verified: export→restore round trip, four malformed/mismatched files rejected with saved state untouched, hostile content in a backup stays inert text (no element injection), and welcome-screen restore.
- **Synthetic example persona wired in** ("Jordan Ellis"): same career shape as the real one, with names, employers, schools, dates, figures, product category, and the two scoping rules swapped (SQL→Power BI, financial modeling→demand forecasting, each with the large/small/personal-project structure). Employers and schools are fictional but deliberately stately-sounding; fictional names rather than real prestigious ones, since invented figures and degrees attached to real institutions would read as claims about them. Resume text is internally consistent with its own hard rules (no bare "Power BI" or "demand forecasting" tags). Job title is "Sr. Category Strategy Manager, Filter Supplies" ("Portfolio &" dropped as redundant). A "Fictional example" note now sits above all four example boxes, and the remaining "Chris" mentions in UI text (About paragraph, load-example confirm) were genericized; the author credit line stays.
- **Privacy wording made complete rather than reassuring.** Audited every external request the page can make: Anthropic API (user-triggered), Google Fonts stylesheets (on load), nothing else — no analytics, cookies, beacons, or third-party scripts. The old copy ("the only network call this page makes is to Anthropic") was not strictly true because of Google Fonts; that claim is gone. The About tab has a new Privacy section, the welcome screen and README say what is sent to Anthropic (resume, JD, rules, and tracker notes for insights), that fonts load from Google, and that GitHub Pages may keep ordinary host logs. Open option: self-host or drop the web fonts to remove the Google request entirely.

**Old `job-description-screener` repo made private (Oct 2026).** Its live site went offline as a result — checked directly: the Pages URL now returns GitHub's "Page not found," and Pipeline HQ's site was unaffected. **Correction to an earlier note in this log:** it said a private repo's Pages site "stays live/public unless Pages is explicitly turned off." For this account's plan that was wrong — Pages isn't supported on private repos here (confirmed earlier when enabling Pages on this repo failed), and flipping the old repo private unpublished its site. Its stale Pages configuration still shows as "built" in the API and would presumably republish if the repo were made public again. The old repo's source (which includes the real example resume and rules) is now private; the earlier history of this repo still contains the original real example text.

**Open / pending Chris's review:** merging and pushing the branch above; demo mode (preloaded synthetic persona, sample JD, canned screen/tailor output, synthetic tracker, isolated `?demo` storage) as the chosen alternative to funded free uses — Chris's other pricing-intel app runs on Streamlit, which suits a server-backed free-tier model, whereas this tool stays server-less; whether to drop Google Fonts; whether to rewrite this repo's history to remove the original real example text (would need a forced push Chris runs himself).

---

**Stage (earlier this month):** Cutover — live. All three build stages are complete; the app is now published and the repo is public.

**Live at https://chrisacannon.github.io/pipeline-hq/** — GitHub Pages publishing from `main` branch, `/docs` folder. Verified directly (not assumed): the app loads and works at the root URL, and `README.md`/`pipeline-hq-project-log.md` both 404 when requested at that URL, confirming Pages only serves what's actually inside `/docs` regardless of what else is public in the repo.

**Repo renamed `job-search-toolkit` → `pipeline-hq`** (this log file renamed to match). Chosen after brainstorming several directions; landed on "pipeline" since it matches the tool's actual mental model (screen → tailor → track → summarize) better than a generic "toolkit" name. App's `<title>`/logo updated to match. README.md added.

**Sensitive-data cleanup, then repo made public:**
- Two files had real personal data: the CSV checkpoint (43 real tracker entries) and `reference/app-tracker-live.html` (a Step-0 build snapshot with the *original* standalone tracker's real 33-entry `SEED` array hardcoded directly in its HTML/JS source — found during the public-repo discussion, not caught earlier). Also removed `reference/screener-index-live-2026-09-16-v4.html` (not sensitive, just build scaffolding no longer needed) while cleaning up.
- Chris's call: rather than just keep these out of what gets published, remove them entirely so there's nothing to remember to manage going forward. Deleted from the tree, then **purged from the full git history** with `git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch checkpoints reference' --prune-empty -- --all`, force-pushed. Verified exhaustively afterward — walked every commit in the rewritten history (`git rev-list --all` + `git ls-tree -r` per commit), confirmed neither path appears anywhere, not just spot-checked. History went from 13 commits to 11 (two commits that only ever touched those paths became empty and were auto-pruned).
- Ran into a real environment issue partway through: Claude Code's own auto-mode classifier blocks `git filter-branch` as a destructive operation, even with explicit user go-ahead — Chris had to run the rewrite himself in his own terminal. That surfaced a second friction point: the repo lives inside a OneDrive-synced folder, and `git gc`'s cleanup step kept hitting OneDrive's file locks (dozens of "deletion failed, retry?" prompts). Resolved by skipping `gc` entirely (local disk housekeeping only, doesn't affect what's on GitHub) and just running the force-push directly.
- Before flipping visibility, ran one more full-history scan for anything email/phone-shaped as a final check — found only Chris's own commit-author email and Claude's attribution line, nothing else. Repo flipped to public, then GitHub Pages enabled and verified live (see above).
- **Reversed an earlier decision along the way**: had briefly settled on "stay private, publish via Pages from a private repo anyway" — turned out GitHub Pages isn't available for private repos on Chris's plan at all (confirmed directly via the API, not assumed), which forced the actual choice between paying for GitHub Pro, a third-party static host, a public mirror repo, or going fully public. Chris's reasoning for going public once that was the real choice: the sensitive-data reason for privacy was the main one and was now fully resolved; the other stated reasons (not a pure work-sample project, wanting curated access) were reasons he didn't *mind* being private, not reasons to actively avoid public.
- Separately flagged for Chris's own reconsideration (not urgent, not blocking): the embedded `EXAMPLE_RULE_ANSWERS` content is more tactically candid than a typical public example (e.g. explicit "AI-assisted, not independent proficiency" framing for Python/R) — he's inclined to anonymize/soften some of it, tabled for later.

**App restructured for publishing:** `index.html` moved to `docs/index.html` — single source of truth for both editing and what Pages publishes, no separate copy to keep in sync.

**Portfolio-readiness pass (this session):**
- "See how Chris answered these" → "See examples"; "Load Chris's example answers" → "Load example answers" — less personal-sounding for something meant to be picked up by others.
- Tracker now seeds one clearly-fictional "Example Corp" entry on first-ever empty load (mirrors the original standalone tracker's `seedIfEmpty()` pattern, fabricated data instead of real history) so the notes field has a visible template.
- Tracker table widened again (max-width 1680px → 1900px, tighter padding, notes column 18%→17%) — measured directly (not just eyeballed) that overflow is down to a 1px rounding artifact at a 1920px viewport; real-world result depends on Chris's actual window width.
- Considered, not built: a synthetic multi-row CSV (matching the tracker's import format) as a richer demo artifact than the single seeded row. Assessed as a nice-to-have, not essential — tabled unless wanted later.

**Next actions:**
- ~~Decide on and execute the old `job-description-screener` repo's visibility + Pages disconnect~~ — done; see the correction above (flipping private did take the site down, contrary to what this line originally said).
- Optional: anonymize `EXAMPLE_RULE_ANSWERS`; optional: synthetic multi-row CSV demo file; optional local `git gc` once OneDrive isn't mid-sync (cosmetic, no functional impact).

**Earlier — Stage 3 recap:** Built and verified (base summary + one trigger). Deliberately holding at one trigger after a design discussion talked several candidate triggers back out of scope — see below.

**Last completed:** Stage 3 — a fifth "Summary" tab with deterministic monthly counts (applications sent, reached-screen-or-further, offers, status breakdown, most common rejection stage) plus an opt-in, trigger-gated AI insights layer. One trigger implemented: 5+ roles at the same company self-scored 7+ that never reached a screen or interview. Chris found a real bug (see below), fixed and reverified.

**Trigger design discussion (Sept 2026) — decided to hold at one trigger for now:** After building the first trigger, brainstormed several more candidates (cross-company gap-language pattern, reverse miscalibration — low scores that still reached interview/offer, exact-score-boundary underperformance, hard-disqualifier-boundary testing, ambiguous-title-family clustering, salary-ask mismatch, stalled-high-scorers). Working through them surfaced a general principle: any trigger needs enough *independent* samples (different companies/JDs) pointing the same direction to overcome the fact that each screening is a single, un-repeated, potentially noisy draw from the model — precise numeric cutoffs (e.g. treating a score of exactly 7 as meaningfully different from 8) are too vulnerable to that noise to trust. The same-company trigger already has that protection built in (it requires 5 independent draws agreeing). But Chris raised a further, more fundamental objection that closed off the two strongest remaining candidates (cross-company gap pattern, reverse miscalibration): how a given "gap" is weighed genuinely varies company to company, and even hiring-manager to hiring-manager within the same company — so aggregating gap language *across* companies assumes a consistency in employer judgment that doesn't actually exist. That's a different problem than model-scoring noise, and sample size doesn't fix it. Decision: keep the original same-company trigger (it aggregates within one hiring process, not across inconsistent ones); the other candidates are recorded below for possible future reconsideration, not implemented now.

**Earlier — Stage 2 recap:** Screening a JD now surfaces a second banner (alongside the existing screening-history one) showing real tracker application history at that company: role, status, date applied, notes snippet. Deterministic fuzzy match, same pattern as the existing screening-history lookup, read-only, never fed back into scoring/tailoring.

**Bug found and fixed during Stage 2 testing:** `const Tracker = (function(){...})();` at script scope does not attach to `window` — only the bare `Tracker` identifier is globally accessible. The new `findTrackerHistory()` and the pre-existing tab-switch refresh (`showTab()`) were both written against `window.Tracker` and were silently no-op-ing. Changed the declaration to `window.Tracker = ...`; both now work. (The tab-switch refresh bug predates Stage 2 — it existed since Stage 1 but never visibly mattered since data doesn't change between tab switches on its own.)

**Earlier — Stage 1 recap:** Merged the JD screener and application tracker into one file (`index.html`) as a fourth "Tracker" tab, built in a new, isolated private repo (`job-search-toolkit`) so neither live tool was touched. Added a CSV Import feature to the tracker (real parser, not a raw storage copy) as the migration path for Chris's real entries.

**Critical bug found and fixed mid-build:** The merged tool initially reused the exact same `localStorage` keys as both live tools. Chrome doesn't reliably scope `localStorage` per exact `file://` path, so opening the merged file read the live tracker's real 43 entries directly out of shared storage — discovered when Chris opened the file and the Tracker tab showed all 43 entries before any import happened. No data was lost (read-only at that point), but it broke the isolation the whole project was built around. Fixed by namespacing every storage key to `jst_*`. Re-verified after the fix: Tracker tab started empty, CSV import brought in exactly 43 entries.

**Also fixed:** A structural HTML bug (Add/Edit modal wasn't nested inside the tracker's scoped container, so it rendered inline instead of as a hidden overlay) — caught via browser testing, not caught by code review alone.

**Simplified per feedback:** Dropped the tracker's independent light/dark toggle — always dark now, matching the rest of the tool. Widened the table layout so it no longer needs a horizontal scrollbar at normal desktop width.

**Stage 3 design settled (not yet built):** Monthly summary gets an opt-in "Generate insights" feature on top of the deterministic counts — AI-generated, advisory-only suggestions about possible scoring-rule tweaks, gated behind deterministic trigger conditions (starting with: 5+ roles at the same company scored 7+ without reaching interview/screen) so the AI only runs against real patterns, not noise. Manual button only, never automatic; output never writes into the actual scoring rules. Full design notes below under Stage 3.

**Next actions:**
- All three planned stages are now complete. Remaining: cutover decision (repoint GitHub Pages / fold into old repo / keep standalone) — intentionally not decided yet, deferred until Chris explicitly approves cutover after full verification.
- Optional, not scheduled: revisit shelved trigger candidates if a better way to handle cross-employer inconsistency comes up.

**Open questions / decisions pending:**
- Repo visibility is currently private (holds real tracker data during migration); revisit whether to flip to public at cutover, per Chris's original instruction that the screener repo is public.
- Trigger set intentionally held at one (`same-company-high-score-no-advance`) — see the trigger design discussion above for why the other candidates were shelved rather than built.

---

# Pipeline HQ — Project Log

*(Named `job-search-toolkit` during development; renamed to `pipeline-hq` at cutover-planning time. References to the old name below are historical record, not stale — that's genuinely what it was called when each entry was written.)*

## Project Overview

A merge of two previously-separate, working single-file HTML tools — a JD screener/resume tailor (Claude API, browser-only, user's own key) and an application tracker (localStorage, no API calls) — into one tool, plus new cross-referencing and monthly-summary features layered on top. Built in a new, isolated repo so the two live tools stayed untouched and usable throughout the build; cutover to be decided explicitly later.

**Design principle, carried over from the source tools:** no self-modifying/self-tuning scoring logic, anywhere. Scoring and tailoring-language rules stay deliberately separate (`getRules()` excludes `languageRules`; `getCustomRules()` includes it). Hard disqualifiers (procurement/sourcing, engineering-domain roles, people-management as a core responsibility) stay hard; years-of-experience and "P&L ownership" language stay intentionally soft/interpretive. The SQL-qualifier/tailoring-language prompt reliability issue is accepted as-is — manual review is the intentional safety net, not something to solve with a self-check pass.

## Stages

| Stage | Name | Status |
|---|---|---|
| 1 | Tracker as a fourth tab, full feature parity, verified CSV migration | Complete |
| 2 | Cross-referencing (screener ↔ tracker, deterministic, read-only) | Complete |
| 3 | Monthly summary: deterministic counts + opt-in AI insights layer | Complete (v1, one trigger — more shelved, see discussion) |

## Stage 1 — Tracker Merge (COMPLETE)

*Completed: September 2026*

### What was built
- Tracker ported in as a fourth tab (`Screen a role` / `Resume & rules` / `Tracker` / `About`), fully namespaced — CSS scoped under `#tab-tracker` with `tk-*` classes, JS wrapped in a `window.Tracker` closure — so nothing collides with the screener's own identically-named classes/globals.
- Full feature parity: add/edit/delete, all 11 status categories, the corrected "Active" filter (excludes all rejected variants, withdrawn, pending), wide fixed-column layout, click-to-expand notes, CSV export.
- New CSV Import feature (RFC4180-ish parser: quoted commas, embedded newlines, escaped quotes) as the real migration path, replacing the old approach of hand-copying a `SEED` array into the file.

### Data safety checkpoint (per the project's own guardrail)
1. Confirmed live screener `index.html` matched "Rules build: 2026-09-16-v4" before touching anything.
2. Confirmed real tracker entry count: 43 (not 34 as in the original scoping doc — just time passing, not a discrepancy).
3. Exported CSV from the live tracker and committed it to the new repo (`checkpoints/applications-2026-09-21-checkpoint.csv`) before any code was written.

### Bugs found during verification
- **Storage isolation bug (critical):** see Current Session above.
- **Modal-nesting bug:** Add/Edit modal rendered inline instead of as an overlay; caught via real browser rendering, fixed by correcting HTML nesting.
- **Missing try/catch on Tracker's theme/save calls:** inconsistent with the rest of the codebase's storage-access pattern; added for graceful degradation under storage-restricted contexts.

### Verification method
Claude's built-in browser preview sandboxes local files as `data:` URLs (no `localStorage`), and Chrome-extension automation can't drive the native address bar for `file://` navigation — so real end-to-end testing required Chris's own browser. Claude's own testing used a shimmed in-memory `localStorage` to exercise the real `Tracker` code path (import, add, edit, delete, search, filter, export) against synthetic data covering every status label and CSV quoting edge case, then Chris performed the real 43-in/43-out verification himself after the isolation fix.

### Post-Stage-1 tweaks (feedback)
- Dropped the tracker's independent light/dark toggle; now always dark, matching the rest of the tool.
- Widened the table layout (`shell.wide` 1500px → 1680px, trimmed cell/card padding) to remove the horizontal scrollbar at normal desktop width.

## Stage 2 — Cross-Referencing (COMPLETE)

*Completed: September 2026*

### What was built
- A second banner in the Screen a role tab, alongside the existing `jst_screen_history_v1` fuzzy-match banner: when the screened company matches a tracker entry (same normalize-and-substring fuzzy match as the screening-history lookup), shows role, status label, date applied, and a notes snippet (truncated ~180 chars) for each real application at that company.
- `Tracker.getApps()` — a new read-only accessor returning copies of tracker entries with a resolved `statusLabel`, so the screener side never touches Tracker's internal state directly.
- Deterministic lookup only — not sent back through the model, same category as the existing screening-history feature.

### Bug found during verification
`const Tracker = (function(){...})();` at script scope does not attach to `window`. Both the new cross-reference lookup and the pre-existing tab-switch refresh referenced `window.Tracker` and were silently no-op-ing (returning `[]` / doing nothing rather than erroring, which is why it wasn't obvious). Fixed by changing the declaration to `window.Tracker = ...`. Caught via direct browser testing of `findTrackerHistory()`, not code review.

### Verification method
Claude tested the real code path (shimmed `localStorage`, seeded a tracker entry, called `findTrackerHistory()` and `renderScreen()` directly with messy casing/punctuation to confirm fuzzy matching) for the same environment reasons as Stage 1. Chris then verified for real: opened the local file, screened a JD for a company already in his tracker, confirmed the banner appeared correctly.

## Stage 3 — Monthly Summary & Insights (COMPLETE, v1)

*Completed: September 2026*

### What was built
A fifth "Summary" tab.

**Deterministic base (always on, no AI):** month selector (derived from tracker dates), applications sent that month, count that reached screen-or-further, offers, full status breakdown, most common rejection stage. "Reached screen+" correctly counts `screen`/`interview`/`offer` **and** `rejected_screen`/`rejected_hm`/`rejected_late` — all of those mean a screen actually happened before the rejection; only `rejected_app`/`rejected_unspecified`/`pending` mean it didn't.

**Insights layer (opt-in, advisory only):**
- Manual "Generate insights" button only — never automatic.
- Gated behind deterministic trigger conditions that must fire before the API is ever called. One trigger implemented: `same-company-high-score-no-advance` — 5+ roles at the same company self-scored 7+ (parsed from notes text, e.g. "8/10 match") that never advanced past `rejected_app`/`pending`.
- If no trigger fires: "No patterns strong enough to flag yet" — no API call, no cost.
- Output is always framed as advisory observations plus an explicit caveat sentence, never a prescription. **Never written into the actual scoring rules automatically.**
- Not persisted to localStorage. Exportable as Markdown (native download), Print/Save as PDF (browser print-to-PDF, no bundled library), and a `mailto:` link (pre-fills a summary; no backend, so nothing sends itself).

### Bug found and fixed
"Reached screen+" initially only checked current status (`screen`/`interview`/`offer`), missing that `rejected_screen`/`rejected_hm`/`rejected_late` also represent having reached a screen before being rejected there. Found by Chris checking real data; fixed and reverified.

### Verification method
Claude tested the full code path end-to-end with seeded synthetic data via a shimmed `localStorage` (same technique as Stages 1–2): correct month-list derivation, correct per-month stats, correct trigger firing *and* correct non-firing on a mixed-status group and a low-score company, a mocked insights call confirming the prompt includes both the flagged entries and the current scoring rules, and correct Markdown/email export content. Chris then verified against his real 43 entries and caught the "reached screen+" undercount that Claude's synthetic test data hadn't happened to exercise in a way that would surface it.

### Trigger design discussion — additional candidates considered, not built
After Stage 3 shipped, brainstormed further trigger ideas and worked through each against a general principle: **any trigger needs enough independent samples (different companies/JDs) pointing the same direction to overcome the fact that each screening is a single, un-repeated draw from the model** — a precise numeric cutoff (e.g. treating score 7 as meaningfully different from 8) is too vulnerable to that per-run noise to trust on its own.

Candidates considered:
1. **Cross-company gap pattern** — same gap language (e.g. "domain gap," "no consulting pedigree") recurring across many different companies at high scores with poor outcomes. Structurally sound against the sample-size principle (many independent companies required), but identifying "the same gap" from free text would itself require an LLM call to cluster meaningfully — which defeats the deterministic-gate design goal. A fixed keyword list (from Chris's own Resume & Rules vocabulary) was proposed as a workaround.
2. **Reverse miscalibration** — 5-6-scored ("review") roles that still reached interview/offer, suggesting scoring might be too conservative. Meshes with the sample-size principle, but "reached interview" doesn't cleanly prove undervalued fit (could be a lenient recruiter, a referral, a JD reading worse than the real role) — the caveat framing would need to carry that explicitly.
3. **Exact-score-boundary underperformance** (score exactly 7 vs. 8+) — rejected: too vulnerable to single-run scoring noise, no sample-size protection since it's about precision at one boundary rather than a repeated directional pattern.
4. **Hard-disqualifier-boundary testing** (people-management language present but not scored as disqualifying) — rejected for the same reason as #3: the underlying judgment is itself a soft, interpretive call, so a single ambiguous case isn't informative regardless of volume.
5. **Ambiguous-title-family clustering** — outcome rate by title pattern (e.g. "Product Manager") vs. overall rate. Rejected: real signal is confoundable with market conditions, referrals, and timing unrelated to scoring at all — too noisy to draw a conclusion from.
6. **Salary-ask mismatch** — stated salary above posted range correlating with fast rejection. Not pursued: this is self-evident to Chris while entering tracker data, not something that needs a computed pattern surfaced back to him.
7. **Stalled-high-scorers** — high scores sitting at "pending/no response" past some threshold. Flagged as potentially misleading: more likely a signal about tailoring/application quality than scoring calibration, and conflating the two risks pointing at the wrong fix.

**Decisive objection (closed further work for now):** working through #1 and #2 — the two strongest remaining candidates — surfaced a problem sample size doesn't solve: how a given "gap" is weighed genuinely varies company to company, and even hiring-manager to hiring-manager within the same company. Aggregating gap language or outcome patterns *across* companies assumes a consistency in employer judgment that doesn't actually exist. The original same-company trigger doesn't have this problem — it aggregates within one hiring process, not across inconsistent ones.

**Decision:** hold at the one implemented trigger. The other candidates are recorded here for possible future reconsideration (e.g. if a good way to handle cross-employer inconsistency comes up), not scheduled.

**Relationship to Stage 2:** independent in practice (score comes from notes-text parsing, not the Stage 2 cross-reference join), but Stage 2 landed first since it was already next and has its own value at screening time.
