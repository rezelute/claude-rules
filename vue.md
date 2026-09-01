# Vue / Nuxt

Vue/Nuxt-specific conventions. See `frontend-common.md` for framework-agnostic
frontend rules (CSS, accessibility, error handling policy, etc.) — this file
covers Vue/Nuxt-specific stuff only. Sections marked **(optional)** only
apply when the project has that piece.

## Stack

- Nuxt (Vue 3, `<script setup lang="ts">`), TypeScript
- Pinia for state, Tailwind + Nuxt UI for styling/components
- Vitest + @vue/test-utils for tests

## Scripts

- `npm run dev` / `build` / `generate` / `preview`
- `npm run lint` — runs oxlint then eslint (`lint:oxlint`, `lint:eslint`)
- `npm run prettier` / `prettier-check`
- `npm run tscheck` — `nuxt typecheck`
- `npm run vitest`

Run `lint`, `prettier-check`, and `tscheck` before considering a change done.

## Recommended Packages

Default starting set for a new project — pull in what applies, drop what
doesn't.

**Core**

- `nuxt` — the framework (SSR/SSG, routing, auto-imports, module ecosystem)
- `vue` — UI framework Nuxt is built on
- `typescript` / `vue-tsc` — types + type-checking `.vue` files

**State & Utilities**

- `pinia` + `@pinia/nuxt` — global state stores, wired into Nuxt
- `@vueuse/core` — ready-made composables for common browser/reactivity needs
- `lodash-es` — general-purpose utility functions (tree-shakeable ESM build)
- `zod` — schema validation/parsing (form input, API responses, env vars)
- `nanoid` — small, fast unique ID generator

**Styling/UI**

- `tailwindcss` + `@tailwindcss/vite` — utility-first CSS
- `@nuxt/ui` — accessible, pre-built component library built on Tailwind

**Error Tracking**

- `@sentry/nuxt` — captures and reports runtime errors/exceptions (see Error
  Handling below)

**Data Fetching (optional — only if the project talks to a backend)**

- `@urql/vue` + `@urql/exchange-auth` — GraphQL client + auth-aware request
  exchange
- `@graphql-codegen/cli` and its `typescript`/`typescript-operations`/
  `client-preset` plugins — generates typed GraphQL operations from
  `.graphql` files
- `@hey-api/openapi-ts` + `@hey-api/client-fetch` — generates a typed REST
  client from an OpenAPI/Swagger spec

**Auth (optional — only if the project has user accounts)**

- `supertokens-web-js` — auth client (session handling, login flows)

**Dev Tooling**

- `eslint` + `@nuxt/eslint` — Vue/Nuxt-aware linting
- `oxlint` + `eslint-plugin-oxlint` — fast additional lint pass layered under
  eslint
- `eslint-plugin-simple-import-sort` — auto-sorts imports
- `eslint-config-prettier` — turns off eslint rules that fight with Prettier
- `prettier` — code formatting
- `vitest` + `@vue/test-utils` + `@nuxt/test-utils` — component/unit testing
- `happy-dom` / `jsdom` — DOM environment for tests
- `cross-env` — cross-platform env vars in npm scripts
- `npm-run-all2` (`run-s`/`run-p`) — chain/parallelize npm scripts (e.g. the
  `lint` script running `lint:oxlint` then `lint:eslint`)

**Nuxt modules (optional, nice-to-have)**

- `@nuxt/a11y` — accessibility hints/linting during development
- `@nuxt/image` — optimized image loading/resizing; use for any user-uploaded
  or CMS-sourced images (pairs with the `alt` text requirement under
  Accessibility)
- `@nuxt/scripts` — optimized loading for third-party scripts
- `@nuxtjs/sitemap` — sitemap generation, only for public-facing/SEO-relevant
  sites
- `@nuxtjs/turnstile` — Cloudflare Turnstile captcha, only if a form needs bot
  protection

## Vue-Specific Code Style

- Vue: `v-on`/attribute hyphenation off (`@myEvent`, `asChild` not
  `@my-event`, `as-child`)

## Vue SFC Order

Follow `vue-file-structure.md` for `<script setup>` block order: imports →
composables/router/store → props/emits → models → constants → state/queries
→ computed → watchers → methods → lifecycle.

## Project Structure

- `app/components/{features,layout,pages,ui,images}` — components grouped by
  role
- `app/{pages,layouts,middleware,plugins,stores,types,utils,config}` —
  standard Nuxt dirs
- `layers/<n>/` **(optional)** — Nuxt layer for code shared across multiple
  apps/clients (mirrors the same
  `components/composables/middleware/pages/plugins/services/stores/utils`
  shape as `app/`)

## State Management

- Pinia stores: only for holding data and exposing functions/getters derived
  from _that store's own_ data. If a computation needs data from more than
  one store, that logic belongs in a composable, not in a store.
- Use a store when either is true: data needs to pass down through more than
  2-3 layers of components, or the data has a good chance of being needed on
  more than one page/component. In that case, fill the store from the API
  (or a JSON file) once — on app init/before the app mounts — rather than
  re-fetching it per component.
- Otherwise (data used by a couple of components, or scoped to one page),
  prefer local component state or a composable over a store.

## Data Fetching **(optional — only if the project talks to a backend)**

Implements the policy in `frontend-common.md`.

- REST: `@hey-api/openapi-ts`, config in `openapi-ts.config.ts`, run via
  `codegen:openapi`
- GraphQL: `@urql/vue` + `@graphql-codegen/cli`, config in `codegen.ts`, run
  via `codegen:gql`

## Error Handling

Implements the policy in `frontend-common.md` as a composable.

- The composable takes flags for whether to show a toast and whether to
  report to Sentry (`Sentry.captureException`), so call sites can opt out
  selectively.
- New projects without an existing error-handling composable should add an
  equivalent early (e.g. `composables/useHandleError.ts`) rather than
  bolting error handling on ad hoc later.

## Auth **(optional — only if the project has user accounts)**

- SuperTokens (`supertokens-web-js`) for auth; auth UI/composables live under
  `layers/system/components/authentication` when using a shared layer

## SEO **(optional — only if the project is public-facing)**

Implements the policy in `frontend-common.md`.

- Per-page: use `useSeoMeta` inside each page for title/description/OG/
  Twitter tags. Build the OG/Twitter share image URL from the site origin.
- Global fallback: set defaults in `nuxt.config.ts` under `app.head.meta` —
  description, `og:type`, `og:site_name`, `og:title`, `og:description`,
  `twitter:card`, `twitter:title`, `twitter:description`.
- Favicon/manifest links (`icon`, `apple-touch-icon`, `manifest`) go in
  `nuxt.config.ts`'s `app.head.link`.

## Testing

- Vitest + `@vue/test-utils`, `happy-dom`/`jsdom` environment, tests under
  `tests/`

## Notes for Claude

- If state already has a home (a Pinia store or a `useX()` composable),
  extend that instead of adding a new one that holds similar/overlapping
  data — don't introduce a different state mechanism (e.g. a standalone
  reactive singleton, an event bus) when Pinia/composables already cover the
  case
- When editing an existing `.vue` file, put new code in the section it
  belongs to (e.g. a new `ref` goes under STATE, a new `computed` under
  COMPUTED) — don't tack it onto the bottom of `<script setup>` regardless of
  what section it belongs in
- Check `nuxt.config.ts` for route-specific SSR overrides (`routeRules`)
  before assuming a page is server-rendered
