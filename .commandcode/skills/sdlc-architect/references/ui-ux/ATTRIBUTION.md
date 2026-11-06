# Attribution

Vendored reference pack, not written by this project.

- **Upstream:** https://github.com/imYChaudhary22/ui-ux-audit
- **Pinned commit:** 9db3658 (2026-07-07) — "docs: add README with multi-agent install guide and capability reference"
- **License:** MIT, Copyright (c) 2026 imYChaudhary22. See `LICENSE`.
- **Vendored on:** 2026-11-06

## Local modifications

Only the filename and frontmatter: the upstream `SKILL.md` was renamed to `README.md` and its YAML frontmatter replaced with a provenance comment, so this directory is a reference pack rather than a discovered skill. `audit.js` is verbatim.

## Runtime requirements

The skill drives a real browser. It needs a browser-automation tool (Command Code ships the `agent-browser` skill for this) or a hand-run pasted script in DevTools. `audit.js` itself is self-contained and makes no network calls — it inspects the page it is injected into.

## How this project uses it

This pack lives at `references/ui-ux/` inside the `sdlc-architect` skill. Its entry point is `README.md` (renamed from the upstream `SKILL.md` so the skill loader does not register it as a second skill).

`../../SKILL.md` Phase 4 runs this pass when a change has user-facing surface and the app can be run locally. The app must be started for the audit and stopped afterwards — do not leave the dev server running.

## Updating

```
git clone --depth 1 https://github.com/imYChaudhary22/ui-ux-audit.git
cp -R ui-ux-audit/skills/ui-ux-audit/. <this directory>/
```

Then update the commit pin above.
