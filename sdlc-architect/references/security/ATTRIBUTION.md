# Attribution

Vendored reference pack, not written by this project.

- **Upstream:** https://github.com/cloudflare/security-audit-skill
- **Pinned commit:** c1c8a8c (2026-09-14) — "Clarify guidance and full audit modes"
- **License:** MIT, Copyright (c) 2025-2026 Cloudflare, Inc. See `LICENSE`.
- **Vendored on:** 2026-11-06

## Local modifications

Only the filename and frontmatter: the upstream `SKILL.md` was renamed to `README.md` and its YAML frontmatter replaced with a provenance comment, so this directory is a reference pack rather than a discovered skill. Reference content is otherwise verbatim, so the upstream can be re-synced by copying the directory again.

`validate-findings.cjs` and `validate-coverage-ledger.cjs` are upstream's own validators used by the skill's report workflow; their `.test.cjs` siblings are upstream tests and are not needed at runtime.

## How this project uses it

This pack lives at `references/security/` inside the `sdlc-architect` skill. Its entry point is `README.md` (renamed from the upstream `SKILL.md` so the skill loader does not register it as a second skill).

`../../SKILL.md` Phase 4 sends work here for full security audits, pen-test requests, or any change touching authentication, payments, or PII. The sdlc-architect per-change checklist stays in place for day-to-day gating; this pack is the depth pass.

## Updating

```
git clone --depth 1 https://github.com/cloudflare/security-audit-skill.git
cp -R security-audit-skill/skills/security-audit/. <this directory>/
```

Then re-check that Phase 4's handoff still matches the upstream workflow, and update the commit pin above.
