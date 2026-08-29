# Introduction

This is a template `AGENTS.md` — a standing set of instructions for any AI
coding agent (Claude Code, Cursor, Codex, Copilot, or similar) working in
this repository. It exists so every agent session starts with real project
context instead of guessing defaults, and so conventions stay consistent
whether a human or an agent writes the next line of code.

**How to use this file, per project:**
- Copy this file to the root of a new project (same level as
  `package.json`) before any agent starts working.
- Update the stack list, folder structure, and any library-specific
  sections below to match what's actually installed — this file should
  always describe the real project, not an aspiration. If a library
  listed here isn't in use, remove its section rather than leaving stale
  instructions behind.
- Treat every rule below as binding unless it's explicitly marked as a
  preference or a "use judgment" case — most sections say which they are.
- This file grows with the project. When a new pattern, library, or
  recurring mistake shows up, add a rule for it here rather than
  correcting the same thing repeatedly across sessions.

**For the agent reading this:**
- Read this entire file before writing any code, not just the section
  that seems most relevant — several sections interact (e.g. staging,
  file size limits, and third-party API usage all apply to the same
  feature at once).
- If a request conflicts with something in this file, say so explicitly
  and ask rather than silently picking one side.
- If Next.js auto-generates or re-adds a `<!-- BEGIN:nextjs-agent-rules
  -->` managed block at the top of this file (it does this automatically
  on `next dev` when it detects an AI agent and no such block exists),
  leave it in place — it's Next.js pointing you at its own bundled,
  version-matched docs and is meant to coexist with everything below it.

## Next.js Version Policy

- Always run the **latest stable** Next.js release, not whatever version
  is familiar from training data. Check `package.json` for the installed
  version before writing any Next.js-specific code — don't assume.
- When starting the project or doing a version bump, install with
  `bun add next@latest` and re-check `node_modules/next/dist/docs/`
  afterward, since APIs and conventions can change between majors
  (this is explicitly called out in the managed block above — Next.js
  itself warns that a new major "may differ from your training data").
- Before using any Next.js API, briefly confirm it still exists and works
  the way you expect in the installed version via the bundled docs —
  don't rely on memory of an older App Router API (e.g. Server Actions,
  caching directives, and config options have all changed across recent
  majors).
- When a breaking change forces a different pattern than what's already
  in the codebase, don't silently leave both old and new patterns mixed
  in — flag it and propose migrating the rest, or note it clearly in
  `PROGRESS.md` as follow-up work.
- Stay on stable releases for this project — don't adopt a canary/RC
  build unless a specific feature is explicitly needed and the trade-off
  is discussed first.

# Project Agent Instructions

## Stack Decision: Choosing Your API Layer

This template's default stack includes Hono as the API layer. Use the
table below once, at project start, to decide if that default applies —
don't revisit or second-guess the decision mid-project.

| If the project has... | Use |
|---|---|
| Only this Next.js app calling its own backend, no webhooks, no other clients planned | **Next.js Route Handlers + Server Actions only.** Skip Hono entirely. Remove the Hono line from the stack list and the "Hono" subsection under Architectural Patterns. |
| Any one of: a mobile app or public API planned, incoming webhooks (payments, email delivery, OAuth callbacks), per-route middleware needs (custom rate limits, custom CORS), or real abuse/scale risk (voting, submissions, user-generated content) | **Hono**, per the sections below. |

If it's unclear which row applies, ask the person building the project
the specific question that resolves it (e.g. "will anything besides this
app's own pages call this backend?") — then apply the table. Once
decided, build accordingly and don't relitigate the choice later in the
project.

This same table logic applies to every stack item below: Cloudflare R2,
Mailgun, Framer Motion, and shadcn are all defaults for a typical
full-stack app, not requirements. If a project has no file uploads,
delete the Cloudflare R2 section; no transactional email, delete Mailgun;
no need for animation, delete Framer Motion. Decide once per item at
project start, then move on.

You are working on a production app built with:

- **Runtime:** Bun
- **Framework:** Next.js (App Router) + TypeScript — always the latest
  stable release, see note below
- **API layer:** Hono (mounted as a Next.js route handler, or standalone —
  see "Architectural Patterns")
- **Database:** Neon (serverless Postgres) via Drizzle ORM
- **Validation:** Zod
- **Auth:** Better-Auth
- **Object storage:** Cloudflare R2
- **Email:** Mailgun
- **Styling:** Tailwind CSS
- **Animation:** Framer Motion (`motion` package)
- **Icons:** Iconify (`@iconify/react`) — never hand-draw or hand-code icon
  SVGs

Prioritize clarity and long-term maintainability over cleverness. When in
doubt, choose the option a new teammate could understand in under a minute.
Never introduce a new library, pattern, or abstraction to solve something
this stack already handles — ask first.

## Staged Development & Progress Tracking

Don't attempt a large feature or task in one uninterrupted pass. Break it
into stages, complete and verify one before moving to the next, and keep a
running record so progress survives across sessions.

**Working in stages**
- Before starting a non-trivial task, briefly outline the stages you'll
  work through (e.g. "1. schema + migration, 2. Zod schema + query
  functions, 3. Hono route, 4. UI, 5. tests") rather than writing
  everything at once.
- Finish and sanity-check one stage (it compiles, the test passes, the
  route responds) before starting the next. Don't leave a stage half-done
  to jump ahead.
- If a task is small (a one-line fix, a copy change), staging is
  unnecessary overhead — use judgment. Staging is for work that touches
  multiple files/layers (schema → query → route → UI).

**Verify before building on top — don't stack unverified work**
The goal of staging is to avoid discovering a foundational problem after
three more features were built on top of it, which then all need
reworking. To actually prevent that:
- A stage isn't "done" because the code compiles or looks right — it's
  done once you've actually run it (a test, a manual call, a rendered
  page) and confirmed the behavior, not just the syntax.
- Before building a new feature on top of an existing piece (a query, a
  route, a shared component), do a quick check that the existing piece
  still behaves as expected — don't assume last session's work is still
  correct just because it was marked done in `PROGRESS.md`.
- When a new feature reveals a flaw in something underneath it, fix the
  root cause at its layer immediately rather than patching around it at
  the layer you're currently working in. A workaround in the UI for a bad
  query is exactly the kind of thing that causes repeated backtracking
  later.
- Check integration points explicitly at each boundary — e.g. after
  adding a Hono route, actually confirm the exact response shape the
  frontend will consume, rather than assuming and finding out when the UI
  stage breaks.
- Prefer writing the test for a stage as part of that stage, not deferred
  to a final "add tests" pass — a test written right after the code is
  what catches a regression before the next feature is built on top of
  the bug.

**PROGRESS.md**
- Maintain a `PROGRESS.md` at the project root as a running log of what's
  been done, what's in progress, and what's next. Update it as you
  complete each stage, not just at the end of a session.
- Keep entries short and scannable — this is a log, not documentation.
  Format:
  ```md
  ## 2026-08-29 — Invoice export feature
  - [x] Add `invoices.exportedAt` column + migration
  - [x] `getInvoicesForExport` query in lib/db/queries/invoices.ts
  - [ ] Hono route: POST /api/invoices/export
  - [ ] UI: export button + optimistic pending state
  Next: wire up the Hono route, then the UI button.
  ```
- Newest entries at the top. Don't delete old entries — they're a
  history of what's shipped, useful when picking work back up later or
  onboarding someone new.
- When picking up a task, check `PROGRESS.md` first for unfinished items
  before starting something new from scratch.
- `PROGRESS.md` is committed to the repo, not gitignored — it's project
  history, not a scratch file.

## Folder Structure

Organize by **feature**, not by file type. Only put something in a shared
top-level folder if it's genuinely used across 3+ features.

```
app/
  (marketing)/
    pricing/
      page.tsx
  dashboard/
    settings/
      page.tsx
      _components/            # private, route-only components
  api/
    [[...route]]/
      route.ts                # Hono app mounted here (catch-all)

server/                        # Hono API implementation, framework-agnostic
  routes/
    users.ts
    invoices.ts
  middleware/
    auth.ts                    # Better-Auth session middleware for Hono
  index.ts                     # Hono app assembly, exported for the route handler

lib/
  auth/
    index.ts                   # Better-Auth server instance/config
    client.ts                  # Better-Auth client for use in components
  db/
    index.ts                   # Drizzle client (Neon connection)
    schema/
      users.ts
      invoices.ts
      index.ts                 # barrel export of all tables
    queries/                   # reusable query functions, grouped by domain
      users.ts
  storage/
    r2.ts                      # R2 client + upload/download/delete helpers
  email/
    mailgun.ts                 # Mailgun client
    templates/                 # email templates (React Email or plain HTML)
  schemas/                     # Zod schemas, grouped by domain
    user.ts
    invoice.ts
  env.ts                       # Zod-validated env var schema, single source of truth
  errors.ts                    # shared AppError type/class
  utils.ts                     # small pure helpers only, no business logic

drizzle/
  migrations/                  # generated Drizzle migrations, never hand-edited

types/
  index.ts                     # barrel export
  user.ts                      # domain types NOT already covered by a Zod
  invoice.ts                   # schema's z.infer<> (see "TypeScript Types")
  api.ts                       # shared request/response shapes for Hono <-> client

components/
  ui/                          # shared, generic, reusable primitives (built
                                # from scratch — see "UI Components")

PROGRESS.md                    # running log of completed/in-progress/next
                                # work — see "Staged Development & Progress
                                # Tracking"
```

Rules:
- Business logic (queries, R2 operations, email sending) never lives inside
  a component or route handler directly — it lives in `lib/` and is called
  from there.
- Hono routes in `server/routes/` stay thin: parse/validate input, call a
  `lib/` function, return a response. No business logic inline in a route.
- Drizzle schema files are the single source of truth for table shape —
  never define the same shape twice in a separate type.
- Do not create new top-level folders without proposing it first.

## Readability Standards

- Optimize for the reader, not the writer. A few extra lines of clear code
  beats a dense one-liner.
- Functions do one thing. If describing it needs "and," split it.
- Soft limits (flag if exceeding, don't silently ignore): components ~150
  lines, functions ~40 lines, files ~300 lines.
- Name things for what they are, not how they're implemented
  (`getActiveUsers`, not `queryUsersWhereStatusFlag`).
- No magic numbers/strings — extract to named constants.
- Avoid nesting beyond 3 levels; prefer early returns / guard clauses.
- Comments explain *why*, not *what*.
- No commented-out code left in commits.

### File size — split, don't dump

Never let one file accumulate everything related to a feature. These are
hard limits, not suggestions — if you're about to exceed one, stop and
split instead of pushing through:

- **Components:** ~120 lines. If a component is growing past this, pull out
  sub-sections into their own components (even small, single-use ones) in
  a co-located file or `_components/` folder. A component file should
  describe *one* piece of UI, not a whole page's worth of markup.
- **Functions:** ~30–40 lines. If a function needs a comment to separate
  "step 1 / step 2 / step 3," those steps are probably separate functions.
- **Route handlers (Hono) and Server Actions:** thin by definition — parse
  input, call a `lib/` function, return a response. If a handler is doing
  more than that, the extra logic belongs in `lib/`.
- **Files overall:** ~250–300 lines is a signal to split, not a hard wall
  to hit exactly. When a file crosses it, look for a natural seam
  (a sub-component, a helper module, a second concern) and extract it.

When splitting, prefer splitting by *responsibility* over splitting
arbitrarily by line count — e.g. `invoice-form.tsx` +
`invoice-form-line-items.tsx` + `use-invoice-form.ts`, not
`invoice-form-part-1.tsx` / `invoice-form-part-2.tsx`.

## Architectural Patterns

**Next.js**
- Server Components by default. Add `"use client"` only when the component
  needs interactivity, browser APIs, or hooks.
- Pages fetch data directly in Server Components (via `lib/db/queries`) when
  the data is simple and page-specific. Use the Hono API layer when the
  same logic needs to be reused across the web app and external/mobile
  clients, or needs its own middleware chain (e.g. rate limiting, webhooks).

**Hono**
- One Hono app assembled in `server/index.ts`, mounted into Next.js via the
  catch-all route handler at `app/api/[[...route]]/route.ts`.
- Group routes by domain (`server/routes/users.ts`, etc.) and compose them
  onto the main app with `.route()`.
- All Hono route handlers validate input with Zod (`@hono/zod-validator` or
  manual `.parse()`) before touching the database or any external service.
- Auth-gated routes use a shared Better-Auth middleware — never re-check
  sessions ad hoc inside individual handlers.

**Drizzle / Neon**
- All queries go through Drizzle — no raw SQL unless Drizzle genuinely
  can't express it, and if so, isolate it in `lib/db/queries` with a comment
  explaining why.
- Schema changes always go through `drizzle-kit generate` — migrations are
  generated, never hand-written or hand-edited.
- Reusable queries live in `lib/db/queries/<domain>.ts`, not inlined in
  routes or components. A query function should be named for the question
  it answers (`getInvoicesForUser`, not `dbQuery1`).

**Zod**
- Every external input boundary (form submission, Hono route body/params,
  webhook payload, R2 upload metadata) is validated with a Zod schema
  before use.
- Schemas live in `lib/schemas/`, one file per domain, and are the source
  of truth for the corresponding TypeScript type via `z.infer<>` — don't
  hand-write a parallel interface.

**Better-Auth**
- Auth config and server instance live in `lib/auth/index.ts` only — never
  instantiate Better-Auth elsewhere.
- Session checks in Server Components use the server instance directly;
  Hono routes use the shared middleware; client components use
  `lib/auth/client.ts`.
- Never roll custom session/JWT handling alongside Better-Auth — if
  something's missing, extend Better-Auth's config/plugins first.

**Cloudflare R2**
- All R2 access goes through `lib/storage/r2.ts` (upload, signed URL
  generation, delete). No direct S3-client calls scattered elsewhere.
- Never expose R2 credentials to the client — uploads happen via a
  server-generated signed URL or a server-side route, never direct
  client-to-R2 with static keys.
- Validate file type/size (Zod or manual checks) before generating an
  upload URL.

**Mailgun**
- All email sending goes through `lib/email/mailgun.ts`. Templates live in
  `lib/email/templates/`, kept separate from send logic.
- Don't inline HTML strings for emails in route handlers or components.

**UI Components**
- Build components from scratch in `components/ui/` for full control over
  markup, styling, and behavior — do not pull in shadcn/ui or a similar
  component library by default.
- shadcn (or another library) may be used only if explicitly requested for
  a specific case, and even then, treat its output as a starting point to
  adapt, not a dependency to leave untouched.
- Style with Tailwind utility classes directly in JSX by default. Extract
  a class string to a variable or a `cva`-style variant helper only once a
  component has several visual variants — don't prematurely abstract.
- Fall back to vanilla CSS (a co-located `.module.css` file) when Tailwind
  genuinely can't express what's needed, or when forcing it into utility
  classes would be noticeably harder to read/maintain than a few lines of
  real CSS. Examples: complex keyframe animations Framer Motion doesn't
  cover, intricate `:has()`/sibling selectors, gradient masks, or
  fine-grained print styles. This is an exception for genuine limitations,
  not a way to avoid learning a Tailwind utility — if there's a
  reasonably direct Tailwind equivalent, use it instead.
- When using vanilla CSS, co-locate it with the component
  (`invoice-chart.module.css` next to `invoice-chart.tsx`), use CSS
  Modules (not global stylesheets) to avoid class name collisions, and
  leave a short comment on why Tailwind wasn't used for that piece.
- Keep components accessible by default: semantic HTML elements, proper
  `aria-*` attributes, visible focus states — don't rely on a library to
  provide this for you since we're not using one.

**Icons**
- All icons come from Iconify (`@iconify/react`'s `<Icon icon="..." />`)
  — never hand-write icon SVGs or copy-paste one-off SVG markup.
- Pick one or two icon sets for visual consistency (e.g. `lucide` or
  `heroicons` via Iconify) rather than mixing icon sets across the app.
- Wrap `<Icon>` in a shared component if you need consistent sizing/color
  defaults across the app, rather than repeating props everywhere.

**Animation**
- Use Framer Motion (`motion`) for transitions, layout animation, and
  gesture-driven interaction — not raw CSS keyframes for anything beyond a
  trivial hover state.
- Keep animation logic out of business-logic components: a component that
  fetches or mutates data shouldn't also own complex motion variants —
  wrap the animated presentation in its own small component.
- Respect `prefers-reduced-motion` for non-essential animations.

**General**
- State management: built-in React state/context first. No new state
  library without explicit sign-off.
- Errors are handled explicitly (`error.tsx`, try/catch with meaningful
  messages, typed error responses from Hono) — no silent failures.

## Loading & Optimistic UI

Skeleton/Suspense and optimistic updates solve different problems — use
the right one for the situation, not whichever is more familiar.

**Skeletons + Suspense — for reads (first-load / navigation)**
- Use when the user is waiting to see data for the first time: page loads,
  tab switches, paginated lists.
- Implement via Next.js `loading.tsx` files and `<Suspense>` boundaries —
  don't hand-roll loading state with `useState` when the framework
  primitive covers it.
- A skeleton must match the real content's exact dimensions (height,
  grid, spacing) so swapping in real data causes zero layout shift. A
  skeleton that doesn't match the final layout is worse than a spinner.

**Optimistic updates — for writes (mutations/actions)**
- Use when the user takes an action and the likely result is already
  known: toggling, liking, marking complete, submitting a comment.
- Implement with React's `useOptimistic` paired with a Server Action (or
  a Hono mutation), not manual local-state juggling.
- Reserve optimistic updates for actions with a low, acceptable failure
  rate. For actions with real failure modes worth surfacing clearly
  (payments, file uploads to R2, anything Better-Auth-gated with
  meaningful consequences), wait for confirmation and show explicit
  pending/error states instead.
- Always handle the rollback path — if the mutation fails, revert the
  optimistic state and surface the error (via the shared `AppError`
  shape), don't leave the UI showing a result that didn't actually happen.

**Rule of thumb:** loading existing data → skeleton. Acting on existing
data → optimistic. Don't reach for optimistic updates just to avoid
building a skeleton, and don't skeleton-wrap something that should feel
instant.

- Centralize types in the `types/` folder — don't scatter one-off
  `interface`/`type` declarations across component files.
- If a type already comes from a Zod schema (`z.infer<typeof userSchema>`),
  don't redeclare it in `types/` — re-export the inferred type from
  `types/` instead so there's one place to import it from:
  ```ts
  // types/user.ts
  export type { User } from "@/lib/schemas/user";
  ```
- `types/` is for types that aren't derived from a schema: shared
  request/response shapes for Hono endpoints, component prop types shared
  across multiple components, and generic utility types.
- A component's own props type (used only by that component) can stay
  local to the component file — it doesn't need to move to `types/`.
- Never use `any`. Use `unknown` and narrow, or define the proper type.

## Environment Variables & Secrets

- All env vars are declared in `.env.local` for local dev (never committed —
  confirm it's in `.gitignore`) and in the hosting platform's dashboard
  (Vercel/Cloudflare) for staging/production.
- Maintain an `.env.example` with every required key present but empty/
  placeholder values, kept in sync whenever a new env var is added.
- Validate env vars at startup with a Zod schema (`lib/env.ts`) — fail fast
  with a clear error if something required is missing, rather than letting
  a `undefined` leak into a Drizzle/R2/Mailgun client at runtime.
- Naming: `SCREAMING_SNAKE_CASE`, prefixed by concern where it helps
  disambiguate (`DATABASE_URL`, `R2_ACCESS_KEY_ID`, `R2_BUCKET_NAME`,
  `MAILGUN_API_KEY`, `BETTER_AUTH_SECRET`).
- Client-exposed env vars (Next.js `NEXT_PUBLIC_*`) must never contain
  secrets — only public config (e.g. a public bucket URL). Double-check
  before prefixing anything with `NEXT_PUBLIC_`.
- Secrets are never logged, never included in error messages returned to
  the client, and never hard-coded as a fallback default in code.

## Security Practices

- Every Hono route that mutates or reads user-specific data checks
  ownership/authorization explicitly — never trust a client-supplied
  `userId`/`id` param without verifying it belongs to the authenticated
  session.
- Rate-limit sensitive routes (auth endpoints, file uploads, email-sending
  routes) at the Hono middleware level — don't rely on Cloudflare alone.
- Validate file uploads for both MIME type and size before generating an
  R2 signed URL — a Zod check on the declared type isn't enough on its
  own; verify server-side where feasible.
- Set CORS explicitly on the Hono app (allowed origins, methods, headers)
  — never wildcard `*` in production.
- Escape/sanitize any user-generated content rendered as HTML (e.g. in
  emails or dashboards) to prevent injection — don't assume Zod validation
  alone covers this.
- Never construct SQL manually from user input, even for the rare raw-SQL
  case — use Drizzle's parameterized query builders.
- Treat all Better-Auth session/cookie handling as security-critical: don't
  modify its cookie/session config without understanding the security
  implication, and don't build a parallel auth mechanism alongside it.

## Error Handling & Logging

- Define a shared `AppError` type/class (`lib/errors.ts`) with a `code` and
  a user-safe `message` — throw/return this instead of raw `Error` objects
  or ad hoc string messages.
- Hono error responses follow one consistent JSON shape across every route
  (e.g. `{ error: { code, message } }`) so the frontend can handle errors
  generically instead of per-endpoint.
- Never leak internal details (stack traces, raw DB errors, raw Mailgun/R2
  provider errors) to the client — log the full detail server-side, return
  a sanitized message to the client.
- Use Next.js `error.tsx` boundaries for route-level UI errors; don't let
  unhandled exceptions render a blank page.
- Log server-side errors with enough context to debug (route, user id if
  available, relevant input) but never log secrets, full request bodies
  containing PII, or auth tokens.
- Don't swallow errors silently (`catch {}` with no action) — at minimum,
  log them; if recoverable, handle explicitly, if not, rethrow as an
  `AppError`.

## Performance & Caching

- Be deliberate with Next.js caching: use `fetch` cache options / route
  segment config intentionally rather than accepting whatever the default
  happens to be — comment when a page or fetch is intentionally dynamic
  vs cached.
- Neon is serverless Postgres — be mindful of connection limits. Use a
  single shared Drizzle client instance (`lib/db/index.ts`) with a pooled/
  serverless driver (e.g. `@neondatabase/serverless`), never open a new
  connection per request.
- Batch or paginate large Drizzle queries — never fetch an unbounded list
  and filter/paginate in application code.
- R2 signed URLs should have a sensible, explicit expiry (short for
  one-time uploads, longer for read access as needed) — don't default to
  the library's max expiry without thinking about it.
- Use Next.js `<Image>` for any user-facing images (including R2-hosted
  ones) rather than raw `<img>`, so optimization/lazy-loading is handled.
- Avoid animating layout-affecting properties (width/height/top/left) with
  Framer Motion where a transform-based animation (`x`, `y`, `scale`,
  `opacity`) would achieve the same effect with better performance.

## Concurrency & Efficiency

- **Parallelize independent fetches.** If two or more `await`s in a Server
  Component, Server Action, or Hono route don't depend on each other's
  result, run them with `Promise.all` (or `Promise.allSettled` if partial
  failure is acceptable) instead of awaiting sequentially.
  ```ts
  // Bad: serial round-trips
  const user = await getUser(id)
  const invoices = await getInvoicesForUser(id)

  // Good: parallel
  const [user, invoices] = await Promise.all([
    getUser(id),
    getInvoicesForUser(id),
  ])
  ```
- **No N+1 queries.** Never loop over rows and issue a Drizzle query per
  row. Use a relational `with` query, a `join`, or a single `inArray()`
  lookup instead. Each extra round-trip to Neon adds real latency — treat
  a query-in-a-loop as a bug, not a style nitpick.
- **Know which cache layer you're using:**
  - React's `cache()` — dedupes identical calls within a single render
    pass (e.g. the same `getUser` called from a layout and a page).
  - Next.js `unstable_cache` / `revalidateTag` / `revalidatePath` — cross-
    request caching with explicit invalidation; use this for data that's
    expensive to compute and safe to serve slightly stale.
  - Hono-level response caching — for computed endpoints hit repeatedly
    with the same input (e.g. a public stats endpoint).
  Pick the right one deliberately; don't stack ad hoc caching on top of
  Next.js's own fetch cache without understanding the interaction.
- **Don't block the response on slow side effects.** Sending a Mailgun
  email or another non-critical side effect shouldn't hold up the
  response the user is waiting on. Fire it after returning the response
  (`after()` in Next.js, or a queue) and handle its failure independently
  — a flaky email provider should never fail or slow down the primary
  action.
- **Rate limiter state must be shared, not in-memory.** A counter in a
  module-level variable doesn't work across serverless/edge instances —
  use a shared store (Cloudflare KV, Durable Objects, or Upstash Redis)
  for any rate limit that needs to hold across requests.
- **Stream large responses** instead of buffering the whole payload —
  Next.js streaming SSR for large pages, Hono's streaming helpers for
  large API responses — rather than assembling everything in memory
  before sending.

## Third-Party API Usage & Credit Limits

Third-party services (Mailgun, R2, or any future integration billed by
usage/request count) are a real cost, not just a technical dependency.
Treat their limits as seriously as a rate limit on your own API.

- **Cache before you call.** If a third-party response doesn't change
  often (lookup data, computed results, anything not user-specific and
  real-time), cache it (`unstable_cache`/KV/Redis with a sensible TTL)
  instead of re-fetching on every request. Never call a paid API inside a
  loop or on every render when the result could be cached.
- **Dedupe concurrent identical calls.** If multiple requests could
  trigger the same third-party call at the same time (e.g. several users
  loading a page that hits the same external lookup), dedupe with
  in-flight request coalescing or React's `cache()` rather than firing one
  call per request.
- **Set a request budget per integration.** Know each third-party
  service's rate/credit limit and stay meaningfully under it — build in
  your own internal cap (e.g. via the shared rate-limit store) rather than
  relying on the provider to reject you when you hit the ceiling.
- **Always implement retry with backoff, not retry-in-a-loop.** Transient
  failures get exponential backoff with a max attempt count; never retry
  in a tight loop, which can spike usage and burn through credits fast
  during an outage.
- **Fail gracefully, don't cascade-retry.** If a third-party service is
  down or rate-limiting you, surface a clear `AppError` and stop — don't
  let a retry storm from multiple users compound the problem and burn
  through remaining credits.
- **Log usage-relevant signals**, not just errors: track call volume per
  integration if the provider doesn't expose usage dashboards, so a
  runaway loop or unexpected traffic spike is caught before it exhausts a
  credit limit.
- **Batch when the provider supports it.** Prefer a bulk endpoint (e.g.
  sending multiple emails in one Mailgun batch call, if available) over N
  individual calls when the provider offers a batched alternative.
- **New third-party integrations go through the Dependency Policy above**
  — including a look at the provider's rate limits and pricing tiers
  before wiring it in, not after hitting a cap in production.

## Git & Commit Conventions

- Commit messages follow Conventional Commits: `feat:`, `fix:`, `refactor:`,
  `chore:`, `test:`, `docs:` — with a short imperative summary
  (`feat: add invoice PDF export`, not `updated stuff`).
- **One type of change per commit — never combine them.** A `feat` commit
  contains only the new feature; a `fix` commit contains only the bug fix;
  a `refactor` commit contains only the restructuring; a `style`/`chore`
  commit contains only formatting or config changes. If work on a feature
  turns up an unrelated bug, fix it in its own separate `fix:` commit, not
  folded into the `feat:` commit — even if both changes are small.
- If you catch yourself about to write a commit message with "and" joining
  two different kinds of change (e.g. "add export button and fix typo in
  header"), that's a signal to split it into two commits first.
- Commit as you complete each stage (see Staged Development) rather than
  batching multiple stages into one commit at the end — this keeps history
  readable and makes a later revert or bisect actually useful.
- Branch naming: `type/short-description` (`feat/invoice-export`,
  `fix/auth-redirect-loop`).
- Keep PRs scoped to one concern — a feature, a fix, or a refactor, not a
  mix. If a change grows to touch unrelated areas, split it.
- Never commit directly to `main`/`production` — always via a
  reviewed PR, even for small fixes.
- Don't commit generated artifacts (`.next/`, `node_modules/`, build
  output) — confirm `.gitignore` covers them.

## Dependency Policy

- Before adding a new package, check whether the existing stack (Bun,
  Next.js, Hono, Drizzle, Zod, Better-Auth, Tailwind, Framer Motion,
  Iconify) already solves the problem — most needs should be met without
  a new dependency.
- Propose new dependencies explicitly (name + why + what it replaces or
  adds) rather than installing silently as part of an unrelated task.
- Prefer well-maintained, widely-used packages with active releases over
  niche or unmaintained ones — check last-publish date and download counts
  before adding.
- Pin dependency versions deliberately; don't introduce a dependency with
  a wide-open version range without reason.

## Documentation

- Every exported function/type in `lib/`, `server/`, and `types/` gets a
  short doc comment (what it does, params, and anything non-obvious about
  behavior) — internal, unexported helpers don't need this unless the
  logic is genuinely non-obvious.
- Document *why*, not *what the code already says* — a doc comment
  restating the function signature in prose adds nothing.
- Keep a root `README.md` current with setup steps (env vars, `bun
  install`, `bun run dev`, running migrations) — update it whenever a
  setup step changes.
- Non-obvious architectural decisions (e.g. "Hono is mounted inside
  Next.js instead of standalone because X") get a short note in this file
  or a linked doc, not just left as tribal knowledge.

## Naming Conventions

- Files: `kebab-case.ts(x)` (e.g. `user-profile-card.tsx`,
  `invoice-schema.ts`).
- Components: `PascalCase` matching the default export (`UserProfileCard`).
- Hooks: `camelCase` prefixed with `use` (`useAuthStatus`).
- Hono route files: named after the resource, plural (`users.ts`,
  `invoices.ts`).
- Server Actions / query functions: verb-first, descriptive
  (`createInvoice`, `getInvoicesForUser`) — never `handler`, `doThing`, or
  `helper`.
- Drizzle tables: `camelCase` variable name, plural
  (`export const users = pgTable("users", ...)`).
- Zod schemas: suffix with `Schema` (`userSchema`, `createInvoiceSchema`);
  inferred types drop the suffix (`type User = z.infer<typeof userSchema>`).
- Types/interfaces: `PascalCase`, no `I`/`T` prefix.
- Booleans: prefix with `is`, `has`, `should`.
- Constants: `SCREAMING_SNAKE_CASE` for true constants/config; otherwise
  `camelCase`.
- Named exports by default. Default exports only where the framework
  requires them (`page.tsx`, `layout.tsx`, `route.ts`).
- Files in `types/`: `kebab-case.ts`, named after the domain (`user.ts`,
  `invoice.ts`, `api.ts`), one barrel `index.ts` re-exporting the rest.
- Shared animation variants (Framer Motion): suffix with `Variants`
  (`fadeInVariants`, `slideUpVariants`).
- Vanilla CSS fallback files: `kebab-case.module.css`, matching the
  component name (`invoice-chart.tsx` → `invoice-chart.module.css`).

## Testing Expectations

Three distinct levels of testing apply here — know which one a given
piece of work actually needs rather than defaulting to one or skipping
the question entirely.

**Unit tests — always, for logic**
- Use Bun's built-in test runner (`bun test`) unless the project has
  already standardized on something else.
- Every `lib/db/queries` function, every Zod schema, and every pure
  helper needs a unit test covering the happy path and at least one
  failure/validation case.
- Mock external services at the boundary — R2 client, Mailgun client, and
  Better-Auth session checks are mocked in unit tests; don't hit a real
  external service or a real Neon database here.
- Zod schemas: test that invalid input is rejected, not just that valid
  input passes.
- This is the default, minimum bar for any new logic. Skipping unit tests
  isn't a judgment call — write them.

**Integration tests — for anything crossing a real boundary**
- Use when a test needs to exercise a Hono route end-to-end, a real
  database query against actual Postgres, or the interaction between two
  internal layers (e.g. a route calling a query calling the DB).
- Use a separate test/branch database (e.g. a Neon branch) for these —
  never run integration tests against the production or shared dev
  database.
- Every Hono route needs at least one integration test hitting the real
  route (not just the underlying function in isolation).
- Needed for: any new API route, any auth-gated flow, any Drizzle query
  with joins or relational complexity worth verifying against a real DB.

**E2E tests — only for critical user-facing flows**
- Use a browser-driving tool (e.g. Playwright) to test a full flow
  through the actual UI, as a real user would.
- Reserve these for flows where a break would be severe and hard to catch
  otherwise: sign-up/login (Better-Auth), any payment or checkout path,
  file upload (R2), and any flow sending a transactional email (Mailgun)
  end-to-end.
- Don't write E2E tests for every page or every component state — they're
  slow and expensive to maintain. If a unit or integration test can catch
  the same bug, prefer that instead. Ask "would this break silently and
  badly in production without an E2E test?" — if no, skip it.
- New E2E tests are proposed explicitly (which flow, why it needs this
  level) rather than added by default alongside every feature.

**General rules across all levels**
- Don't skip or delete a failing test to unblock a build — fix it or flag
  it explicitly.
- Every bug fix ships with a regression test that fails without the fix.

## Before You Finish

- Re-read the diff as if reviewing someone else's PR.
- Confirm folder placement matches the structure above.
- Confirm no business logic leaked into a component or a thin Hono route.
- Confirm new external input is validated with Zod.
- Confirm Drizzle schema changes have a generated migration.
- Confirm tests exist for new logic and pass locally with `bun test` —
  unit tests for the logic itself, an integration test if a new Hono
  route or DB boundary was added, and an E2E test only if this touches a
  critical flow (auth, payment, upload, transactional email).
- Confirm no file/component/function has silently grown past the size
  limits above — split before finishing, not after.
- Confirm new shared types live in `types/`, not scattered inline.
- Confirm icons use Iconify, not hand-written SVGs.
- Confirm no shadcn/ui (or similar) component was pulled in unless
  explicitly requested.
- Confirm no secrets are logged, hard-coded, or leaked to the client.
- Confirm new/changed env vars are reflected in `.env.example` and the
  `lib/env.ts` schema.
- Confirm errors thrown/returned use `AppError` and don't leak internal
  detail to the client.
- Confirm the commit message follows Conventional Commits, contains only
  one type of change (not a feat mixed with a fix or a refactor), and the
  PR is scoped to one concern.
- Confirm reads use Suspense/skeletons matching real content dimensions,
  and writes use optimistic updates only where a failure is low-stakes and
  a rollback path exists.
- Confirm independent fetches run in parallel (`Promise.all`) and no
  Drizzle query runs inside a loop (N+1).
- Confirm any new third-party API call is cached/deduped where possible,
  has retry-with-backoff (not retry-in-a-loop), and stays under a known
  request/credit budget.
- Confirm `PROGRESS.md` is updated with what was completed and what's
  next before ending the session.
- Confirm each stage was actually verified (run, not just written) before
  the next stage was built on top of it — no assumptions carried forward
  unchecked.
