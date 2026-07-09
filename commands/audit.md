---
name: mirror-design:audit
description: Audit screens against the design census — severity-scored drift findings backed by file:line evidence on both sides, delivered with a mandatory before/after mockup. Judges only; fixes on request.
---

# Audit screens against the census

Find where a screen drifts from what this codebase actually uses. The structure mirrors a strict design review — severity scoring, a false-positive filter, verdict only — but the judging standard is different: **every finding must cite census evidence.** Taste is not a standard here. "This looks wrong to me" is inadmissible; "5 list screens use preset buttons (`census.md` → `ChatlogList.tsx:264`), this one introduces a radio group with no precedent" is a finding.

This command **judges by default.** Report findings and a verdict; only fix when the user asks. Keep the audit and the mutation cleanly separate.

## How to run it

### Step 1 — Load the census

Read `.mirror-design/census.md`. If it doesn't exist, stop and say: "No census found — run `/mirror-design:init` first." An audit without a census would be an opinion, and opinions are what this skill exists to eliminate. Spot-check the census entries you're about to rely on (their file:line still matches); refresh a stale entry before judging with it.

### Step 2 — Bound the scope

- An argument (file, directory, or screen name) → audit exactly that.
- No argument → if the git working tree has UI changes, audit that diff. Otherwise ask **one** question to pin the scope, then proceed.

### Step 3 — Evidence pass

Decompose each screen in scope into UI elements (same decomposition as planning: skeleton, filter bar, table & columns, search, pagination, states, interactions). For each element:

1. Find the census's dominant pattern for that element (or grep-measure it now if the census lacks the entry — and add it).
2. Compare against what the screen actually does.
3. For every deviation, record **both sides' evidence**: the screen's value (file:line) and the census's standard (file:line + usage count).

A candidate finding without both citations is not a finding yet — go read the code until it has them.

### Step 4 — Score severity

- **Blocker** — an invention: a pattern/control with no precedent in the codebase, or hardcoded values where the codebase uses tokens. The exact failure mirror-design exists to prevent.
- **Should-fix** — a minority pattern where a dominant one exists (2/10 screens do it this way, 8/10 the other), or a sibling-consistency violation (same-role elements using different controls on one screen).
- **Note** — minor drift; mention once, don't dwell.

### Step 5 — Filter false positives

Drop these before reporting — this filter is half the command:

- **Ratified variation.** If the census records the deviation as a legitimate context-specific pattern ("RadioGroup appears only in form dialogs"), a form dialog using it is not drift.
- **Outside the scope** set in Step 2, or lines a diff-audit didn't touch.
- **No census evidence.** If you can't cite the standard being violated, cut the finding. The audit must not invent, either.
- **Lint/format territory** — owned elsewhere, not a drift finding.

Prefer a few high-conviction findings over a long cosmetic list.

### Step 6 — Report with the mockup (mandatory)

**(a) Before/After mockup — always produced, never optional.** Build a standalone HTML file at `.mirror-design/plan/<screen>-audit.html` per `references/mockup-guide.md`:

- **Before**: the screen as it currently is, reproduced from measured values, with a `DRIFT` badge on each violating element.
- **After**: the same screen with each violation replaced by the census-standard pattern, drawn from the census's measured values.
- Side by side, the audit findings table embedded below.
- Every style value in both panes comes from real code — the Before pane from the audited screen, the After pane from the census sources. No invented values anywhere, including here.

**(b) Findings table** (also in the reply):

| Element | Screen's value (file:line) | Census standard (file:line, usage) | Severity |
|---|---|---|---|

**(c) Verdict.** Blockers present → **not passed**, list what must change. No Blockers → **passed**, with Should-fix items as recommendations.

### Applying fixes — only when asked

If the user asks to fix, enter the standard revision loop (Phase 1.5 of the skill): update the mockup in place at its stable path so the user watches the screen evolve, mark changed elements, keep a one-line change log, and implement only after approval — through the same human gate as any planned screen.
