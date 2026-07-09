# Building the visual mockup

The mockup is a **standalone HTML file** a reviewer opens in a browser (or sees as a screenshot in chat). It exists so a human can judge the plan without reading code. It is *evidence*, not art.

## The one rule

**Every style value in the mockup comes from the measured-style table.** If a color, radius, spacing, or font isn't in that table (with its file:line source), it cannot appear in the mockup. A mockup containing an invented value is invalid — that's the failure mode this whole skill exists to prevent.

Corollaries:
- Properties **absent** from the real component are absent from the mockup. Real table has no outer border/shadow → mockup table has none. No decorative upgrades.
- Placeholder content (names, dates, numbers) should look like the domain's real data — copy realistic shapes from existing screens' mock/test data if available.

## Structure

- One self-contained `.html` file per screen: inline CSS, no external requests (reviewers may be offline; files may be shared into chat tools that block remote loads).
- Reproduce the app's **page chrome** (header/sidebar/nav) from census values, so the reviewer sees the screen in context — but keep chrome simplified; the new screen's content area is the subject.
- Build the chrome skeleton **once per repo** from census values and reuse it across mockups (it can live next to the census, e.g. `.mirror-design/mockup-chrome.html`).

## Change badges

- Elements **added** by this plan: a small `NEW` badge.
- In revision rounds, elements **changed** since the last reviewed version: a `CHANGED` badge (visually distinct from `NEW`).
- In audit mockups, elements **deviating** from the census: a `DRIFT` badge (visually distinct from both — audit only).
- Badge styling is the one place invented style is allowed — it's annotation, not design. Keep it obviously an annotation (e.g. small colored tag that no real screen uses).

## File locations

```
.mirror-design/                   # unified workspace in the target repo
├── .gitignore                    # content: "plan/" — keeps working artifacts uncommitted
├── census.md                     # committed — the measured survey
├── mockup-chrome.html            # committed — shared page chrome, built once per repo
└── plan/                         # working artifacts, ignored
    ├── <screen>-mockup.html      # overwrite in place across revisions (stable path)
    ├── <screen>-audit.html       # audit before/after mockup
    ├── <screen>.png              # capture (optional)
    └── <plan>-v<N>.pdf           # versioned plan doc (optional)
```

## Optional: screenshot + plan PDF

If Playwright is available in or near the repo:

```bash
npx playwright screenshot --channel chrome --viewport-size=1600,1000 --full-page \
  "file://$PWD/.mirror-design/plan/<screen>-mockup.html" .mirror-design/plan/<screen>.png

npx playwright pdf --channel chrome --paper-format A4 \
  "file://$PWD/.mirror-design/plan/<plan>.html" ".mirror-design/plan/<plan>-v1.pdf"
```

- `--channel chrome` uses the already-installed Chrome — useful where corporate proxies break `playwright install` downloads.
- The plan document embeds the screenshot plus: background/purpose/scope, per-screen sections (mockup image + element notes + the reference mapping table), data contract (API → displayed fields), implementation plan, and a change-log table. Write it so a reviewer understands each element **without opening code** — what it does, and why that form (source + usage count).
- Re-capture and re-render on every revision; bump the PDF version, keep the HTML paths stable.
