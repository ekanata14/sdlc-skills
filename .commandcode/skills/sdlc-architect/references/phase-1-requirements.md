# Phase 1 — Requirement & Tech Stack Alignment

Goal: produce a requirements document you could hand to another engineer, with the stack confirmed by the user rather than inferred, and with every ambiguity either resolved or labelled `Assumed:`.

## Stack confirmation

If `docs/sdlc/<slug>/00-brief.md` exists, start from its §8 answer and confirm it — the brief states intent, the repo states reality, and the two are allowed to disagree.

Propose **one** stack. Evidence first, then a yes/no question. Never open with a menu of five options.

| Repo evidence | Propose | Confirmation question |
|---|---|---|
| `composer.json` with `laravel/framework` + `livewire/livewire` | Laravel + Livewire + Tailwind | "Laravel 11 + Livewire 3 + Tailwind, MySQL/Postgres as per `.env.example`. Correct?" |
| `composer.json` with `laravel/framework` only | Laravel API + chosen frontend | "Laravel as the API; frontend is SvelteKit per `package.json`. Staying there?" |
| `composer.json` + `package.json` with `@sveltejs/kit` | Laravel API + SvelteKit SPA/SSR | Name the boundary: is SvelteKit talking to Laravel over HTTP, or is this a separate app? |
| `package.json` with `express` | Express + chosen view layer | Confirm persistence, auth strategy, and whether it is TypeScript |
| `package.json` with `react` + `vite`/`next` | React (Vite or Next) + API | Confirm the API base and whether auth is token or cookie |
| `docker-compose.yml` + `nginx/*.conf` | Existing topology — reuse it | "New service goes behind the existing nginx vhost. Right?" |
| Nothing (empty repo) | Ask, one question | "Greenfield: Laravel+Livewire or SvelteKit+Express? What's the persistence layer?" |

If the brief names a stack the repo contradicts, surface the conflict and ask which wins. Never resolve it yourself.

If two stacks are genuinely both viable and the brief is silent, present **two** with a one-line trade-off each and ask which — no more than two.

## Clarification batch

At most 4 questions, numbered, in one message. Rank by design impact and stop at 4.

High impact (ask first):
- **Actors and permissions** — who can do this, and is authorization per-record, per-tenant, or per-role?
- **State machine** — what states does the core entity move through, and can it go backwards?
- **Money** — currency, rounding, taxes, refunds, and who bears the fee?
- **Data lifecycle** — what happens on delete (hard, soft, archive), and who can see history?
- **External systems** — which are sandboxed, which are live, and what happens when they time out?
- **Concurrency** — can two people act on the same record at once, and who wins?

Medium impact (ask only if budget remains):
- Timezone and date boundaries for reporting
- Notifications: in-app, email, both, digest?
- Import/export requirements and file formats
- Expected volume and any latency budget
- i18n and RTL

Do not ask:
- Anything answerable from the repo
- Framework basics ("do you want validation?")
- Things you can decide with a documented assumption (pagination defaults, index naming)

## Requirement writing rules

- Numbered `FR-1`, `FR-2`, … Each one testable: a reviewer can say pass or fail.
- Avoid "support", "handle", "manage" without a concrete observable behaviour. "Support CSV export" → "An admin can export filtered orders to CSV with columns A,B,C; export is capped at 50k rows."
- Acceptance criteria in Given/When/Then for anything touching money, auth, or state transitions.
- Every `Assumed:` line states the assumption and the consequence if wrong. Example: `Assumed: prices stored in minor units (cents); wrong choice would require a migration and a rounding audit.`
- Non-goals are mandatory. They are what prevents scope creep during Phase 3.

## Non-functional prompts

Produce these with concrete numbers or mark `unbounded`:
- Expected records and requests per month, and peak multiplier
- Latency budget for the primary user action
- Uptime expectations and maintenance windows
- Data retention and backup requirement
- Compliance constraints (PII, payment data, audit logging, data residency)
- Browser/device floor

## Ambiguity checklist

Walk through this silently. Anything unresolved and design-relevant either becomes a question (if budget) or an `Assumed:` line (if not):

- Auth: session vs token; 2FA; impersonation; password reset path
- Roles: static enum or dynamic permissions; who grants them
- Multi-tenancy: shared tables with `tenant_id`, schema-per-tenant, or single-tenant
- Soft deletes vs hard deletes; restoration; cascading deletes
- Money: currency storage, tax, discounts, refunds, partial payments
- Time: storage in UTC, display timezone, business-day boundaries
- Files: where stored, size/type limits, signed URLs, virus scanning
- Notifications: channels, retries, deduplication
- Integrations: credentials handling, idempotency, failure and retry policy
- Audit: who did what, how long retained, is it user-visible
- Search: exact, fuzzy, or full-text; which fields
- Rate limits and abuse protection

## Document

Write `docs/sdlc/<slug>/01-requirements.md` using `assets/templates/01-requirements.md`. Fill every section. Use `none` rather than deleting a section, so reviewers can see it was considered.

Then print the gate 1 card. Nothing after it.
