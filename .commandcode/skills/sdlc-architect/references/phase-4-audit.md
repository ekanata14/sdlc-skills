# Phase 4 — Security & Audit

Goal: an audit of *this* change, with evidence. Not a generic checklist dump.

This file is the fast per-change gate. For a genuine audit, pen test, or any change touching auth, payments, or PII, escalate to the security pack at `references/security/README.md` (full audit mode) and route by surface:

| Surface in this change | Depth file in `references/security/` |
|---|---|
| HTTP endpoints, sessions, OAuth, CSRF, CORS | `WEB-PROTOCOL-AND-AUTH.md` |
| Browser-rendered UI, DOM sinks, storage | `CLIENT-SIDE.md` |
| Containers, proxies, IAM, deploy config | `CLOUD-AND-DEPLOYMENT.md` |
| Tenancy, PII, retention, deletion | `DATA-ISOLATION-AND-LIFECYCLE.md` |
| Dependencies, build, release pipeline | `SUPPLY-CHAIN-AND-RELEASE.md` |
| Uploads, queues, parsers, rate limits | `RESOURCE-EXHAUSTION-AND-AVAILABILITY.md` |
| LLM or agent features, prompt handling | `AI-AND-LLM.md` |
| Native/CLI/binary components | `MEMORY-SAFETY-AND-BINARY.md` |
| WebSockets, gRPC, queues, IPC | `PROTOCOLS-RPC-AND-MESSAGING.md` |

Do not load all of them. Load the rows that match.

## How to run the audit

1. List the surfaces this change touches: public endpoints, authenticated endpoints, admin actions, file uploads, background jobs, webhooks, database writes, third-party calls, infrastructure config, documentation.
2. Read only the rows below that match those surfaces.
3. For each: **pass**, **fail**, or **N/A**, plus evidence — file and line, or command and output.
4. Severity every failure. Blocker and High must be fixed or explicitly accepted by the user.

Severity guide:
- **Blocker** — exposed secret, missing authorization, injection, unauthenticated money mutation, destructive migration without backup path
- **High** — missing validation on a trust boundary, missing rate limit on an expensive endpoint, PII in logs, idempotency gap on a payment
- **Medium** — weak defaults, verbose errors in production, overly broad CORS, missing security headers
- **Low** — hardening nits, dependency freshness, comment-level hygiene

## Cross-cutting

| Check | Expected evidence |
|---|---|
| Every endpoint has an authorization decision, not just authentication | test or policy reference per route |
| Input validated at the trust boundary only | FormRequest / zod / express-validator location |
| Output escaped by default; raw HTML rendering justified | grep for raw-output helpers with a reason per hit |
| No secret in source, logs, error pages, or client bundle | `.env.example` diff, grep on the changed files |
| Errors do not leak stack traces or SQL in production | `APP_DEBUG=false` / error handler config |
| Rate limits on auth, search, upload, and money endpoints | limiter definitions |
| Idempotency on money and state transitions | unique key + test for duplicate submit |
| Audit trail for privileged actions | table or log line with actor, target, before/after |
| Migrations are reversible or explicitly irreversible | down() or a documented backfill |
| Dependencies added are necessary and pinned | lockfile diff, advisory check |

## Laravel

| Check | Expected evidence |
|---|---|
| No `$guarded = []`; mass assignment limited by `$fillable` | model diff |
| Validation in FormRequest, not controller | request class referenced in the action |
| Authorization via Policy/Gate, called on every write | `$this->authorize()` or policy middleware |
| Queries scoped to tenant/user; no unscoped `find($id)` on user input | scope usage, route-model binding with scoping |
| `env()` only inside `config/` | grep for `env(` outside config |
| Queued jobs are idempotent and safe to retry | `tries`, `backoff`, unique job middleware |
| Serialized job payloads carry IDs, not whole models | job constructor |
| File uploads validated by MIME + size + storage disk | validation rules, storage config |
| Signed URLs for private files, not public paths | `temporaryUrl` usage |
| `APP_KEY`, DB, and third-party keys not in repo | `.env.example` only |

## Livewire

| Check | Expected evidence |
|---|---|
| Public properties hold no trusted state (role, ownership, price) | `#[Locked]` or re-derivation server-side |
| Authorization re-checked inside every public action | `$this->authorize()` at method top |
| `wire:model` uses `live` only where needed; debounce on text | template diff |
| No N+1 in computed properties or `render` | eager loading in the query |
| File uploads use `WithFileUploads` + validation + `temporaryUrl` | component diff |
| `wire:key` present on list items | template diff |
| Actions that mutate money are guarded against double submit | disabled state + server-side idempotency |

## SvelteKit

| Check | Expected evidence |
|---|---|
| `load` returns no secrets or full internal records | `+page.server.ts` return shape |
| Server-only modules never imported by client code | `$lib/server` placement |
| `$env/static/private` / `$env/dynamic/private` used only server-side | grep in `src/routes` |
| Form actions validate input and re-check authorization | action body |
| CSRF/origin checking applies to state-changing requests | SvelteKit defaults not disabled |
| Auth state read from `locals`, not from a client-supplied header | hooks file |

## React

| Check | Expected evidence |
|---|---|
| No `dangerouslySetInnerHTML` without a sanitizer | grep + justification |
| Tokens not in `localStorage` for a first-party app with cookies available | auth module |
| Route guards are UX only; server enforces | API authorization evidence |
| No secrets in `VITE_*` env (they ship to the browser) | env usage |
| Server state cached with sane invalidation; no PII in URLs | query keys |

## Express

| Check | Expected evidence |
|---|---|
| `helmet` and explicit CORS origin list | app bootstrap |
| Body size limits set | `express.json({ limit })` |
| Every route param and body validated | validator middleware |
| Central error handler returns the shared envelope without stack traces | error middleware |
| Auth middleware applied per router, not conditionally inside handlers | router composition |
| Rate limiting on public and auth routes | limiter config |

## Nginx

| Check | Expected evidence |
|---|---|
| TLS config current; no deprecated protocols | server block |
| `proxy_set_header Host/X-Forwarded-*` set correctly | location block |
| `proxy_buffering off` for SSE/streaming endpoints | location block |
| `client_max_body_size` matches the app's upload limit | both values cited |
| No directory listing; no traversal-prone `alias` | config |
| Static assets cached; HTML not stale | cache headers |
| FastCGI/PHP only passes existing files; path info handled | `try_files` chain |

## Docker

| Check | Expected evidence |
|---|---|
| Multi-stage build; dev deps absent from runtime image | Dockerfile stages |
| Non-root user at runtime | `USER` directive |
| Base images pinned by tag or digest, not `latest` | Dockerfile |
| No secrets in build args, layers, or image history | build section |
| Healthcheck defined; restart policy sane | compose/Dockerfile |
| Writable paths limited; read-only FS where possible | compose `read_only` + volumes |

## Documentation audit

Project docs are deliverables. Check all three, with evidence:

| Check | Expected evidence |
|---|---|
| `docs/architecture.md` exists and was updated during this work | file timestamp or diff, not a claim |
| It contains a component flowchart, a Level 0 DFD, a Level 1 DFD, and an ERD | name the sections |
| The ERD matches the migrations actually written — same tables, same columns, same nullability | side-by-side list |
| Every component in the flowchart exists in the codebase; nothing removed is still documented | grep for each component |
| `docs/deployment.md` exists, names real environments, and its pipeline commands match the CI config | config file cited |
| Runbook commands are executable as written (or marked `proposed`) | run or inspect |
| `docs/README.md` links every file in the set | index vs `ls docs/` |
| `docs/journey.html` exists, starts at the project start, and has an entry for every gate card in `STATE.md` | id sequence vs gate rows |
| Every revision in the journey links back via `revises` to an existing entry | id cross-check |
| The journey JSON parses and the page renders | open it, or parse the JSON block |
| No diagram is a screenshot, ASCII art, or external link | file inspection |
| ADR log gained a row for every decision this change locked | compare with the gate cards |

Severity for failures: missing `architecture.md` or `deployment.md` on a project that has code → **High**. Stale ERD that contradicts migrations → **High**. Missing DFD levels → **Medium**. Broken index link → **Low**.

## Test matrix

| Layer | Cases |
|---|---|
| Unit | Domain rules: price computation, state machine transitions, permission predicate |
| Feature/integration | Happy path, validation failure, unauthenticated, wrong role, cross-tenant access, duplicate submit, external timeout |
| E2E | Primary user journey end to end, plus the one failure path a user will actually hit |

Map each case to `FR-n`. A requirement with no test is a finding, not a gap to mention in passing.

## CI/CD gates

| Stage | Command shape | Fails when |
|---|---|---|
| Lint/format | `composer pint --test` / `npm run lint` | any diff |
| Static analysis | `phpstan` / `tsc --noEmit` | any error at the configured level |
| Tests | `php artisan test` / `vitest run` / `playwright test` | any failure |
| Build | `npm run build` / `docker build` | build error |
| Dependency audit | `composer audit` / `npm audit --audit-level=high` | High or above |
| Container scan | `trivy image` | fixable critical |
| Migration safety | dry run against a production-shaped copy | destructive change without an approved plan |

State which of these already exist in the repo and which are recommendations. Do not claim a pipeline exists without seeing the config file.

## Document

Write `docs/sdlc/<slug>/04-audit.md` using `assets/templates/04-audit.md`, then print the gate 4 card. Nothing after it.
