---
name: accessibility/component
version: 1.0.0
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
2. Identifies the widget type (button, dialog, listbox, combobox, etc.)
3. Evaluates the component against the rules below.
4. Writes a report to
   `sanctum/00 Inbox/accessibility-component-[name]-[date].md`.
5. Reports: "Component: [name] · Widget type: [type] · Violations: [N]
   · APG keyboard model: [pass/fail/not-applicable]."

## What it does NOT do

- Does not rewrite the component. It reports violations for the
  `react/component` skill to address.
- Does not validate visual design — contrast ratios, colour blindness
  considerations are out of scope here. Those belong in design/validator.
- Does not replace a screen reader test. This skill catches structural
  and semantic issues; runtime testing with actual assistive technology
  is a separate step.

## Inputs

**Required:**
- `file_path` — path to the component TSX file.
- `widget_type` — the ARIA widget category this component represents.
  One of: `button`, `dialog`, `listbox`, `combobox`, `menu`, `tooltip`,
  `checkbox`, `radio`, `switch`, `slider`, `tablist`, `disclosure`,
  `form-field`, `display` (non-interactive).

**Optional:**
- `component_spec` — the original spec used to produce the component.
  Helps identify intended behaviour versus implemented behaviour.

## Output

One markdown report at
`sanctum/00 Inbox/accessibility-component-[name]-[date].md`.

Each violation format:
```
Rule: [rule name]
Widget type: [widget]
Location: [file:line or description]
Found: [what the component does]
Required: [what it must do instead]
APG reference: [relevant APG pattern if applicable]
```

## Rules

### Semantic HTML first

- Use the native HTML element whose semantics match the component's role.
  `<button>` for buttons. `<a>` for navigation links. `<input>` for
  text entry. `<select>` for single-value selection when native is
  sufficient. Native elements get keyboard interaction, focus, and ARIA
  semantics for free.
- Add ARIA only when native HTML cannot express the required semantic.
  ARIA augments — it does not replace native semantics.
- Never place interactive ARIA roles on non-interactive elements without
  also implementing the full keyboard contract for that role.

### ARIA rules

- Every `aria-*` attribute must be a valid ARIA attribute name.
  `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-expanded`,
  `aria-selected`, `aria-disabled`, `aria-hidden` are valid.
  Misspellings (`aria-lable`, `aria-discription`) are silent failures.
- ARIA names match the HTML specification exactly. Do not invent
  `aria-*` attributes.
- Interactive elements that lack a visible text label must have
  `aria-label` or `aria-labelledby`. An icon-only button is a common
  failure point.
- `aria-hidden="true"` removes an element from the accessibility tree.
  Do not apply it to elements that contain focusable children.
- `role="presentation"` and `role="none"` strip semantics. Use them
  only on structural wrappers where the element's native role is
  misleading (e.g. a `<table>` used for layout, not data).

### Keyboard interaction — by widget type

**Button:**
- `Enter` activates. `Space` activates.
- Focus stays on the button after activation unless the action
  removes it from the page.
- If the button opens something (dialog, menu), focus moves inside
  the opened content.

**Dialog (modal):**
- On open: focus moves to the first focusable element inside the
  dialog, or to the dialog container if no focusable element exists.
- While open: `Tab` and `Shift+Tab` cycle through focusable elements
  inside the dialog only. Focus must not leave the dialog.
- `Escape` closes the dialog. On close: focus returns to the element
  that triggered the dialog.
- Content behind the dialog must be inert (`aria-modal="true"` on the
  dialog, or `inert` attribute on the background).

**Listbox:**
- Only one tab stop in the listbox. Arrow keys move between options.
- `Home` moves to the first option. `End` moves to the last.
- `Enter` or `Space` selects the focused option.
- Type-ahead: typing a character moves focus to the next option
  starting with that character.
- Use either roving `tabindex` (set `tabindex="0"` on the focused
  option, `tabindex="-1"` on others) or `aria-activedescendant`
  (keep focus on the container, update `aria-activedescendant` to
  the ID of the active option). Choose one pattern and apply it
  consistently.

**Combobox:**
- The input has `role="combobox"`, `aria-expanded`, `aria-haspopup`,
  and `aria-controls` pointing to the listbox.
- Arrow keys open the listbox and navigate options.
- `Enter` selects. `Escape` collapses.

**Tooltip:**
- Triggered by hover and focus. Never by click alone.
- `role="tooltip"` on the tooltip element.
- `aria-describedby` on the trigger pointing to the tooltip element.
- `Escape` dismisses.

**Form field:**
- Every input has an associated `<label>` via `for`/`id`, `aria-label`,
  or `aria-labelledby`. Placeholder text is not a label.
- Error messages are associated via `aria-describedby`.
- Invalid fields have `aria-invalid="true"`.
- Required fields have `required` on the native element or
  `aria-required="true"`.

### Focus management

- Interactive elements must be reachable by keyboard.
- Custom interactive components not built on native HTML elements must
  have `tabindex="0"` to be keyboard-focusable.
- Disabled elements should use `aria-disabled="true"` rather than
  the `disabled` HTML attribute when the element should remain in the
  tab order (e.g. with a tooltip explaining why it is disabled).
  Use `disabled` when removing it from the tab order is correct.
- Focus indicators must be visible. Do not suppress the browser's
  default focus ring without replacing it with an equally visible
  custom indicator.

### State exposure for styling

- Interaction state must be exposed through `data-*` attributes so
  Press CSS can target behavior without coupling logic and visuals:
  - `data-hovered` — pointer hover state
  - `data-pressed` — active press state
  - `data-focused` — keyboard or pointer focus state
  - `data-focus-visible` — keyboard focus only (not pointer)
  - `data-disabled` — disabled state
  - `data-selected` — selection state (listbox, tabs, etc.)
  - `data-expanded` — open/closed state (disclosure, combobox, etc.)
- These attributes are set by React Aria automatically when using
  React Aria primitives. Do not reimplement them manually.

### React Aria integration

- Interactive components must wrap a React Aria primitive rather than
  reimplementing keyboard and focus behavior.
- Verify that the React Aria primitive is correctly imported and used.
  `useButton` for buttons. `useDialog` for dialogs. `useListBox` for
  listboxes. etc.
- The React Aria primitive's return value (`buttonProps`, `dialogProps`,
  etc.) must be spread onto the correct DOM element.
- Do not override event handlers that React Aria provides unless the
  override is intentional and documented.

## Failure modes

**Widget type unknown:** Stop. Report that the widget type is required
to evaluate the keyboard model. Ask the caller to specify it.

**Native HTML element available but not used:** Flag the gap. Report
which native element should be used and why the ARIA-only approach
is insufficient.

**React Aria primitive not used for interactive component:** Flag as a
high-severity violation. Reimplementing keyboard and focus behavior
from scratch is a reliability and maintenance risk.

**Scope creep:** Evaluate one component at a time. If the file contains
multiple components, evaluate only the one named in the input.

---

## Sources

Rules derived from:
- `sanctum/40 Sources/Accessibility/wai-aria-authoring-practices-guide.md`
- `sanctum/40 Sources/Accessibility/adobe-react-aria-styling.md`
- W3C WAI-ARIA Authoring Practices Guide: Button, Dialog, Listbox,
  Combobox, Tooltip patterns
- React team docs: Invalid ARIA Prop Warning, Common DOM components
- Adobe React Aria: state exposure via data-* attributes
