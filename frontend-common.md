# Frontend Common

Frontend conventions that hold regardless of which component framework a
project uses. Framework-specific rules (Vue, and later React) live in their
own files and build on top of this one.

## Code Style

- No semicolons, double quotes, trailing commas — match the project's actual
  Prettier config for exact print width and other specifics.
- Imports auto-sorted (side effects → packages → aliases → relative).
- Don't hand-edit anything under `**/generated/**` — it's regenerated, not
  written.

## CSS & Styling

Three-layer token system: primitives (`:root`) → semantic (`@theme`) → utilities (`@utility`). Primitives stay `:root` since components never touch them directly. Semantic uses `@theme` so Tailwind auto-generates utilities (`bg-brand`) from it — if Tailwind is ever dropped, swap `@theme` back to `:root` and everything downstream still works.

```
tokens/
├── primitives/
│   ├── color.css
│   ├── spacing.css
│   ├── radius.css
│   └── typography.css
├── semantic/
│   ├── color.css
│   ├── spacing.css
│   └── typography.css
├── utilities/
│   ├── stack.css
│   ├── inset.css
│   └── typography.css
└── index.css
```

**`primitives/spacing.css`**

```css
:root {
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
}
```

**`semantic/spacing.css`**

```css
@theme {
  /* Gap between stacked children — consumed by vstack-* utilities. */
  --spacing-stack-md: var(--spacing-md);
  --spacing-stack-lg: var(--spacing-lg);
}
```

**`utilities/stack.css`**

```css
@utility vstack-lg {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-stack-lg);
}
```

**Rules**

- Never access primitives directly — always through a semantic var.
- Rarely use Tailwind's default scale for color/spacing/font-size/radius/shadow (`bg-blue-500`, `p-4`) — use tokens, so values stay themeable and tracked in one system.
- No arbitrary values (`bg-[#3b82f6]`, `p-[18px]`) — add a token instead.
- Reusable multi-property patterns → `@utility` (free variant support: `hover:`, `dark:`, `md:`). One-off, component-specific patterns → vanilla CSS in that component's file.
- Never `@apply` — vanilla CSS reading `var(--...)` is more readable than a long `@apply` line.
- Theme overrides only touch `semantic/*.css`.

## Data Fetching **(optional — only if the project talks to a backend)**

- All API calls live in a dedicated `api`/`services` folder or layer — never
  call a fetch/query client directly from a component or page.
- Prefer codegen (REST or GraphQL) from the backend's schema/OpenAPI spec
  over hand-written request code where one is available; regenerate after any
  backend schema/API change, and never hand-edit generated output.
- Handling failed requests: surface a user-facing error state (toast/inline
  message), never swallow a caught error silently. Match whatever error
  shape the backend returns (see the backend rules' API error response
  section) so the frontend can branch on `error.code` rather than parsing
  message strings.

## Static JSON Data

- When JSON files are used as static data in the frontend — either because
  there's no backend yet, or because part of the data is static while the
  rest will come from a backend — always structure/build that JSON the way
  it would look coming from a real API response, and split it accordingly
  (e.g. separate files/keys per resource, same shape a backend endpoint
  would return) rather than as one ad hoc blob.
- Why: this keeps the eventual swap to a real API a data-source change only
  — components and consuming code don't need to be rewritten once the
  backend part lands.

## Error Handling

- Route the majority of `catch` blocks through one shared error-handling
  utility instead of each call site reimplementing logging/reporting/toast
  logic. (In Vue this is a composable; in React it'd be a hook — see the
  framework-specific rules for the concrete shape.)
- What it should do: log the error, report it to the project's error
  monitoring tool (tagging/attaching extra context when the error is a
  recognized app-level error type, so issues are easier to triage), and —
  when told to — show a user-facing toast, falling back to a generic
  "something went wrong" message when the error doesn't carry a specific one.
- Take flags for whether to show a toast and whether to report to error
  monitoring, so call sites can opt out selectively (e.g. an expected/handled
  failure that shouldn't page anyone, or a background retry that shouldn't
  interrupt the user).
- New projects without an existing error-handling utility should add an
  equivalent early and route catches through it from the start, rather than
  bolting it on after error handling is already scattered ad hoc through the
  app.

## SEO **(optional — only if the project is public-facing)**

- Prefer per-page metadata (title/description/OG/Twitter tags) over one
  global default, so metadata reflects each page's actual content.
- A global fallback is fine for a single-purpose site where every page
  shares the same description, or as a baseline before per-page metadata is
  added — but any project with distinct per-page content should prefer
  per-page metadata over relying on the global block.
- Favicon/manifest links are one-time boilerplate, not something to actively
  maintain — set once per new project.

## Accessibility

- Use semantic HTML first (`button`, `nav`, `label`, `dialog`) — reach for
  ARIA attributes only to fill a genuine gap, not as a default
- Every interactive element must be reachable and operable by keyboard alone
  (tab order follows visual order; no keyboard traps in modals/dropdowns)
- Manage focus explicitly on route change and when opening/closing modals or
  drawers (move focus into the modal on open, return it to the trigger on
  close)
- All images need meaningful `alt` text; purely decorative images use
  `alt=""`
- Form inputs always have an associated `<label>` (via `for`/`id` or
  wrapping) — placeholder text is not a substitute for a label
- Verify keyboard behaviour and ARIA roles whenever customising a component
  library's markup/slots, even when the library ships reasonable a11y
  defaults
- Maintain WCAG AA contrast (4.5:1 body text, 3:1 large text/UI components) —
  check custom color combinations, don't assume the design is compliant

## Testing

- Test files are co-located with the source file they test (e.g. `UserCard.vue` + `UserCard.spec.ts` in the same folder) — not grouped into a separate `tests/` tree.
- Access elements/components under test through a shared `data-test` lookup
  helper (e.g. `findElementById`/`findComponentById`) instead of writing
  `[data-test="..."]` selectors inline in every test — one place to change if
  the attribute or lookup logic changes.
- Mock data lives in one shared folder (`tests/mocks/`) reused across tests —
  never inline ad hoc mock objects in individual test files. Type each mock
  against its
  corresponding generated/existing type (API response type, model, etc.) so
  it breaks at compile time if the real shape changes, rather than drifting
  silently out of sync.
