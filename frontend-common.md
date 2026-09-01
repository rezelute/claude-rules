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

- **Never hardcode specific values** — padding, margin, color, font size,
  radius, shadow, etc. — directly in a component or stylesheet. Every such
  value must come from the token system (a Tailwind token/utility backed by
  `@theme`, or a CSS variable). If a value you need doesn't have a token yet,
  add one rather than writing the raw value inline.
- Always set up a token system, even on a small project — raw
  colors/sizes/fonts hardcoded inline make a later rebrand, dark-mode pass, or
  spacing tweak a find-and-replace across every component instead of a
  one-line edit to a token.
- Split token/style files by type rather than dumping everything in one
  stylesheet — e.g. separate files for colors, typography/fonts, headings,
  spacing, component overrides — all pulled together from one entry
  stylesheet (`main.css`).
- **Tailwind v4 (optional — only if using Tailwind):** tokens are CSS-first,
  defined in `@theme` blocks (`--color-*`, `--font-*`, `--text-*`,
  `--spacing-*`, etc.) — there's no `tailwind.config.js`. `@theme` isn't
  limited to a single file: split it the same way as any other tokens
  (`theme/colors.css`, `theme/typography.css`, ..., each with its own
  `@theme { ... }` block) and `@import` every partial into the one entry
  stylesheet — Tailwind picks up every `@theme` block in the build regardless
  of which imported file declares it.
- Custom classes that aren't plain Tailwind utility usage (heading variants,
  page margins, component style overrides) belong in `@layer components` /
  `@layer utilities`, not bare unscoped CSS — that keeps them playing
  correctly with Tailwind's cascade/specificity instead of fighting utility
  classes at random.
- Only the entry stylesheet imports Tailwind and the token/style partials —
  components should never `@import` a CSS file directly.

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
