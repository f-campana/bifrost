---
name: react/component
version: 1.0.0
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
library. This skill governs the component architecture, API design, state
model, and composition patterns. It is the React-specific counterpart to
`frontend/island` — where `frontend/island` handles the Astro integration
and Press motion rules, this skill handles React correctness, component
API design, and the behavioral primitive model.

Always pair this skill with `typescript/component` for type validation
and `accessibility/component` for behavioral correctness.

## What it does

1. Reads the Design Contract and component specification.
2. Reads the relevant React source material in `sanctum/40 Sources/React/`.
3. Produces one React functional component as a `.tsx` file following the
   rules below.
4. Writes a run report to
   `sanctum/00 Inbox/react-component-[name]-[date].md`.
5. Reports: "Component: [name] · Props: [interface] · State: [model]
   · Composition pattern: [pattern used] · Lines: [N]."

## What it does NOT do

- Does not produce class components — functional components only.
- Does not produce Astro files — use `frontend/component` or
  `frontend/island` for Astro integration.
- Does not implement accessibility behavior from scratch — behavioral
  primitives (keyboard, focus, ARIA) come from React Aria. This skill
  owns the visual layer on top.
- Does not manage global or cross-component state — each component is
  self-contained. Application state is the consuming app's concern.
- Does not choose a state management library — `useState` and `useReducer`
  are the tools at this layer.

## Inputs

**Required:**
- `design_contract` — path to the Design Contract.
- `component_spec` — description of the component: what it is, what
  props it receives, what state it has, what Press rules apply, which
  React Aria primitive it wraps (if interactive).

**Optional:**
- `output_path` — override the default output directory.

## Output

One `.tsx` file at `runestone/src/components/[ComponentName].tsx`.

## Rules

### Component model

- Functional components only. No class components.
- Define the component at the module level. Never define a component
  inside another component — this creates a new component identity on
  every render, breaking reconciliation and state.
- Export the component as a named export and the props interface as a
  named export. Both must be exported — consumers need to type wrappers.
- Component names use PascalCase. Prop interface names use
  `[ComponentName]Props`.

### Props

- Every prop is explicitly typed. No implicit `any`.
- Use `React.ReactNode` for children unless the component requires
  a single element (document why if so).
- Prefer `variant: 'primary' | 'ghost'` over `isPrimary: boolean`.
  Mutually exclusive states are discriminated unions, not boolean flags.
- Spread native element props where the component wraps a native element:
  `interface ButtonProps extends React.ComponentProps<'button'>`.
  This allows consumers to pass any valid HTML attribute.
- Support `asChild` for polymorphic usage using Radix's `Slot` when
  the component needs to render as a different element.
- Export the props interface. Hidden interfaces are a component library
  anti-pattern.

### State

- Use `useState` with an explicit type when the initial value does not
  infer the full state shape.
- Model multi-step interaction states as discriminated unions:
  `'idle' | 'pressed' | 'loading'`, not separate booleans.
- State that is derived from props is not state — compute it during render.
- Lift state only when two components genuinely need to share it. Do not
  lift preemptively.
- `useReducer` over multiple `useState` calls when state transitions
  follow a defined set of actions.

### Effects and side effects

- Effects are for synchronisation with external systems — DOM APIs,
  timers, subscriptions. They are not for deriving state from props.
- Every effect must have a complete dependency array. Missing dependencies
  are bugs.
- Clean up subscriptions, timers, and event listeners in the effect's
  return function.
- Do not fetch data in effects inside components. Data fetching belongs
  at the page or loader level.

### Composition patterns

- Prefer composition over configuration. A component that accepts
  children and renders them is more flexible than one with twenty props
  describing what to render inside it.
- Use compound components for UI that has a rigid structural relationship:
  `<Select>` + `<Select.Trigger>` + `<Select.Content>` rather than
  `<Select trigger={...} content={...} />`.
- Use render props or `children` as a function only when the parent
  needs to share internal state with children that it does not own.
- The `asChild` pattern (Radix `Slot`) is the correct tool for
  polymorphism. Avoid generic `as` props unless `asChild` cannot
  express the requirement.

### Behavioral primitives

- Interactive components (buttons, dialogs, listboxes, tooltips,
  select menus, comboboxes) must be built on React Aria primitives,
  not from scratch.
- React Aria handles: keyboard interaction, focus management, ARIA
  roles/states/properties, touch and pointer event normalization,
  screen reader announcements.
- The Press component owns: visual presentation, token application,
  Press inviolable rules, CVA variants.
- The boundary between React Aria and Press is the `data-*` attribute
  surface: React Aria exposes `data-hovered`, `data-pressed`,
  `data-focused`, `data-disabled`. Press CSS targets these attributes
  for state-driven styling. No JavaScript needed to style hover or
  focus states.

### Performance

- Do not wrap every component in `React.memo`. Memoize only when a
  profiler identifies a specific re-render problem.
- If a callback prop is passed to a memoized child, stabilize it with
  `useCallback`. Otherwise `useCallback` adds overhead without benefit.
- Expensive computations in render belong in `useMemo`. Cheap
  computations do not.
- Structural fixes before memoization: if a component re-renders because
  its parent re-renders, consider whether composition (lifting the
  stable part out of the re-rendering parent) solves it without memo.

### Purity

- Components must be pure with respect to rendering: given the same
  props and state, they must return the same JSX.
- Do not mutate props, state, context, or external variables during render.
- Side effects belong in event handlers or effects, not in the render body.

## Failure modes

**Component requires data fetching:** Stop. Report that data fetching
belongs at the page or loader level. Describe what data the component
needs and where the parent should source it.

**Spec implies a class component:** Stop. All components are functional.
Describe the equivalent functional pattern.

**Spec requires reinventing an accessible primitive:** Stop. Report which
React Aria primitive covers this behavior and dispatch with the React Aria
primitive as the base instead.

**Scope creep:** One component per dispatch. If the spec implies multiple
related components, stop and split the spec.

---

## Current implementation

**React version:** React 18
**TypeScript:** strict mode required — see `typescript/component` skill
**Styling:** Tailwind v4 utility classes via Press adapter, CVA for variants
**Accessible primitives:** React Aria (`@react-aria/*` or `react-aria`)
**Package manager:** pnpm

**If the stack changes:** update this section only. The contract above
remains stable.

---

## Sources

Rules derived from:
- `sanctum/40 Sources/React/Concepts/React - Component Model, Props, and State.md`
- `sanctum/40 Sources/React/Concepts/React - Composition, Lifting State, and Controlled vs Uncontrolled.md`
- `sanctum/40 Sources/React/Concepts/React - Hooks, Effects, and Closures.md`
- `sanctum/40 Sources/React/Concepts/React - Memoization and Performance Tradeoffs.md`
- `sanctum/40 Sources/React/Lessons/React - Lesson 04 - Advanced Patterns at Scale.md`
- `sanctum/40 Sources/React/Lessons/React - Lesson 05 - Performance Beyond Memo.md`
- Vercel composition patterns: compound components, injectable state,
  boolean prop soup anti-pattern
- React team docs: Thinking in React, Components and Hooks must be pure,
  Using TypeScript
