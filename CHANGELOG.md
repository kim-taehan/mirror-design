# Changelog

## 0.2.0 — 2026-07-09

- Unified workspace: everything now lives in `.mirror-design/` — `census.md` and `mockup-chrome.html` are committed team assets; `plan/` working artifacts (mockups, screenshots, PDFs) are ignored via the workspace's own `.gitignore`. The separate `mirror-design-plan/` directory is gone.
- New command `/mirror-design:init` — scaffold the workspace and build (or drift-check) the census; ends with a summary + trap report. Asks no design questions.
- New command `/mirror-design:audit` — audit screens against the census: severity-scored findings (Blocker/Should-fix/Note) with file:line evidence on both sides, a census-evidence-only false-positive filter, and a mandatory before/after mockup with `DRIFT` badges. Judges only; fixes enter the standard revision loop on request.

## 0.1.0 — 2026-07-09

- Initial release.
- `mirror-design` skill: plan and implement new frontend screens by mirroring existing ones — design census, per-element reference mapping with file:line provenance, measured-style HTML mockup, human review gate before implementation.
- Input is a free-text screen request (no issue-tracker integration).
