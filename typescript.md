# TypeScript Standards

Applies to any TypeScript project — frontend and backend alike.

- **No `any`** — use `unknown` with type guards/narrowing, or a proper
  generic/type. Treat it as a rare, commented escape hatch (e.g. an awkward
  third-party type gap), not a default.
- **No non-null assertions (`!`)** — handle the null/undefined case
  explicitly instead of asserting it away.
- **Avoid `as` type assertions** — prefer real typing, generics, or `zod`'s
  `.parse()`/`z.infer`. When unavoidable, leave a comment explaining why.
- **`@ts-expect-error` (with a comment) over `@ts-ignore`** when a type error
  genuinely must be suppressed.
- **Full strict mode in tsconfig** (`strict: true`, plus
  `noUncheckedIndexedAccess`) — don't loosen it project-wide to work around a
  one-off error.
- **Derive types from Zod schemas** (`z.infer<typeof schema>`) instead of
  hand-writing a duplicate TS type/interface for the same shape.
