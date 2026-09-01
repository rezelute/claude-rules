# CLAUDE.md — Backend Project Template

Starter conventions for a new Node/TypeScript backend, based on the stack
used in nerdy-nutrient-server. Delete the scenario sections that don't apply
to this project. See `common.md` and `typescript.md` for cross-project
conventions — this file covers backend-specific stuff only.

## Core stack (always)

- **Runtime**: Node + TypeScript, run/dev via `tsx` (`tsx watch src/index.ts`
  for dev)
- **Server**: `fastify`, with `@fastify/helmet` and `@fastify/cors` on by
  default
- **Database**: `prisma` + `@prisma/adapter-pg` against Postgres
- **Validation**: `zod` for all input schemas
- **Env vars**: `dotenv` (+ `dotenv-cli` in npm scripts so one root `.env`
  can serve multiple packages)
- **Lint/format**: `oxlint` (fast first pass) + `eslint`/`typescript-eslint`
  (deeper rules) + `prettier`. Package scripts: `lint`, `lint:errors`,
  `prettier`, `prettier-check`, `tscheck`
- **tsconfig**: path alias `@/*` → `./src/*`; `strict: true`

## API style

### GraphQL

- `graphql-yoga` as the server, `graphql` + `graphql-scalars` for
  schema/scalars
- `@graphql-tools/load` + `@graphql-tools/graphql-file-loader` to load
  `.graphql` SDL files
- `@graphql-codegen/cli` + `typescript` + `typescript-resolvers` plugins,
  driven by a `codegen.ts`, exposed as an npm `codegen` script
- Use when the client needs typed queries/codegen or the data shape is
  nested/graph-like

### REST

- `fastify-type-provider-zod` to bind Zod schemas straight to Fastify routes
- `@fastify/swagger` + `@fastify/swagger-ui` for auto-generated docs
- Use for simple CRUD/webhook-style services where a GraphQL layer is
  overkill

## API Error Responses

Consistent error shape regardless of API style, so the frontend can branch on
a `code` rather than parsing message strings.

### REST

- Every error response is `{ error: { code, message, details? } }`
  - `code`: a stable, machine-readable string (`"VALIDATION_ERROR"`,
    `"NOT_FOUND"`, `"UNAUTHORIZED"`, `"CONFLICT"`, `"INTERNAL_ERROR"`) —
    never changes across deploys, safe for the frontend to switch on
  - `message`: human-readable, safe to display or log
  - `details`: optional — e.g. Zod's flattened field errors on a 400
- Map errors to correct HTTP status codes (400 validation, 401/403 auth,
  404 not found, 409 conflict, 500 unhandled) — don't return 200 with an
  error body
- Zod validation failures are caught by a single Fastify error handler and
  reshaped into the standard envelope, not handled ad hoc per route
- Unhandled/unexpected errors: log the full error server-side (with stack),
  but return only `{ error: { code: "INTERNAL_ERROR", message: "Something
  went wrong" } }` to the client — never leak stack traces, SQL, or internal
  messages in production

### GraphQL

- Use `extensions.code` on GraphQL errors for the same stable
  machine-readable codes as REST (`"VALIDATION_ERROR"`, `"UNAUTHORIZED"`,
  etc.)
- Validation errors (Zod) get mapped to a single `GraphQLError` with
  `extensions.code = "VALIDATION_ERROR"` and field details under
  `extensions.details`, not one error per field with no code
- Mask unexpected/internal errors before they reach the client
  (graphql-yoga's error masking) — log the real error server-side, return a
  generic message + `"INTERNAL_ERROR"` to the client

## Auth

### Projects without login

- Skip `supertokens-node` and `jsonwebtoken` entirely
- No `User`/`Session` models in the Prisma schema
- Any protection is per-route (API key, IP allowlist, `@fastify/rate-limit`),
  not per-user

### Projects with login

- `supertokens-node` for session management + auth methods (email/password,
  social, etc.)
- Only add `jsonwebtoken` if issuing custom tokens _outside_ SuperTokens
  sessions (e.g. service-to-service or short-lived invite links)
- Prisma schema needs `User` (+ related session/invite tables as needed)
- Standard pattern: a session-verification middleware/plugin that populates
  `request.user`/GraphQL context, and an explicit allow-list of public
  routes/resolvers
- Admin/one-off actions (e.g. inviting a user) go in npm scripts like
  `admin-invite-user`, run via `tsx`, not exposed as HTTP endpoints

## Optional add-ons (only pull in if the project needs them)

- **Transactional email**: `node-mailjet` — add a `preview:emails` script if
  templates are non-trivial
- **Error monitoring**: `@sentry/node` — recommended for anything with real
  users/prod traffic
- **Rate limiting**: `@fastify/rate-limit` — recommended for any publicly
  reachable API
- **Spreadsheet/data import**: `xlsx` — only if the project ingests
  spreadsheet data at setup time (e.g. a one-off `gen-*` script under
  `app-setup/`)

## Project layout

```
src/
  app/       # routes/resolvers, feature code
  system/    # cross-cutting infra: mailer, scripts, auth wiring
  utils/
  env.ts     # typed env var loading (zod-parsed)
  index.ts   # server bootstrap
prisma/      # schema + migrations
app-setup/   # one-off setup scripts (codegen input, data generation, first-time-setup.sh)
```

## Prisma script conventions

Mirror local vs. devcontainer variants (`migrate-dev` vs `dc-migrate-dev`,
etc.) so the same commands work on host and in a devcontainer without
editing `.env` paths:

```
migrate-dev / dc-migrate-dev
generate    / dc-generate
seed        / dc-seed
```

## Formatting

Match `.prettierrc`: no semicolons, 3-space indent, double quotes,
`printWidth: 100`, `arrowParens: always`.
