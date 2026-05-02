---
name: frontend/component
version: 1.0.0
type: producer
domain: frontend
tier: action
triggers:
  - /frontend-component
  - "build this as a frontend component"
  - "create a component for"
reads_from: Design Contract (DESIGN_[PROJECT].md) + component specification
writes_to: runestone/src/components/[ComponentName].astro
model_preference: claude-sonnet
---

# frontend/component

## When to use

Use when a new static, non-interactive frontend component is needed and a
Design Contract already exists. The component specification must come from
the Design Contract — this skill does not invent design decisions. The
typical trigger is building one section of the Runestone (a Rune card, the
nav, a radar visualisation, a token swatch row) as a reusable typed
component rather than inline markup in a page file.

This skill produces static components with no client-side JavaScript. For
components that require interactivity — motion moment previews, component
galleries with live hover states, state machines — use `frontend/island`
instead.

## What it does

1. Reads the Design Contract in full before writing any code.
2. Reads the component specification — which section of the page this
   component represents, what data it receives, what design rules apply.
3. Produces one component file following the Current implementation
   section below.
4. The component:
   - Defines typed props
   - Uses only CSS custom properties from the Design Contract token set
   - Applies no border-radius — the global reset handles it
   - Uses scoped styles for component-specific rules
   - Does not import or inline global styles
   - Does not use box-shadow unless the Design Contract explicitly permits
   - Does not add motion unless the spec names a specific named Moment
     from the Design Contract
5. Writes a run report to
   `sanctum/00 Inbox/frontend-component-[name]-[date].md`.
6. Reports: "Component: [name] · Props: [list] · Lines: [N] · Press
   rules applied: [list each rule checked and whether it passed or
   was not applicable — a numeric count alone is insufficient]."

## What it does NOT do

- Does not invent design decisions not present in the Design Contract.
- Does not produce interactive components — use `frontend/island`.
- Does not produce page files — only component files.
- Does not use inline styles for static values — those belong in the
  scoped style block. Inline styles are only for dynamic values computed
  from props (e.g. a bar width derived from a score).
- Does not import fonts or global CSS — handled by the layout component.
- Does not produce a component that works only for one specific Rune —
  components must accept typed props and work for any conforming data.

**Single source rule:**
If a route needs a component that already exists,
reuse the component file.
Do not inline a second copy of the component markup
into the route file to avoid modifying the original.
If the route needs different copy or notes, add
typed props or create a thin wrapper component and
report the divergence explicitly.

## Inputs

**Required:**
- `design_contract` — path to the Design Contract.
  Example: `runestone/DESIGN_RUNESTONE.md`
- `component_spec` — natural language description: what the component
  is, what props it receives, which Press rules are critical.
  Example: "A RadarRow component that receives a label string and a
  score number (1–10). Renders a label, the score in orange mono, and
  a horizontal bar: --pr-border-light track, --pr-orange fill scaled
  to score/10. 2px border on the container."

**Optional:**
- `output_path` — override the default output directory.

## Output

One component file at `runestone/src/components/[ComponentName].astro`
where `ComponentName` matches the Rune section name or the exported
component name — no abbreviations, no generic names like `Section` or
`Block`. (See Current implementation for file format.)

## Press rules the output must satisfy

Before writing the component, verify each inviolable rule:

1. Orange (`--pr-orange`) used only as a structural element — never
   decorative
2. No border-radius declarations — global reset handles it
3. All explicit borders are `2px solid` — never 1px
4. Light-first surfaces — no dark backgrounds unless the component is
   explicitly the nav band
5. No second accent colour — only `--pr-orange` plus neutrals

If the component_spec would require violating any rule, stop and report
the conflict before writing any code.

## Failure modes

**Design Contract not found:** Stop. Report the exact path provided.
Do not proceed without the contract.

**Component spec violates a Press inviolable rule:** Stop. Report which
rule is violated and what in the spec causes it.

**Component requires client-side interactivity:** Stop. Report that this
component needs `frontend/island` instead. Name the specific behaviour
that requires JavaScript.

**Scope creep:** If the spec implies multiple components or a full page
section, stop. This skill produces one component per dispatch.

---

## Current implementation

**Framework:** Astro 6 (static output, no hydration)
**File format:** `.astro` with TypeScript frontmatter
**CSS approach:** Scoped `<style>` block using `--pr-*` custom properties
**Props:** Typed `interface Props` in the component frontmatter
**No hydration directive** — this component has no client-side JavaScript

**CSS layer ordering requirement:**
press.css must wrap its universal reset in @layer base.
If this is not done, Tailwind utility classes for padding
and margin will be silently overridden by the reset.
Verify this is in place before producing any component
that uses padding or margin utilities. The anti-pattern
is documented in press.md section 10.

Component structure:
```astro
---
export interface Props {
  // typed props derived from component_spec
}
const { ... } = Astro.props;
// derived values computed here (e.g. bar widths)
---

<div class="component-root">
  <!-- markup -->
</div>

<style>
  .component-root {
    /* component-specific styles using var(--pr-*) */
    /* no border-radius — reset handles it */
    /* no box-shadow unless Design Contract permits */
  }
</style>
```

**If the stack changes:** update this section only. The contract above
remains stable.
