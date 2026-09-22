# Current Session

**Stage:** 1 — Complete and verified. Stage 2 not yet started. Stage 3 scoped/designed, not yet built.

**Last completed:** Merged the JD screener and application tracker into one file (`index.html`) as a fourth "Tracker" tab, built in a new, isolated private repo (`job-search-toolkit`) so neither live tool was touched. Added a CSV Import feature to the tracker (real parser, not a raw storage copy) as the migration path for Chris's real entries.

**Critical bug found and fixed mid-build:** The merged tool initially reused the exact same `localStorage` keys as both live tools. Chrome doesn't reliably scope `localStorage` per exact `file://` path, so opening the merged file read the live tracker's real 43 entries directly out of shared storage — discovered when Chris opened the file and the Tracker tab showed all 43 entries before any import happened. No data was lost (read-only at that point), but it broke the isolation the whole project was built around. Fixed by namespacing every storage key to `jst_*`. Re-verified after the fix: Tracker tab started empty, CSV import brought in exactly 43 entries.

**Also fixed:** A structural HTML bug (Add/Edit modal wasn't nested inside the tracker's scoped container, so it rendered inline instead of as a hidden overlay) — caught via browser testing, not caught by code review alone.

**Simplified per feedback:** Dropped the tracker's independent light/dark toggle — always dark now, matching the rest of the tool. Widened the table layout so it no longer needs a horizontal scrollbar at normal desktop width.

**Stage 3 design settled (not yet built):** Monthly summary gets an opt-in "Generate insights" feature on top of the deterministic counts — AI-generated, advisory-only suggestions about possible scoring-rule tweaks, gated behind deterministic trigger conditions (starting with: 5+ roles at the same company scored 7+ without reaching interview/screen) so the AI only runs against real patterns, not noise. Manual button only, never automatic; output never writes into the actual scoring rules. Full design notes below under Stage 3.

**Next actions:**
- Stage 2 — cross-reference screener JD screens against tracker company history (deterministic, read-only, surfaced the same way as the existing screening-history banner)
- Stage 3 — build the deterministic monthly counts first, then the opt-in insights layer described below
- Cutover decision (repoint GitHub Pages / fold into old repo / keep standalone) — intentionally not decided yet, deferred until Chris explicitly approves cutover after full verification

**Open questions / decisions pending:**
- Repo visibility is currently private (holds real tracker data during migration); revisit whether to flip to public at cutover, per Chris's original instruction that the screener repo is public.
- Stage 3 trigger conditions are deliberately left open-ended — more trigger types to be added as they come up, not a fixed list.

---

# Job Search Toolkit — Project Log

## Project Overview

A merge of two previously-separate, working single-file HTML tools — a JD screener/resume tailor (Claude API, browser-only, user's own key) and an application tracker (localStorage, no API calls) — into one tool, plus new cross-referencing and monthly-summary features layered on top. Built in a new, isolated repo so the two live tools stayed untouched and usable throughout the build; cutover to be decided explicitly later.

**Design principle, carried over from the source tools:** no self-modifying/self-tuning scoring logic, anywhere. Scoring and tailoring-language rules stay deliberately separate (`getRules()` excludes `languageRules`; `getCustomRules()` includes it). Hard disqualifiers (procurement/sourcing, engineering-domain roles, people-management as a core responsibility) stay hard; years-of-experience and "P&L ownership" language stay intentionally soft/interpretive. The SQL-qualifier/tailoring-language prompt reliability issue is accepted as-is — manual review is the intentional safety net, not something to solve with a self-check pass.

## Stages

| Stage | Name | Status |
|---|---|---|
| 1 | Tracker as a fourth tab, full feature parity, verified CSV migration | Complete |
| 2 | Cross-referencing (screener ↔ tracker, deterministic, read-only) | Not started |
| 3 | Monthly summary: deterministic counts + opt-in AI insights layer | Designed, not built |

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

## Stage 2 — Cross-Referencing (NOT STARTED)

When screening a JD, check the tracker's own data (not just the existing `jst_screen_history_v1` fuzzy-match banner) for real application history at that company — did Chris actually apply, what was the outcome, relevant notes — and surface it the same way: deterministic lookup, read-only, not fed back into the scoring/tailoring prompts.

## Stage 3 — Monthly Summary & Insights (DESIGNED, NOT BUILT)

**Deterministic base (always on):** counts by status, most common rejection stage, most-flagged gaps — computed directly from tracker data, no AI involved.

**Insights layer (opt-in, advisory only):**
- Triggered by a manual "Generate insights" button — never automatic. A date-based nudge (e.g. "last generated 34 days ago") may surface, but nothing fires without a click.
- Gated by deterministic trigger conditions that must be met before the AI even runs, keeping API cost and attention focused on real patterns rather than noise. Starting trigger (first to implement, thresholds tunable once seen against real data):
  - **Same-company repeat pattern**: 5+ roles at the same company scored 7+ without reaching interview/screen.
  - Open-ended — more trigger types to be added as they come up (e.g. a shared gap flagged across multiple different companies, high score + early rejection more broadly). Not committing to a fixed list now.
- Score signal is parsed directly out of the tracker's own notes text (regex for patterns like "8/10 match") rather than joining against the separate screening-history log — more robust, works even for entries the cross-reference join in Stage 2 doesn't match, and isn't capped by that log's 200-entry retention limit.
- Output is always framed as an observation worth a look, not a prescription — advisory text only. **Never written into the actual scoring rules automatically**, consistent with the project's standing no-self-modification guardrail.
- Not persisted to localStorage. Exportable as Markdown (native download) and PDF (via browser print-to-PDF, no bundled library). A `mailto:` link can pre-fill an email with the summary for the user to send themselves — there's no backend, so nothing is sent automatically.

**Relationship to Stage 2:** independent in practice (score now comes from notes-text parsing, not the cross-reference join), but Stage 2's cross-referencing still lands first since it was already next and has its own value at screening time.
