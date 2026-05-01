---
name: accessibility/component
version: 1.1.0
type: validator
domain: accessibility
tier: action
triggers:
  - /accessibility-component
  - "check accessibility of this component"
  - "apply accessibility rules to"
reads_from: any TSX component file + component specification
writes_to: sanctum/00 Inbox/accessibility-component-[name]-[date].md
model_preference: claude-sonnet
---

# accessibility/component

## When to use

Use when producing or reviewing a React component for the Press component
library that has interactive behaviour — buttons, dialogs, listboxes,
menus, tooltips, form fields, or any element that receives focus or
responds to keyboard input. This skill is a validator: it checks an
existing component file against accessibility requirements and reports
violations. It does not produce components.

Always load this skill alongside `react/component` for interactive
component dispatches. For purely decorative or display components with
no interactivity, this skill is not required.

## What it does

1. Reads the component file and the component specification.
2. Identifies the widget type.
3. Evaluates the component against the rules below.
4. Writes a report to
   `sanctum/00 Inbox/accessibility-component-[name]-[date].md`.
5. Reports: "Component: [name] · Widget type: [type] · Violations: [N]
   · APG keyboard model: [pass/fail/not-applicable]
   · Base UI primitive: [used/missing/not-applicable]
   · Runtime tests required: [yes/no]."

## Scope of this skill

This skill validates structural correctness only: ARIA attributes,
correct primitive usage, keyboard contract, data-* or CSS state exposure.

**It cannot validate runtime behaviour.** Disabled state semantics —
whether a click handler still fires when disabled, whether focus
management works correctly on dialog open/close — require tests or
code review, not static analysis. The run report must explicitly flag
when runtime test coverage is required. Static validation is necessary
but not sufficient.

## What it does NOT do

- Does not rewrite the component.
- Does not validate visual design — contrast, colour blindness.
  Those belong in design/validator.
- Does not replace a screen reader test.
- Does not replace runtime tests for disabled state and focus management.

## Inputs

**Required:**
- `file_path` — path to the component TSX file.
- `widget_type` — one of: `button`, `dialog`, `listbox`, `combobox`,
  `menu`, `tooltip`, `checkbox`, `radio`, `switch`, `slider`, `tablist`,
  `disclosure`, `form-field`, `display`.

**Optional:**
- `component_spec` — the original spec.

## Output

One markdown report at
`sanctum/00 Inbox/accessibility-component-[name]-[date].md`.

Violation format:
```
Rule: [rule name]
Widget type: [widget]
Location: [file:line or description]
Found: [what the component does]
Required: [what it must do]
APG reference: [if applicable]
```

The report must end with a Runtime tests section stating which
disabled-state and focus-management behaviours require tests.

## Rules

### Semantic HTML first

- Use the native HTML element whose semantics match the component's role.
- Add ARIA only when native HTML cannot express the required semantic.
- Never place interactive ARIA roles on non-interactive elements without
  implementing the full keyboard contract.

### ARIA rules

- Every `aria-*` attribute must be a valid ARIA attribute name.
  Misspellings are silent failures — flag every one.
- Interactive elements without visible text labels must have
  `aria-label` or `aria-labelledby`. Icon-only buttons are the
  most common failure point.
- `aria-hidden="true"` must not be applied to elements containing
  focusable children.

### Keyboard interaction — by widget type

**Button:**
- `Enter` activates. `Space` activates.
- If the button opens something, focus moves inside the opened content.
- Focus stays on the button after activation unless the action removes
  it from the page.

**Dialog (modal):**
- On open: focus moves to the first focusable element inside, or the
  dialog container.
- While open: `Tab` and `Shift+Tab` cycle inside only. Focus must not
  leave the dialog.
- `Escape` closes. On close: focus returns to the trigger.
- Content behind the dialog must be inert.

**Listbox:**
- One tab stop. Arrow keys move between options.
- `Home` → first. `End` → last. `Enter`/`Space` selects.
- Type-ahead: character key moves to next matching option.
- Roving tabindex or aria-activedescendant — choose one consistently.

**Combobox:**
- Input: `role="combobox"`, `aria-expanded`, `aria-haspopup`,
  `aria-controls` pointing to listbox.
- Arrow keys open and navigate. `Enter` selects. `Escape` collapses.

**Tooltip:**
- Triggered by hover and focus. Never click alone.
- `role="tooltip"`. `aria-describedby` on the trigger.
- `Escape` dismisses.

**Form field:**
- Every input has an associated label — `for`/`id`, `aria-label`,
  or `aria-labelledby`. Placeholder is not a label.
- Error messages via `aria-describedby`.
- Invalid: `aria-invalid="true"`. Required: `required` or
  `aria-required="true"`.

### Focus management

- Interactive elements must be reachable by keyboard.
- Custom components not built on native HTML must have `tabindex="0"`.
- Focus indicators must be visible. Press components use:
  `focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2
   focus-visible:outline-[var(--pr-focus-ring)]`
  Do not suppress this.

### Disabled state

The correct disabled pattern depends on tab-order intent:

**Remove from tab order (default):** spread Base UI primitive props with
`disabled`. Base UI applies `aria-disabled` and removes from tab order.

**Keep in tab order:** spread Base UI primitive props with
`focusableWhenDisabled={true}`. Use when a tooltip explains why disabled.

**Static validation checks:**
- Is the Base UI primitive used? If yes, `disabled` handling is correct
  by construction.
- Are there custom DOM `click` handlers on the component? If yes, flag
  for runtime test — static analysis cannot verify they respect disabled.
- Is `disabled` being manually stripped from props? Flag as
  high-severity — this was the P1 pattern in PressButton v1.

**Runtime test requirement (always flag for interactive components):**
Tests must cover: keyboard activation blocked when disabled; click
handler not invoked when disabled; `focusableWhenDisabled` keeps element
in tab order; `focusableWhenDisabled=false` removes it.

### Interaction state styling

**Simple components (Button, Badge, Tag, Input):**
Interaction states should use CSS pseudo-classes: `hover:`, `focus-visible:`,
`disabled:`, `active:`, `aria-invalid:`. Flag the following as violations
for simple components:
- `useHover`, `useFocusRing`, or `useFocusWithin` hooks
- Manual `data-hovered`, `data-pressed`, `data-focused` spreading
These add complexity that CSS handles natively.

**Complex composites (Dialog, Select, Listbox, Menu):**
Base UI exposes `data-*` attributes automatically where CSS pseudo-classes
are insufficient. Verify these are used rather than custom implementations.

### Base UI primitive integration

- Interactive components must extend a Base UI primitive from
  `@base-ui/react`.
- Verify the correct primitive is imported and that the component
  extends `[Primitive]Primitive.Props`.
- The primitive's props must be spread via `...props` so consumers
  receive the full API including `focusableWhenDisabled`.
- Flag as high-severity: interactive component that does not use a
  Base UI primitive and reimplements keyboard or focus behavior.
- Flag as high-severity: use of react-aria hooks (useButton, useHover,
  useFocusRing) — these are not the default primitive layer per ADR D2.

## Failure modes

**Widget type unknown:** Stop. Ask the caller to specify it.

**Native HTML available but not used:** Flag. Report which native element
to use.

**Base UI primitive missing for interactive component:** Flag high-severity.
Report which Base UI primitive covers this behavior.

**Manual disabled attribute manipulation:** Flag high-severity. Correct
pattern is `focusableWhenDisabled` on the Base UI primitive.

**react-aria hooks found:** Flag as ADR D2 violation. Report the Base
UI equivalent.

**Custom click handler present with disabled state:** Flag for runtime
test — cannot be verified statically.

**Scope creep:** Evaluate one component at a time.

---

## Sources

- `sanctum/10 Knowledge/Concepts/press-component-architecture-adr.md`
- `sanctum/40 Sources/Accessibility/wai-aria-authoring-practices-guide.md`
- `sanctum/40 Sources/Accessibility/adobe-react-aria-styling.md`
- W3C WAI-ARIA Authoring Practices Guide: Button, Dialog, Listbox,
  Combobox, Tooltip patterns
- React team docs: Invalid ARIA Prop Warning, Common DOM components
- Base UI source: `@base-ui/react` Button component
