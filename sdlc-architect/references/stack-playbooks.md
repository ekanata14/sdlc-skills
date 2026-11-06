# Stack Playbooks

Load this when designing (Phase 2) or auditing (Phase 4) against one of these stacks. Read only the relevant sections.

## Laravel

**Architecture shape.** Thin controllers, one class per use case in `app/Actions`, orchestration in `app/Services`, policies for authorization, jobs for anything slower than ~200ms or touching a third party. Eloquent models carry relationships and scopes, not business rules.

**Honour.** FormRequest for validation, `authorize()` in the action, `DB::transaction` around multi-write operations, `dispatch()->afterCommit()` for side effects, queued jobs with `tries` and `backoff`, `Model::preventLazyLoading()` in non-prod.

**Pitfalls.**
1. Validation in controllers, so API and Livewire paths diverge and one of them forgets a rule.
2. `$guarded = []` inherited from a tutorial, turning every `fill()` into a mass-assignment hole.
3. Side effects dispatched inside the transaction, so a queue failure rolls back or a rollback still sends the email.
4. `env()` called outside `config/`, which breaks under `config:cache` on deploy.
5. Missing indexes on FKs, so the join works at 1k rows and dies at 500k.

## Livewire

**Architecture shape.** One component per meaningful unit of interaction, state derived from the database rather than trusted from the browser. Actions are the mutation boundary. Anything that can be recomputed should be a computed property, not a public property.

**Honour.** `#[Locked]` on identifiers, `$this->authorize()` at the top of every action, eager loading in `render`, `wire:key` on every looped element, `wire:model.live.debounce` on text inputs, `WithFileUploads` with validation, `$this->dispatch()` for cross-component signals.

**Pitfalls.**
1. Trusting a public property (`public $price`, `public $role`) — the client can set it. Derive it or lock it.
2. Authorization only in `mount()`. `mount` runs once; actions run many times.
3. A computed property that queries per iteration — N+1 disguised as clean code.
4. `wire:model.live` on every field, re-rendering the component on each keystroke.
5. Mutating money without server-side idempotency, so a double-click becomes a double charge.

## Tailwind

**Architecture shape.** Tokens in the config/theme layer, components as Blade/Svelte/React partials rather than repeated utility strings. Variant logic lives in the component, not in string concatenation scattered through templates.

**Honour.** Consistent spacing scale, responsive breakpoints established once, dark mode via the chosen strategy (class or media), focus states that are visible, `@apply` reserved for genuinely repeated primitives.

**Pitfalls.**
1. Arbitrary values (`w-[427px]`) replacing the scale, which makes responsiveness unpredictable.
2. No focus-visible styling, silently breaking keyboard accessibility.
3. Full class names built dynamically (`bg-${color}-500`), which the compiler cannot see.
4. Utility soup duplicated across five templates instead of one partial.

## SvelteKit

**Architecture shape.** `+page.server.ts` for authenticated data and mutations, `+server.ts` for API endpoints, `$lib/server` for anything secret, `hooks.server.ts` for session resolution into `locals`. Client components handle interaction only.

**Honour.** Typed `load` returns with an explicit shape, form actions for progressive enhancement, `$env/*/private` only server-side, `invalidate`/`depends` for cache intent, error pages via `+error.svelte`.

**Pitfalls.**
1. A `load` that returns the whole record including internal fields, leaking them to the client payload.
2. Importing a server-only module into a client component — build error at best, secret in bundle at worst.
3. Authorization only in `load` while form actions mutate without re-checking.
4. `VITE_`-prefixed env vars treated as private; everything prefixed ships to the browser.
5. Fetching Laravel/Express from the client with a token stored in `localStorage` when a server-side proxy would avoid it.

## React

**Architecture shape.** Feature-based: `src/features/<feature>/` owns that feature's `components/`, `hooks/`, `api/`, `types/`, and tests; `src/components/` holds only shared primitives; `src/lib/` holds the query client and the single API wrapper. Server state goes through the query layer, UI state stays local. Routing guards are UX; authorization is server-side.

**Honour.** Typed API responses, explicit loading/empty/error states per view, optimistic updates only where rollback is defined, keys from stable IDs, `useMemo` only with a measured need.

**Pitfalls.**
1. `dangerouslySetInnerHTML` on user content without sanitization.
2. Tokens in `localStorage` in an app that already has cookie infrastructure.
3. Auth checks in route guards, with the API still accepting unauthenticated calls.
4. Fetching in `useEffect` with no cleanup, producing race-conditioned state and double renders.
5. PII in URLs and query keys, ending up in logs and analytics.

## Express

**Architecture shape.** MVC, one layer per concern: `routes/` maps each URL to a controller method, `controllers/` handle request/response only, `services/` hold the business logic multiple entry points need, `models/` own the schema and persistence, `middleware/` carries parse → validate → authorize → error, and one central error handler emits the shared envelope. No business logic leaks into a router file or a controller body.

**Honour.** `helmet`, explicit CORS allowlist, `express.json({ limit })`, validation middleware per route, async wrapper so rejections reach the error handler, rate limiters on auth and expensive endpoints, graceful shutdown.

**Pitfalls.**
1. `async` handlers without a wrapper, so a rejected promise becomes a hung request.
2. Error handler that returns `err.stack` in production.
3. `cors()` with no origin list.
4. Authorization decided inside the handler per route, so one route forgets.
5. Unbounded `find()` responses with no pagination ceiling.
6. A fat controller that talks to the ORM directly, so the same rule has to be re-implemented for the next endpoint or job.

## Nginx

**Architecture shape.** One server block per vhost, TLS termination, `try_files` front-controller pattern, explicit proxy headers, separate locations for static, SSE/streaming, and the app.

**Honour.** `proxy_set_header Host $host` and `X-Forwarded-Proto`, `proxy_buffering off` for SSE, `client_max_body_size` aligned with the app's upload limit, gzip for text assets only, immutable cache for hashed assets, `location = /favicon.ico` short-circuit.

**Pitfalls.**
1. Buffering left on for SSE/Livewire streaming, so events arrive in bursts or never.
2. `client_max_body_size` lower than the app's upload limit — 413s that look like app bugs.
3. Missing `X-Forwarded-Proto`, so Laravel generates `http://` URLs behind TLS and redirect loops.
4. `alias` with a path that allows traversal.
5. Static location ordering that shadows the front controller.

## Docker

**Optional in this workflow.** Read this section only when the repo is already containerised or the user asked for containers. The default run path is local processes (`npm run dev`, native database) — never propose a compose file as the standard way to start a project.

**Architecture shape.** Multi-stage build (deps → build → runtime), non-root runtime user, one process per container, config through env, healthcheck, volumes only for data that must survive.

**Honour.** Pinned base image tags, `.dockerignore` covering `node_modules`/`vendor`/`.git`, build cache friendly layer ordering, `compose` healthchecks with `depends_on: condition: service_healthy`, secrets via file mounts or env injection at runtime.

**Pitfalls.**
1. `latest` base images, so builds are not reproducible.
2. Dev dependencies and compilers in the runtime image, inflating attack surface.
3. Running as root with a writable filesystem.
4. Secrets passed as `ARG`, preserved in image history.
5. No healthcheck, so the orchestrator routes traffic to a container that is still booting.
