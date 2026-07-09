# The design census — measuring what the codebase actually uses

The census replaces the design-system documentation this codebase doesn't have. It is a **measured** survey — every claim in it carries a file:line (or a usage count), so it can be trusted for mockups and re-verified when the code drifts.

Output: `.mirror-design/census.md` in the target repo. Recommend committing it so teammates and future runs reuse it.

## Rules of measurement

1. **Count, don't judge.** "Which pattern is standard?" is answered by `grep -rl <anchor> <pages-dir> | wc -l`, not by which looks cleanest. Record the count next to the verdict (e.g. "TextField + SearchIcon adornment — 3/6 list screens, dominant").
2. **Resolve indirection to real values.** Theme tokens, spacing scales, CSS variables → actual hex/px. The mockup can only use resolved values, so the census stores them resolved (with the token name kept alongside: `gray.500 = #6A7282`).
3. **Cite everything.** Every value and every pattern verdict gets a file:line. This is what makes staleness detectable later.
4. **Record the traps.** Components that exist but are effectively unused (0–1 real usages) are the most dangerous references — list them explicitly with a "do not use as reference" note.
5. **Parallel subagents are fine** for the survey (one per section below); accuracy beats token thrift. Each returns only facts + file:line.

## What to measure (census sections)

### 1. Stack & styling standard
- Framework, styling mechanism actually dominant (inline `sx` / styled-components / Tailwind / CSS modules — count files per mechanism), theme/token entry point, i18n or hardcoded strings.

### 2. Design tokens (resolved values)
- Brand/primary palette, gray scale, semantic colors — token name → actual hex, from the theme source file.
- Typography: font family, the sizes/weights actually used on pages.
- Layout constants: header height, sidebar width, standard card radius, standard page padding.

### 3. Shared components, ranked by usage
| Component | Location | Purpose | Used by (count) |
|---|---|---|---|
- Rank by `grep -rl` count across the pages dir. Include the workhorses (page wrapper, table, dialog, pagination, empty state…).
- **Traps subsection**: shared components with ~0 real usages, and what screens actually do instead (with file:line).

### 4. Screen-type skeletons
For each screen type the codebase actually has (list / detail / form / dashboard — or its own taxonomy):
- The dominant skeleton (which wrappers, in what order), with the usage count that makes it dominant ("8 of 10 detail screens"), and known deviations.
- Navigation pattern between them (route-per-screen vs modal, back-navigation convention).

### 5. Recurring element patterns (the expensive part — cache pays off here)
For the elements new screens most often need, record the dominant pattern + measured values + source:
- **Search input** (control, size, radius, debounce or not, page-reset behavior)
- **Single-select / multi-select filters** (chips vs select vs radio — and in *which context* each is legitimate; e.g. "RadioGroup appears only in form dialogs — never in list filter bars")
- **Date/period filter** (presets, custom range control)
- **Data table** (header bg/height/font, row height, borders — including which properties are *absent*: outer border? radius? shadow?)
- **Pagination, total count, reset button, empty/loading/error states**
- **URL/query-param driven state** (deep-link convention, if any)

### 6. File & naming conventions
- Directory-per-feature? Suffix conventions (`*List.tsx`, `*Detail.tsx`)? These become the grep anchors for reference search.
- Data-fetch pattern (hooks layer? services? inline fetch?).

### 7. Registration points
- Everything a new screen must be registered in: route table (file + nesting convention + permission wrappers), navigation/sidebar (file + entry shape), anything else (menu configs, breadcrumbs). **Missing one of these is the most common integration bug — list them exhaustively.**

## Cache discipline

- Header of the census file: survey date + repo commit hash at survey time.
- Before *relying* on a cached entry in a plan, spot-check its file:line still matches; if it doesn't, re-measure that section and update the census (append a dated one-line change note).
- A full re-survey is rarely needed — sections drift independently.
