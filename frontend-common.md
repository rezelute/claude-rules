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

### The core rule

Never hardcode a value — color, spacing, font size, radius, shadow, etc. — directly in a component. Every value comes from a token. If a token doesn't exist yet for what you need, add one instead of writing the value inline.

Why: without tokens, a rebrand or dark-mode pass means hunting down every hardcoded value across every component. With tokens, it's a one-line edit.

### The two layers

There are two layers, and they should never be confused with each other:

1. **Tokens (plain CSS custom properties)** — the source of truth. Just `:root { --color-primary: #2563eb; }`. No Tailwind syntax at all. Works whether or not Tailwind is even installed.
2. **Tailwind (optional)** — if used, it only _reads_ the tokens, it never defines new values of its own. Tailwind should always be a thin layer sitting on top of the tokens, not a second source of truth.

### File layout

Split files by category from the start, even on a small project — there's no downside to doing this early, and it avoids a painful reorganization later.

```
assets/css/
├── tokens/
│   ├── colors.css        ← plain CSS vars — vanilla, no Tailwind
│   ├── typography.css
│   └── spacing.css
├── theme/                 ← Tailwind only: maps tokens into Tailwind's namespace
│   ├── colors.css
│   ├── typography.css
│   └── spacing.css
├── utilities/              ← Tailwind only: custom classes built on the tokens
│   ├── surfaces.css
│   ├── text.css
│   ├── borders.css
│   ├── interactive.css
│   └── layout.css
└── main.css                 ← imports everything, in order
```

- **`tokens/`** is the only folder that's pure vanilla CSS. Delete Tailwind from the project and these files don't need to change.
- **`theme/`** and **`utilities/`** only exist if Tailwind is in the project. They don't define new values — they just expose the tokens as Tailwind utilities.
- **`main.css`** is the only file allowed to `@import` anything. Components never `@import` a CSS file directly.
- Import order matters: tokens → theme → utilities. Each layer depends on the one before it.

### Tailwind v4 specifics (skip this section if not using Tailwind)

Tailwind v4 defines its theme in CSS directly, using `@theme` blocks — there's no `tailwind.config.js` needed for this (the config file still exists, but only for plugins/JS config now, not for tokens).

Each `theme/*.css` file maps its matching token file into Tailwind's namespace, e.g.:

```css
/* theme/colors.css */
@theme {
  --color-primary: var(--color-primary);
}
```

Never write a custom class as bare, unwrapped CSS (`.surface-card { ... }` with no `@layer`/`@utility` wrapper) — it sits outside Tailwind's cascade system entirely, so whether it wins or loses against a real Tailwind utility becomes unpredictable (dependent on source order rather than intent). Every custom class goes into one of two buckets:

**`@utility` — layout/spacing primitives, single-purpose classes, anything that should support variants.**

```css
/* utilities/layout.css */
@utility vstack-lg {
  @apply flex flex-col gap-stack-lg;
}
```

This registers the class as a first-class Tailwind utility: it gets automatic variant support (`hover:vstack-lg`, `dark:vstack-lg`, `md:vstack-lg` just work, no extra setup) and utility-level override priority.

**`@layer components` — multi-property styled components meant to be overridable by utilities.**

```css
/* components/buttons.css */
@layer components {
  .btn {
    @apply px-4 py-2 rounded-md font-medium bg-primary text-white;
  }
}
```

This gives the class _lower_ priority than utilities, on purpose — so a one-off `class="btn bg-red-500"` cleanly overrides just the background without a specificity fight. Things like `.btn`, `.badge`, `.card` (bundling several properties together, expected to be tweaked per-instance) belong here, not in `@utility`.

Rule of thumb: if it's a single-property pattern (spacing, a surface color, a text color) → `@utility`. If it's a bundled, multi-property component meant to be overridden per-instance → `@layer components`.

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

- Tests mirror the source path being tested.
