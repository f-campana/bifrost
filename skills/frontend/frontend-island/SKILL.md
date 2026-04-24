---
name: frontend/island
version: 1.0.0
type: producer
domain: frontend
tier: action
triggers:
  - /frontend-island
  - "build an interactive component"
  - "this needs client-side interactivity"
  - "motion moment preview"
reads_from: Design Contract (DESIGN_[PROJECT].md) + component specification
writes_to: runestone/src/components/[ComponentName].tsx
model_preference: claude-sonnet
---

# frontend/island

## When to use

Use when a Runestone component requires client-side interactivity that
CSS alone cannot achieve: motion moment previews that respond to user
input, component galleries with live interactive states, state machines
that cycle between named states, or any component that needs event
handlers or reactive state.

This skill produces interactive components hydrated as islands in an
otherwise static Astro page. Use `frontend/component` for everything
that does not require JavaScript at runtime. When in doubt, start with
`frontend/component` and escalate to `frontend/island` only when
interactivity is confirmed necessary.

## What it does

1. Reads the Design Contract in full before writing any code.
2. Reads the component specification — which named Moment or interactive
   behaviour this component demonstrates, what states it has, what user
   gestures trigger transitions.
3. Produces one interactive component file (see Current implementation).
4. The component:
   - Defines typed props
   - Uses only CSS custom properties from the Design Contract token set
   - Implements motion using CSS keyframes or transitions — not a motion
     library — unless explicitly approved by the human
   - Applies Press easing: `cubic-bezier(0.0, 0.0, 0.2, 1)` or `linear`
   - Respects the 300ms duration ceiling — no single animation longer
   - Applies no border-radius — zero everywhere
   - Does not use box-shadow unless the Design Contract permits it
5. Writes a run report to
   `sanctum/00 Inbox/frontend-island-[name]-[date].md`.
6. Reports: "Island: [name] · States: [list] · Named moments: [list]
   · Lines: [N] · Press motion rules applied: [list each rule checked
   and whether it passed or was not applicable]."

## What it does NOT do

- Does not install motion libraries without explicit human approval —
  CSS handles all three named Press Moments without a library.
- Does not manage global state — each island is self-contained.
- Does not fetch data — all data comes through props from the parent
  page.
- Does not implement motion that violates Press rules: no spring
  physics, no `ease-in-out`, no animations longer than 300ms.
- Does not produce static components — use `frontend/component`.

## Inputs

**Required:**
- `design_contract` — path to the Design Contract.
  Example: `runestone/DESIGN_RUNESTONE.md`
- `component_spec` — natural language description of the interactive
  behaviour. Must name the specific Motion Moment being demonstrated
  or the specific interaction being implemented.
  Example: "A StampPreview component. Has a button labelled 'Press'.
  Clicking triggers scale(0.94) at 80ms linear, returns to scale(1)
  at 120ms linear. States: idle | stamping. Shows the Stamp moment
  from the Press Rune."

**Optional:**
- `output_path` — override the default output directory.

## Output

One component file at `runestone/src/components/[ComponentName].tsx`
(see Current implementation for format).

## Press motion rules the output must satisfy

- Easing: `cubic-bezier(0.0, 0.0, 0.2, 1)` or `linear` — no others
- Duration ceiling: 300ms maximum per animation step
- Named Moments must match the Design Contract exactly:
  - Press Mark: clip-path wipe left→right, 280ms linear, nav only,
    first mount only
  - Hard Land: translateY(-6px)→0 + opacity 0→1, 160ms hard decelerate
  - Stamp: scale(0.94)→1, 80ms out + 120ms return, linear both ways
- No spring physics
- No `ease-in-out` or `ease` — too organic for Press
- Hover states: colour/opacity transition only, no transform

## Failure modes

**Motion library required:** Stop. Report which behaviour requires the
library and why CSS cannot achieve it. Await human decision.

**Component needs data fetching:** Stop. Report that data must come
through props from the parent page. Describe what props are needed.

**Motion spec violates Press rules:** Stop. Report the specific
violation — duration, easing, or a Moment that doesn't match the
contract. Do not implement non-Press motion.

**Scope creep:** If the spec implies multiple interactive components
or a full interactive section — stop. One island per dispatch.

---

## Current implementation

**Current stack:** Astro 6 + React 18 islands
**File format:** `.tsx` (React functional component)
**Hydration:** `client:load` directive in the parent Astro page
**CSS approach:** `<style>` tag embedded in the component JSX,
using `--pr-*` custom properties. Inline `style` props only for
dynamic values computed from state or props.
**State:** React `useState` for component state, `useEffect` only
when needed for lifecycle (e.g. animation cleanup)

One island = one self-contained interactive unit. An island may
include its own internal motion states (e.g. idle → stamping →
idle), but must not bundle multiple unrelated UI pieces into one
dispatch. If the spec describes two independent interactive
behaviours, dispatch this skill twice.

Component structure:
```tsx
import { useState } from 'react';

interface Props {
  // typed props
}

export default function ComponentName({ ... }: Props) {
  const [state, setState] = useState<'idle' | 'active'>('idle');

  return (
    <>
      <div
        className="island-root"
        style={{ /* dynamic values only */ }}
      >
        {/* markup */}
      </div>
      <style>{`
        .island-root {
          border-radius: 0;
          /* static styles using --pr-* tokens */
        }
        @keyframes named-moment {
          /* keyframes from Design Contract, exact values */
        }
      `}</style>
    </>
  );
}
```

Usage in an Astro page:
```astro
---
import ComponentName from '../components/ComponentName';
---
<ComponentName client:load prop="value" />
```

**If the stack changes:** update this section only — the contract,
motion rules, and Press constraints above remain stable.
