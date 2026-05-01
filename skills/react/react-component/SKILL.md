---
name: react/component
version: 1.1.0
type: producer
domain: react
tier: action
triggers:
  - /react-component
  - "build a React component"
  - "produce a production React component"
reads_from: Design Contract (DESIGN_[PROJECT].md) + component specification
writes_to: runestone/src/components/[ComponentName].tsx
model_preference: claude-sonnet
---

# react/component

## When to use

Use when producing a production React component for the Press component
library. This skill governs component architecture, API design, state
model, and composition patterns. Always pair with `typescript/component`
for type validation and `accessibility/component` for behavioral
correctness.

Before using this skill, read the component architecture ADR at:
`sanctum/10 Knowledge/Concepts/press-component-architecture-adr.md`

## What it does

1. Reads the Design Contract and component specification.
2. Reads the component architecture ADR.
3. Produces one React functional component as a `.tsx` file following
   the rules and template below.
4. Writes a run report to
   `sanctum/00 Inbox/react-component-[name]-[date].md`.
5. Reports: "Component: [name] · Primitive: [Base UI primitive used]
   · Props: [interface] · Lines: [N]."

## What it does NOT do

- Does not produce class components — functional components only.
- Does not produce Astro files — use `frontend/component` or
  `frontend/island` for Astro integration.
- Does not implement accessibility behavior from scratch — Base UI
  handles keyboard, focus, and ARIA. This skill owns the visual layer.
- Does not use asChild or Radix Slot — see ADR Decision 1.
- Does not use react-aria hooks (useButton, useHover, useFocusRing,
  mergeProps) by default — see ADR Decision 2.
- Does not manage global or cross-component state.

## Inputs

**Required:**
- `design_contract` — path to the Design Contract.
- `component_spec` — what the component is, what props it receives,
  which Base UI primitive it wraps (if interactive).

**Optional:**
- `output_path` — override the default output directory.

## Output

One `.tsx` file at `runestone/src/components/[ComponentName].tsx`.

## Component template

Every Press component follows this structure. Do not deviate:

```typescript
// Hydration boundary note:
// In Astro (current system): add client:* on the .astro page, not here.
// In RSC frameworks (Next.js App Router): add 'use client' at this line.

import { [Primitive] as [Primitive]Primitive } from '@base-ui/react/[primitive]'
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const press[Component]Variants = cva(
  [
    // Press inviolable rules — never remove these three
    'border-2 border-solid rounded-none shadow-none',
    // Base layout
    '...',
    // Focus — keyboard only via CSS, Press focus-ring token
    'focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2',
    'focus-visible:outline-[var(--pr-focus-ring)]',
  ],
  {
    variants: {
      variant: {
        primary: [...],
        ghost: [...],
      },
    },
    defaultVariants: { variant: 'primary' },
  }
)

export interface Press[Component]Props
  extends [Primitive]Primitive.Props,
    VariantProps<typeof press[Component]Variants> {}

export function Press[Component]({
  className,
  variant,
  ...props
}: Press[Component]Props) {
  return (
    <[Primitive]Primitive
      className={cn(press[Component]Variants({ variant, className }))}
      {...props}
    />
  )
}
```

## Rules

### React 19 idioms

- No `forwardRef` — ref is a regular prop in React 19.
- Do not add `useMemo` or `useCallback` manually. The React Compiler
  handles memoization. Manual memoization is technical debt.
- `VariantProps<typeof variantFn>` drives prop types — do not hand-mirror
  variant unions in the interface.

**Hydration boundary:** In Astro, hydration is controlled at the page
level via `client:*` on the `.astro` file — not via `'use client'`
inside component files. In RSC frameworks (Next.js App Router), add
`'use client'` at the top of interactive component files. The Press
component library is agnostic about the hydration boundary; the
consuming framework determines where it sits.

### Component model

- Functional components only. No class components.
- Define the component at the module level. Never define a component
  inside another component.
- Export both the component (named export) and the props interface
  (named export). Consumers need to type wrappers.
- Component names use PascalCase. Prop interfaces use `[Name]Props`.

### Props

- Every prop is explicitly typed. No implicit `any`.
- Use `React.ReactNode` for children unless the component requires
  a single element — document why if so.
- Prefer `variant: 'primary' | 'ghost'` over `isPrimary: boolean`.
  Mutually exclusive states are discriminated unions, not booleans.
- Extend the Base UI primitive's props type:
  `interface PressButtonProps extends ButtonPrimitive.Props, VariantProps<...>`
  This gives consumers the full primitive API including
  `focusableWhenDisabled`, `disabled`, `onPress`, etc.
- Do not add asChild. See ADR D1.
- Export the props interface.

### Polymorphism

Polymorphism is handled by separate components, not asChild.
`PressButton` renders a `<button>`. `PressButtonLink` renders an `<a>`.
Both share the same CVA variant definitions. Both extend their
respective Base UI primitive's props.

When a spec asks for a "button that renders as a link," produce two
components sharing variants. Never use Radix Slot or asChild.

### Base UI primitives

Interactive components wrap a Base UI primitive from
`@base-ui/react`:
- Button → `/button`
- Dialog → `/dialog`
- Popover → `/popover`
- Select → `/select`
- Checkbox → `/checkbox`
- Switch → `/switch`
- Tooltip → `/tooltip`
- Menu → `/menu`

Base UI handles: keyboard interaction, focus management, ARIA
roles/states/properties, press normalization, `focusableWhenDisabled`.

The default primitive layer is Base UI. If no Base UI primitive exists
for the use case, report the gap rather than switching to another
library — this requires an explicit ADR update.

### Interaction state styling

Interaction states are styled via CSS pseudo-classes and ARIA attribute
selectors — not JavaScript-driven data-* attributes:

```
hover:bg-[var(--pr-orange)]          ← pointer hover
focus-visible:outline-[var(...)]     ← keyboard focus only
disabled:opacity-50                  ← disabled via native attribute
aria-invalid:border-[var(...)]       ← validation state
aria-expanded:bg-[var(...)]          ← open/closed composites
active:translate-y-px                ← press feedback
```

Do not add `useHover`, `useFocusRing`, or manual `data-hovered`
spreading for simple components. CSS handles it.

Base UI exposes `data-*` attributes automatically for complex composites
(Dialog, Select) where CSS pseudo-classes are insufficient. Simple
primitives (Button, Badge, Tag) do not need JS state machines.

### Disabled state

Use Base UI's `focusableWhenDisabled` prop when the component must
remain in the tab order while disabled. Do not manually strip the
native `disabled` attribute. Do not manually add `aria-disabled`.
Base UI handles all of this correctly.

Default: `focusableWhenDisabled` is false. Set it explicitly when a
tooltip explains why the element is disabled.

If the component adds custom DOM `click` handlers (beyond Base UI's
`onPress`), those handlers must check and respect the `disabled` state.
Base UI suppresses press events — it does not suppress native DOM click.

### CVA and class composition

- CVA base string must always include:
  `'border-2 border-solid rounded-none shadow-none'`
  These are Press inviolable rules. Present in every component's base.
- Use `cn()` (clsx + tailwind-merge) to merge CVA output with
  consumer className.
- Focus visible classes must be in the base string of every interactive
  component.

### State and hooks

- `useState` only for UI state: open/closed, loading, selected index.
- Model multi-step states as discriminated unions, not booleans.
- `useReducer` when state transitions follow a defined action set.
- Do not add hooks for interaction state (hover, press, focus) —
  CSS handles those.

### Composition

- Prefer composition over configuration.
- Compound components for rigid structural relationships.
- Do not use render props or children-as-function unless the parent
  genuinely needs to share internal state with children it does not own.

### Purity

- Pure render: same props + state → same JSX.
- Do not mutate props, state, context, or external variables in render.
- Side effects in handlers or effects, not in render.

## Failure modes

**Spec asks for asChild:** Stop. Propose the two-component approach
from ADR D1 (PressButton + PressButtonLink).

**Spec says to use react-aria hooks:** Stop. The default primitive
layer is Base UI. Report which Base UI primitive covers the use case.
If none exists, report the gap — do not switch libraries unilaterally.

**No Base UI primitive for this use case:** Stop. Report the gap.
This requires an ADR update before proceeding.

**Spec requires data fetching:** Stop. Data belongs at the page level.

**Scope creep:** One component per dispatch.

---

## Current implementation

**React version:** React 19
**TypeScript:** strict — see `typescript/component` skill
**Behavioral primitives:** Base UI (`@base-ui/react`)
**Variant system:** CVA (`class-variance-authority`)
**Class merging:** `cn()` = clsx + tailwind-merge
**Styling:** Tailwind v4 + Press adapter + CSS pseudo-classes
**Hydration (Astro):** `client:*` on the `.astro` page
**Package manager:** pnpm

**Architecture reference:**
`sanctum/10 Knowledge/Concepts/press-component-architecture-adr.md`

---

## Sources

- `sanctum/10 Knowledge/Concepts/press-component-architecture-adr.md`
- `sanctum/40 Sources/React/Concepts/React - Component Model, Props, and State.md`
- `sanctum/40 Sources/React/Concepts/React - Composition, Lifting State, and Controlled vs Uncontrolled.md`
- `sanctum/40 Sources/React/Concepts/React - Hooks, Effects, and Closures.md`
- `sanctum/40 Sources/React/Concepts/React - Memoization and Performance Tradeoffs.md`
- `sanctum/40 Sources/React/Lessons/React - Lesson 04 - Advanced Patterns at Scale.md`
- `sanctum/40 Sources/React/Lessons/React - Lesson 05 - Performance Beyond Memo.md`
- Base UI source: `@base-ui/react` Button component
- shadcn/ui Base UI button source (2025)
- Adobe React Spectrum Button source (2024)
