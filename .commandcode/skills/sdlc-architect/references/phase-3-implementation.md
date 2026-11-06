# Phase 3 — Development Execution

Goal: full files, in dependency order, commented for intent, verified against the approved design. The formatting here is a hard requirement of this workflow, not a style suggestion.

## Output contract

Every code block follows this shape:

1. **1–3 bullets of logical flow.** Not a paragraph, not a numbered essay.
2. **The full file**, headed by its path on its own line.
3. **At most 3 lines after the code**: what to run and what to check.

### Good

````
- Validate the slot still exists and is free inside a transaction with a row lock.
- Write the booking with an idempotency key so retries cannot double-book.
- Dispatch the confirmation job after commit, so a queue failure cannot roll back the booking.

`app/Actions/CreateBooking.php`

```php
<?php
// ...
```

Next: `php artisan test --filter=CreateBooking`, then re-check the 409 path manually.
````

### Bad

- A diff against an imagined previous version.
- `// ... existing code ...` inside the block.
- "Here's the complete implementation of the booking system:" followed by 400 lines across six files with no file headings.
- A closing paragraph explaining what the code does, since the comments already do that.

### Rules

- **Full files.** Emit the complete file even when the change is six lines — the output must be copy-pasteable and unambiguous. No diff hunks, no elided bodies.
- **No placeholders.** No `// TODO: implement`, no omitted imports, no `...`.
- **Dependency order.** Migration → model/entity → service or action → controller/endpoint/Livewire action → UI → routes/providers/config → seeds. Wait for nothing; present them in one response unless the user asked for incremental delivery.
- **One path heading per file.** No diff hunks, no line numbers.
- **Match the repo.** Naming, formatting, return types, and test style follow the existing code, even where you would choose differently on a greenfield project.

## Inline comments

The rule for this workflow: comment **why**. Syntax is not worth a comment; decisions, constraints, and traps are.

Required comment sites:
- Every transaction boundary — why this scope, what must be atomic
- Every authorization check — who is excluded and what would leak otherwise
- Every non-obvious branch — the business rule that causes it
- Every external call — timeout, retry, and what happens on failure
- Every deliberate deviation from the obvious approach — the reason

Forbidden:
- Restating the code (`// increment counter`)
- Framework explanation (`// Laravel validates this via FormRequest`)
- Section banners with no information
- Comments that will be false after the next refactor

```php
// Row lock + uniqueness on (slot_id) is what makes double-booking impossible under
// concurrent requests; checking availability before the transaction would race.
$slot = Slot::whereKey($slotId)->lockForUpdate()->firstOrFail();
```

## Per-stack placement

| Stack | Where things go |
|---|---|
| Laravel | `app/Actions` for one-off domain operations, `app/Services` for multi-step orchestration, `FormRequest` for validation, `Policy` for authorization, `app/Jobs` for async, migrations in `database/migrations` with timestamped names |
| Livewire | Component class beside its Blade view in `app/Livewire` + `resources/views/livewire`; authorization at the top of every public action; `#[Locked]` on any property that must not be tampered with; computed properties for derived data |
| SvelteKit | `+page.server.ts` for authenticated data loading and form actions; `+server.ts` for API endpoints; shared types in `$lib/types`; never return secrets from `load` |
| React | Feature folder with `api/`, `components/`, `hooks/`; server state in the query layer, UI state local; auth attached in one request wrapper, not per call |
| Express | `routes/` per resource, `controllers/`, `middleware/` for auth + validation + errors; validation at the router boundary, not deep in handlers |
| Nginx | Server block per vhost, TLS and proxy headers, explicit `client_max_body_size`, buffering decisions for SSE/streaming |
| Docker | Multi-stage build, non-root runtime user, healthcheck, secrets via env/secret mount only |

## Verification

- Run the project's lint, typecheck, and test commands. Report what you ran.
- If a command does not exist in the repo, say so instead of inventing one.
- If tests were already failing before your change, distinguish pre-existing failures from regressions.
- Kill any dev server, watcher, or background process you started. Never leave a port occupied.
- Do not claim a test passed if you did not run it.

## Debugging sub-path

Use this for bug reports, failing tests, and stack traces — with or without the four-phase workflow.

1. **Reproduce or explain the exact failure mechanism.** Name the file, the line, and the sequence. "The queue worker reads `created_at` in the app timezone because `config/app.php` sets it to UTC+7 while the column is UTC" — not "there's a timezone issue somewhere".
2. **Fix, one or two lines.** Why this fix and not the neighbouring alternative.
3. **Only the modified snippet.** With enough surrounding context to paste, and nothing else in the file.

Do not restate the whole file, do not list "things to check", do not hedge with three possible causes when you know which one it is. If you genuinely do not know, say so and state the single experiment that would settle it.

## Design drift

When implementation contradicts `02-design.md`:

1. Stop expanding the deviation.
2. Append to `03-implementation.md`: what the design said, what reality is, why.
3. Ask one question: reopen Phase 2, or accept the deviation (recorded as an `Assumed:` decision)?

Small mechanical differences (an extra index, a renamed private method) do not need this. Schema shape, endpoint shape, authorization model, and state machine do. Any deviation that touches the ERD, a flow, or a component also requires updating `docs/architecture.md` before the gate — the docs are wrong the moment the deviation lands.

## Change log

`03-implementation.md` accumulates, per file: path → intent → status (done / partial / blocked). Plus the verification commands run and their outcome, plus any deviation. This is what makes gate 4 auditable.
