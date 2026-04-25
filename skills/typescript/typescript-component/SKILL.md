---
name: typescript/component
version: 1.0.0
type: validator
domain: typescript
tier: action
triggers:
  - /typescript-component
  - "check TypeScript in this component"
  - "apply TypeScript rules to"
reads_from: any TypeScript or TSX file + tsconfig.json
writes_to: sanctum/00 Inbox/typescript-component-[name]-[date].md
model_preference: claude-sonnet
---

# typescript/component

## When to use

Use when producing or reviewing TypeScript code for the Press component
library — prop interfaces, utility types, hook signatures, or any
TSX file. This skill encodes the TypeScript rules required for a
production component library: strict compiler settings, safe prop
interfaces, runtime boundary discipline, and inference-friendly API
design. Load it alongside `frontend/component` or `frontend/island`
for every component dispatch.

## What it does

1. Reads the TypeScript or TSX file and the project tsconfig.json.
2. Evaluates the code against the rules below.
3. Identifies violations — each one named, located, and described.
4. Writes a report to `sanctum/00 Inbox/typescript-component-[name]-[date].md`.
5. Reports: "File: [name] · Violations: [N] · Rules applied: [list]."

## What it does NOT do

- Does not rewrite the file — it reports violations, it does not fix them.
  Use `frontend/component` or `frontend/island` to produce the corrected file.
- Does not enforce application-level TypeScript patterns — this skill is
  scoped to component library code only.
- Does not validate runtime behaviour — only static type correctness.

## Inputs

**Required:**
- `file_path` — path to the TypeScript or TSX file to evaluate.

**Optional:**
- `tsconfig_path` — path to the tsconfig.json. Defaults to the nearest
  tsconfig.json in the project tree.

## Output

One markdown report at
`sanctum/00 Inbox/typescript-component-[name]-[date].md`.

Each violation is reported as:
```
Rule: [rule name]
Location: [file:line]
Found: [what the code does]
Required: [what it should do instead]
```

## Rules

### Compiler settings (check tsconfig.json)

All of these must be enabled. If any are absent, report them as violations
before evaluating the code itself.

- `strict: true` — enables `strictNullChecks`, `noImplicitAny`,
  `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`,
  `noImplicitThis`, and `alwaysStrict`.
- `exactOptionalPropertyTypes: true` — distinguishes `prop?: string` (may
  be absent) from `prop: string | undefined` (must be present but may be
  undefined). Required for honest prop interfaces.
- `noUncheckedIndexedAccess: true` — array and object index access returns
  `T | undefined`, not `T`. Required for safe collection handling in components.

### Prop interfaces

- Every component must define an explicit typed interface for its props.
  `React.ComponentProps<"div">` is acceptable as a spread base, but the
  component's own props must be named and typed.
- `any` is forbidden in prop interfaces. Use `unknown` for genuinely
  untyped values and narrow at the boundary.
- Optional props must be declared with `?`, not typed as `T | undefined`
  with a required key. With `exactOptionalPropertyTypes`, these are distinct.
- Boolean props that toggle a variant must be typed as a discriminated
  union or a CVA variant, not as `boolean`. Prefer:
  `variant: 'primary' | 'ghost'` over `isPrimary: boolean; isGhost: boolean`.
- Polymorphic components that accept an `as` or `asChild` prop must type
  that prop correctly. `asChild` uses Radix's `Slot` pattern;
  `as` requires a generic constrained to `React.ElementType`.

### Children and composition

- Use `React.ReactNode` for children props that accept any renderable
  content. Do not use `React.ReactElement` unless the component actually
  requires a single element (rare, and usually a design mistake).
- Do not use TypeScript to constrain children to a specific JSX element
  type at compile time — TypeScript cannot enforce JSX children types
  reliably. If a component requires specific children, enforce it at
  runtime with a guard, not at the type level.

### State and hooks

- `useState` must be typed explicitly when the initial value does not
  infer the full type. `useState<'idle' | 'pressed' | 'disabled'>('idle')`
  is correct. `useState(null)` when the state can be an object is not.
- `useRef` must be typed with the element type. `useRef<HTMLButtonElement>(null)`,
  not `useRef(null)`.
- `useCallback` and `useMemo` dependency arrays must be complete. Missing
  dependencies are TypeScript-visible when the callback captures typed values.
- Custom hooks must declare return types explicitly. Inferred return types
  for hooks are acceptable only when the inference is unambiguous and stable.

### Runtime boundaries

- Values crossing a trust boundary (API response, localStorage, user input,
  props from an untyped parent) must be typed as `unknown` at the entry
  point and narrowed before use.
- Do not cast with `as T` to bypass narrowing. `as T` is permitted only
  when the runtime guarantee is explicit and documented in a comment.
- Branded types are encouraged for semantic tokens and IDs that must not
  be mixed: `type RuneSlug = string & { readonly __brand: 'RuneSlug' }`.

### API design

- Prefer inference-friendly signatures. If TypeScript can infer the return
  type from the implementation, do not annotate it redundantly. If the
  return type is complex or non-obvious, annotate it explicitly.
- Avoid overloads when a union or generic achieves the same expressiveness.
  Overloads are for cases where the return type genuinely depends on the
  input type in ways a union cannot express.
- Do not use `object` or `{}` as prop types. Name the shape explicitly.
- Export prop interfaces. Consumers of the library need to type their own
  wrappers. Hidden interfaces are a component library anti-pattern.

## Failure modes

**tsconfig.json not found:** Report the missing file. Do not evaluate
the component — compiler settings are a prerequisite.

**Required compiler flag absent:** List all missing flags before
evaluating code. Do not skip to code violations.

**`any` in prop interface:** Flag every occurrence. `any` in a prop
interface propagates unsafety to every consumer.

**Scope creep:** If the file contains application logic, server actions,
or infrastructure code rather than component library code — report the
scope boundary and evaluate only the component-facing surface.

---

## Sources

Rules derived from:
- `sanctum/40 Sources/TypeScript/Concepts/TypeScript - Compiler and API Design.md`
- `sanctum/40 Sources/TypeScript/Concepts/TypeScript - Runtime Boundaries and Safety.md`
- `sanctum/40 Sources/TypeScript/Lessons/TypeScript - Lesson 04 - Staff and Principal.md`
- `sanctum/40 Sources/TypeScript/Concepts/TypeScript - Type System Fundamentals.md`
- TypeScript compiler docs: `strict`, `exactOptionalPropertyTypes`,
  `noUncheckedIndexedAccess`
- React team docs: `Using TypeScript`, `Common DOM components`
