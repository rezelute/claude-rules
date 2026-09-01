# Common Standards

Cross-project conventions — language-agnostic, applies to frontend and backend
alike. See `typescript.md` for TypeScript-specific rules on top of this.

## Dependencies

- Always use the latest **stable** version of every dependency, including the
  framework/runtime itself — don't pin to or assume a specific major version.
  Check for the current stable release when starting a new project or
  upgrading. Skip alpha/beta/rc/next tags unless the project explicitly needs
  one.

## Code Structure

- Prefer clear and simple code over clever or compact code — an extra few
  lines that are easy to follow beats a one-liner that needs unpacking.
- Comment the _why_, not the _what_ — the code already shows what it does; a
  comment earns its place by explaining a non-obvious reason, trade-off, or
  gotcha.
- Keep functions/methods focused on one responsibility — if something is
  doing several distinct things, extract them into named helpers rather than
  growing one long function.
- Avoid deep nesting (more than 2-3 levels of `if`/loops) — prefer early
  returns and guard clauses over nested conditionals.
- Avoid magic numbers/strings — give them a named constant when the value
  isn't self-explanatory.
- No hard rule on file length, but if a file/function is hard to hold in your
  head at once, that's the signal to split it.
