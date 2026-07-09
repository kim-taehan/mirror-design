---
name: mirror-design:init
description: Scaffold the .mirror-design/ workspace and build (or drift-check) the design census — a measured survey of what this codebase actually uses. Asks no design questions; the answers are in the code.
---

# Initialize mirror-design in this repo

Set up the workspace and build the census so every later run — planning, revision, audit — has measured evidence to stand on. **Ask no design questions.** This is not the place to pick a direction or a feel; the codebase already made those decisions, and the census only records them.

## Steps

### 1. Confirm the target repo

This must run inside a frontend repo: a pages/views directory and a routing setup must be findable. If they aren't, ask for the repo path and stop. Never assume.

### 2. Scaffold the workspace

Create the unified workspace (skip anything that already exists):

```
.mirror-design/
├── .gitignore          # content: "plan/"
├── census.md           # written in step 3
└── plan/               # working artifacts (mockups, screenshots, PDFs) — ignored
```

The internal `.gitignore` is what keeps the commit policy self-enforcing: `census.md` and `mockup-chrome.html` are shareable team assets; everything under `plan/` is per-run working material.

### 3. Build or refresh the census

- **No `census.md` yet** → run the full survey per `references/census.md` (in the mirror-design skill folder): stack & styling standard, resolved design tokens, shared components ranked by usage, screen-type skeletons, recurring element patterns, file & naming conventions, registration points. Every claim carries a file:line or a usage count.
- **`census.md` exists** → do NOT re-survey. Spot-check its file:line citations against the current code; re-measure and update only the sections that drifted, appending a dated one-line change note.

### 4. Report

End with a short summary the team can act on:

- Stack and styling mechanism detected
- Token count and where they resolve from
- Dominant screen skeletons (with the usage counts that make them dominant)
- Top shared components by usage
- **Traps, highlighted** — components that exist but are effectively unused ("`Search` exists but 0 list screens use it — do not use as a reference"). These are the most dangerous references; surface every one.
- Recommend committing `.mirror-design/census.md` (and `mockup-chrome.html` once it exists) so teammates' runs reuse it.

## Notes

- Running init is optional. A screen request without a prior init still builds the census automatically (Phase 0 of the skill) — init is the explicit entry point for team onboarding and cache refresh.
- Everything here obeys the skill's first principle: **measure, don't judge.** If a value can't be traced to real code, it doesn't go in the census.
