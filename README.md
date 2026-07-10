# mirror-design 🪞

**English** · [한국어](README.ko.md)

<p align="center">
  <img src="assets/mascot.svg" alt="mirror-design — a lazy developer who copies your existing UI from a mirror (Ctrl+C) and pastes it onto the screen (Ctrl+V)" width="440">
</p>

<p align="center">
  <a href="https://kim-taehan.github.io/mirror-design/"><b>▶ See what it built — live case file</b></a><br>
  <sub>two production-looking screens + an audit, assembled from a real codebase with 0 invented pixels</sub>
</p>

**No designer? Your existing screens ARE the design system.**

mirror-design is a [Claude Code](https://claude.com/claude-code) skill that plans and implements new frontend screens by *mirroring what your codebase already has* — instead of letting the model invent plausible-looking UI that matches nothing.

A mirror invents nothing — it can only show what is already there. This skill builds new screens the same way: every element is a reflection of a proven piece collected across your existing ones.

## The problem

Ask an AI agent to "add an announcements admin page" and you get a screen that is *almost* right: a slightly different table style, a filter bar that exists nowhere else in the app, colors that are close-but-not-your-tokens. Every AI-built screen drifts the UI a little further apart — exactly the job a designer or a design system would have prevented. Most internal tools have neither.

## The idea

The design system you don't have is already encoded in the screens you do have. So:

1. **Decompose** the requested screen into UI elements (skeleton, filter bar, table, badges, pagination, empty states…).
2. **For each element, find the existing screen that implements it best** — by grep-counting usage, not by taste. 5 screens use preset date buttons, 0 use a custom pill? Preset buttons. The pill would be an invention.
3. **Extract measured style values** from the winning source (`file:line`), resolving theme tokens to real hex/px.
4. **Produce a review package**: a reference mapping table (every element → its source, `file:line`), a standalone HTML mockup drawn *only* from measured values, and a paste-ready summary for your team chat.
5. **Stop.** A human reviews the plan. Revisions loop until approved.
6. **Implement** — assembling each element from its mapped source, registering routes/nav, and building until green.

Two hard gates keep it honest:

- **Completeness gate** — every element in the mockup must have a `file:line` source in the mapping table. No source → not drawn; mirror-design stops and asks instead.
- **Sibling-consistency gate** — same-role elements on one screen use the same control. No radio-button filter next to two chip filters just because they came from different reference screens.

## Install

As a Claude Code plugin:

```
/plugin marketplace add kim-taehan/mirror-design
/plugin install mirror-design@mirror-design
```

Or manually, as a plain skill:

```bash
git clone https://github.com/kim-taehan/mirror-design
cp -r mirror-design/skills/mirror-design ~/.claude/skills/        # user-wide
# or: cp -r mirror-design/skills/mirror-design <your-repo>/.claude/skills/   # per-project
```

## Use

Inside your frontend repo:

```
/mirror-design add an announcements admin page: list with category filter + detail view
```

Two explicit entry points:

```
/mirror-design:init            # scaffold .mirror-design/ + build (or drift-check) the census
/mirror-design:audit <path>    # audit screens against the census — verdict + before/after mockup
```

Everything mirror-design produces lives in one workspace: `.mirror-design/` (census and shared mockup chrome are committed team assets; `plan/` working artifacts are ignored via the workspace's own `.gitignore`).

First run in a repo, mirror-design builds a **design census** (`.mirror-design/census.md`): a measured survey of your tokens (resolved to real values), shared components ranked by usage, dominant screen skeletons, recurring element patterns, and registration points — each with `file:line` evidence. Later runs reuse the cache and re-verify entries before relying on them. Commit it; your teammates' runs get faster too.

## What you review

The plan package, before any code:

| Element | Source screen (file:line) | Pattern to reuse |
|---|---|---|
| Page skeleton | `AnnouncementList.tsx:35-38` | Header + ContentContainer |
| Filter bar | `ChatlogList.tsx:264-277` | date presets + search field |
| Category badge | `AnnouncementList.tsx:74-90` | CATEGORY_CONFIG chip |
| Deep link | `DeploymentList.tsx:120-127` | useSearchParams seed |

…plus a standalone HTML mockup (new elements badged `NEW`, revisions badged `CHANGED`), and optionally a screenshot-embedded plan PDF you can drop straight into Slack.

## Scope & philosophy

- Framework-agnostic: React/Vue/Svelte/Angular; MUI/AntD/Tailwind/hand-rolled tokens. mirror-design measures whatever is there.
- **Not a creative design tool.** If your codebase has no precedent for an element, mirror-design won't improvise one — it stops and asks. If your existing screens are ugly, mirror-design will faithfully build you another ugly screen. Consistency is the product.
- Planning and implementation are always separated by a human gate. That gate is the point.

## Acknowledgements

Command structure and audit methodology (severity tiers, false-positive filter, judge-only-by-default) were inspired by [interface-design](https://github.com/Dammyjay93/interface-design) by Dammyjay93 — a craft-first design skill that sits on the opposite side of the philosophy: it designs from trained principles, mirror-design reproduces from measured evidence. No code or text was copied; the structural ideas were adapted to an evidence-only judging standard.

## License

MIT
