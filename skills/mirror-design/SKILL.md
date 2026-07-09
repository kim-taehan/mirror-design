---
name: mirror-design
description: Plan and implement new screens/pages by mirroring what your codebase already has — no designer needed. Decomposes the requested screen into UI elements, finds the most-used implementation of each element across existing screens (grep, frequency-first), extracts measured style values from real code, produces a reviewable HTML mockup + a reference-mapping table with file:line provenance, stops for human approval, then implements. Use when asked to "add a screen", "add a page", "plan a screen", "new admin page", "make it match the existing UI", or when building UI in a codebase that has no documented design system.
---

# mirror-design

**Your existing screens ARE the design system.** A mirror invents nothing — every element of a new screen is a reflection of a proven piece found across your existing ones.

Given a screen request: decompose it into UI elements → for each element, find the existing screen that implements it best (**assemble best-of sources**, the heart of this skill) → produce a **review-ready package** (reference mapping table + visual mockup + paste-ready summary) → and only after a human says OK, implement it.

## ⛔ This is NOT a creative design tool (first principle)

**Never invent a design, layout, or style.** The moment you make up a plausible-looking color, radius, or spacing, this skill has failed. The only job is: **search the screens that already exist in this codebase and reproduce, with their actual values, the form used most often.**

- **No invention**: every color, px, radius, font, and component shape that appears in a mockup or implementation must be **a value extracted from real code**. Eyeballed approximations and improvised styles are forbidden.
- **Frequency first**: if several screens implement the same element differently, **grep-count the usages and pick the most common form**. (Example: 5 screens use preset buttons for date filtering and 0 use a custom pill → preset buttons. The pill would be an invention.)
- **Exhaustive search is allowed**: when a source is ambiguous, don't save tokens — **search every relevant screen** to confirm the real form and values. Grep beats guessing.
- **Resolve theme indirection to real values**: spacing scales (e.g. MUI `sx` numbers × 8px), theme tokens (`theme.gray[500]`, Tailwind classes, CSS variables) must be resolved to their actual computed values before they go into a mockup.
- If no real precedent exists, **do not improvise — stop and ask** (see the completeness gate below).

## Prerequisites

- Run **inside the target frontend repo** (cwd = repo root, or ask for the path). Any stack — React/Vue/Svelte/Angular; MUI/Ant Design/Tailwind/CSS modules/hand-rolled tokens. Detect the stack from `package.json` and the source layout.
- If you can't see a pages/views directory and a routing setup, ask for the repo path. Never assume.

## Phase 0 · Census (first run per repo — cached)

Before the first planning run in a repo, build a **design census**: a measured survey of what the codebase actually uses. Follow `references/census.md` (in this skill folder) for what to measure and the output format.

- Save it to **`.mirror-design/census.md`** in the target repo. Committing it is recommended (teammates and future runs benefit); gitignore it if the team prefers.
- On later runs, **reuse the cache** instead of re-surveying. Re-verify a cached entry when you're about to rely on it and the cited file has changed, and refresh sections that turn out stale — the census records file:line evidence, so drift is detectable.
- The census is also where per-repo traps live (e.g. "a shared `Search` component exists but 0 list screens actually use it — do not pick it as a reference").

## Phase 0.5 · Input

- Use the free-text argument as the requirement directly.
- If the requirement is ambiguous, don't guess — narrow it with **one** question, then proceed.

## Two-phase flow (Plan → [human gate] → Implement)

Planning and implementation are **always separated.** With no designer on the team, a human looking at the plan before code is the only gate. If only planning was requested, stop after Phase 1.

---

### Phase 1 · Plan (no code written — produces a review package)

0. **Screen-count check (repeat Phase 1 per screen).** One ticket often means several screens (list + detail, list + create form, multiple tabs). List the screens you'll build and confirm with the user ("① announcement list ② announcement detail — 2 screens"). Then:
   - Plan **each screen separately** (decompose → map → measure → mock). Screen types differ, so their references differ.
   - **Measure shared elements once and reuse** (common page chrome, the same category badge). No duplicate extraction across screens.
   - Deliver the review package **per screen** (N mockups, N mapping tables). If the user is in a hurry, plan and get approval for the primary screen (usually the list) first.

1. **Classify** — map the request to a screen type: `List` / `Detail` / `Form (Add·Edit)` / `Dashboard` — adjusted to whatever taxonomy the census found in this codebase. Also identify the domain.

2. **Element decomposition** — the **core move**. Don't copy a screen wholesale; split the target screen into UI elements: page skeleton / filter bar / data table & columns (badges, chips) / search / pagination / empty·loading·error states / special interactions (deep links, read markers, …). Each element may have a *different* best-reference screen.

3. **Per-element reference search (in living code, via grep)** — **never mirror one screen wholesale.** For each element, find the screen that implements that pattern best, and combine the best sources. Search anchors (adapt to the census's naming conventions):
   - Screen-type skeleton: suffix globs like `src/pages/**/*List.tsx`, `*Detail.tsx`, `*Form.vue` — pick the one **closest in domain** as the skeleton baseline.
   - Filter bar / search / pagination / empty & error states / URL-driven state: grep for the census's known identifiers (component names, hook names, query-param usage).
   - Pin every source as **file:line** (verified by actually reading the code — no guessing).

4. **Reference mapping table** — the **first and most important deliverable**. "What comes from which screen":

   | Element | Source screen (file:line) | Component/pattern to reuse |
   |---|---|---|
   | Page skeleton | `AnnouncementList.tsx:35-38` | Header + ContentContainer |
   | Filter bar (date/search/preset) | `ChatlogList.tsx:264-277` | startDate/endDate + activePreset |
   | Table + category badge | `AnnouncementList.tsx:74-90` | CATEGORY_CONFIG · PriorityChip |
   | Deep link (URL preset) | `DeploymentList.tsx:120-127` | useSearchParams().get(...) |

   → This table is the implementation blueprint *and* the evidence of consistency. The mockup and the implementation both derive from it.

   **⛔ Sibling-consistency gate — within one screen, elements with the same role use the same control.**
   - Element-wise sourcing has a trap: siblings on one screen can end up mismatched (e.g. three single-select filters in one filter bar, but one is a radio group sourced from a form dialog while the others are chips sourced from list filters).
   - After building the table, **group same-role elements** (single-select filters together, inputs together, action buttons together) and check the controls match.
   - If they don't, unify on the source that **dominates in this screen type** (most usages). Never transplant a form-dialog pattern into a list filter bar — the context differs.
   - Even if the user explicitly asks for a specific control (e.g. "make it radio buttons"), if it breaks sibling consistency, don't silently comply — surface it: "that clashes with the other two chip filters in the same bar. (A) all chips / (B) all radios — which?" Screen-level consistency wins.

   **⛔ Completeness gate (the invariant of this skill) — every UI element that appears in a mockup or implementation must have a file:line source in the mapping table.**
   - Before drawing the mockup, self-check: "Is every element of this screen in the table? Did I invent anything without a source?" **If an element has no source, do not draw it.**
   - For an element with no precedent, mark it `⚠ no source` in the table and **stop**: "this element has no precedent in this codebase. (A) substitute the nearest real pattern X (file:line) / (B) introduce a new pattern (needs team sign-off) — which?" **Never hand-draw it on your own.**
   - "Nearest real pattern" must itself be grep-verified to exist.

5. **Measured style extraction (⭐ mandatory before drawing the mockup — drawing without this step is failure).** For each element in the mapping table, **open the source file:line** and extract the style values. Every color/px/radius/font in the mockup must come from here; no other value may appear.
   - Resolve indirection per the census: spacing-scale numbers to px, theme tokens and CSS variables to actual hex, framework classes to computed values.
   - If elements are many, **fan out parallel subagents per element** (accuracy over token thrift). Each returns only "property: value + file:line".
   - Output: a **measured-style table** per element (e.g. `table th → bg #F3F4F6, color #384153, 14px/400, padding 0 16px, height 40px, border-bottom 1px #E5E7EB` ← `Table.tsx:334`). This table is the mockup's only style source.
   - ⚠ **Properties absent from the real component must be absent from the mockup** (if the real table has no outer border/radius/shadow, don't wrap the mockup table in a card). No "make it look nicer" additions.

6. **Produce the review package** — the mapping table + measured-style table plus:

   **(a) Visual mockup** — a standalone HTML file drawn **only from the measured-style table**, per `references/mockup-guide.md`. Mark new/changed elements with a `NEW` badge. Save under `.mirror-design/plan/<screen>-mockup.html` in the repo (working artifacts — the workspace's own `.gitignore` keeps `plan/` uncommitted).

   **(a′) Optional: screenshot / plan PDF** — if Playwright (or similar) is available, capture the mockup and embed it in a short plan document (background/scope, per-screen sections with the mapping table, data contract, implementation plan, change log) and render it to PDF — the easiest artifact to drop into Slack/Teams for review. Tip: `npx playwright screenshot --channel chrome …` uses the installed Chrome, which sidesteps corporate-proxy download failures of `playwright install`.

   **(b) Paste-ready summary** — a short message the user can paste into chat verbatim:
   ```
   [Screen plan review request] <screen name>
   • Type: list/detail/form/dashboard
   • Purpose: <one line>
   • References (per element): skeleton←<screen> · filters←<screen> · badges←<screen> (file:line)
   • Reused components: <list>
   • Data: <service.method> → <displayed fields>
   • New/changed: <bullets>
   • Touch points: route registration + nav/sidebar registration
   • Mockup: .mirror-design/plan/<file> (or attached PDF)
   ```

   **(c) Implementation plan** — files to create (following the census's naming rules), registration points (routes, nav), permission wrappers, loading/empty/error handling. **Every item links back to a mapping-table source.**

7. **Human gate** — stop with: "Implement as planned? If you want a design review first, post the mapping table + mockup." **Do not proceed without an explicit go.**

---

### Phase 1.5 · Revision loop (before approval — repeat as needed)

When review comments come back ("category filter should be radio, not dropdown", "pin urgent items on top", "drop the author column"):

- **Update everything together**: mockup + plan doc (re-capture → re-render, bump version) + summary + implementation plan. Updating only one desyncs them.
- **Keep paths stable**: overwrite the mockup/plan HTML at the same path. Version the PDF (v2, v3…) so the review thread shows the progression.
- **Mark what changed**: elements changed in this revision get a `CHANGED` label (distinct from `NEW`). Reviewers check "was my comment applied?" at a glance.
- **One-line change log**: "v2: category filter dropdown→radio (reviewer comment)".
- **Convention-violating requests get an alternative, not compliance**: if a comment asks for a component/pattern/hardcoded color the codebase doesn't have, propose the nearest existing pattern with evidence and confirm.
- Loop until approved. Then Phase 2.

> If change requests arrive after Phase 2 has started, apply the same principle: update code, mockup, and plan together.

---

### Phase 2 · Implement (only after approval)

Follow the census conventions, with **the mapping table as the blueprint**:

- **Element-wise mirroring**: assemble each element from its mapped source (skeleton←A, filters←B, badges←C). No wholesale screen copies, no new layouts, no hardcoded values where the codebase uses tokens.
- **Files & naming**: follow the census's file/naming rules (e.g. `src/pages/Xxx/Xxx{List,Add,Detail,Edit}.tsx`). Follow the codebase's existing data-fetch pattern — don't introduce a new state-management approach.
- **Registration — every touch point the census lists** (typically: route table + navigation/sidebar; missing the nav entry is the classic "why isn't my page showing" bug).
- **Verify**: run the repo's typecheck/build (from its README/CLAUDE.md). Fix until green.
- **No commits** unless the user asks. Report the changed-file list and confirm the registration points were done.

## Safety guards

- Repo or path unclear → stop and ask.
- **Never implement without the plan gate** — the gate is the point of this skill.
- `.mirror-design/plan/` artifacts are not source code — the workspace's own `.gitignore` keeps them uncommitted.
- Don't expand scope beyond the requested screens/features.

## Commands

- `/mirror-design:init` — scaffold the `.mirror-design/` workspace and build (or drift-check) the census, ending with a summary + trap report
- `/mirror-design:audit` — audit screens against the census: severity-scored drift findings with evidence on both sides, a mandatory before/after mockup with `DRIFT` badges, verdict only (fixes on request)

If a user asks to initialize, survey, or audit in natural language, perform the equivalent of the matching command inline.
