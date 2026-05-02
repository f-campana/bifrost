---
name: frontend/prose-route
version: 1.0.0
type: producer
domain: frontend
tier: action
triggers:
  - /frontend-prose-route
  - "build a prose route"
  - "create a /spec route"
  - "render markdown into the product shell"
reads_from: Design Contract (DESIGN_[PROJECT].md) + markdown source content + route specification
writes_to: runestone/src/pages/[route]/[slug].astro + runestone/src/styles/[rune]-prose.css
model_preference: claude-sonnet
---

# frontend/prose-route

## When to use

Use when a Rune needs a documentation route such as `/spec` that renders markdown
or MDX inside the product shell and the rendered output must conform to the
Design Contract for the active Rune. Typical trigger: a Rune needs both an
experience surface and a reference surface, and the reference surface uses
long-form markdown including code blocks, tables, lists, blockquotes, and
images.

This skill produces documentation routes scoped to the product shell rather
than free-floating markdown viewers. It is not for pure markdown viewers with
no design contract, JSON-driven docs, or components that render one or two
markdown fields inline. For inline prose inside an existing component, use the
inline prose pattern in `frontend/component` instead.

## What it does

1. Reads the Design Contract in full before writing any code.
2. Reads the markdown source content to identify which prose elements actually
   appear: `h1`-`h6`, `p`, `code`, `pre`, `ul`, `ol`, `table`, `blockquote`,
   `hr`, `img`.
3. Produces two files following the Current implementation section below.
4. The route file:
   - Lives at `runestone/src/pages/[route]/[slug].astro`
   - Imports the prose stylesheet
   - Wraps `<Content />` in a class-scoped prose wrapper
   - Renders the page header inside the product shell
   - Renders supporting components when the route specification calls for them
5. The prose stylesheet:
   - Lives at `runestone/src/styles/[rune]-prose.css`
   - Maps every prose element that appears in the source content to Rune tokens
   - Neutralises third-party syntax-highlighting output when present
   - Uses no arbitrary values outside the active Rune token set unless the
     Design Contract explicitly defines them
6. Writes a run report to
   `sanctum/00 Inbox/frontend-prose-route-[name]-[date].md`.
7. Reports: "Route: [route] · Source: [markdown file] · Elements covered:
   [list] · Supporting components: [list or none] · Lines: [N route + N CSS]
   · Press rules applied: [list each rule checked and whether it passed or
   was not applicable]."

## What it does NOT do

- Does not invent design decisions not present in the Design Contract.
- Does not rely on framework markdown defaults for any prose element.
- Does not use third-party syntax highlighting themes without explicit
  override; the Rune's Chromatism score determines whether token colour is
  permitted.
- Does not produce a generic prose stylesheet; every rule binds to Rune
  tokens, not arbitrary values.
- Does not skip any element type that appears in the source content.

Operating rules:
- Do not rely on generic markdown defaults for headings, lists, tables, code, blockquotes, or inline code.
- Create a dedicated prose stylesheet for the route and map every prose element to rune tokens.
- If the markdown pipeline emits syntax-highlighted code, neutralise third-party token colours or disable them before delivery.
- Report every framework default you had to override: markdown prose, syntax highlighting, list markers, table borders, and code backgrounds.

## Inputs

**Required:**
- `design_contract` — path to the Design Contract.
  Example: `runestone/DESIGN_RUNESTONE.md`
- `markdown_source` — path to the markdown content file that will be rendered.
- `route_path` — destination route.
  Example: `runes/press/spec`
- `rune_namespace` — token prefix for the active Rune.
  Example: `pr` for Press

**Optional:**
- `supporting_components` — list of components to render above or alongside the
  prose body. Example: `PressGallery` for `/runes/press/spec`
- `prose_max_width` — defaults to `680px`; override only when the Design
  Contract specifies a different value

## Output

Two files:
- `runestone/src/pages/[route]/[slug].astro` — route file that imports the
  prose stylesheet, renders the page header, wraps `<Content />` in the scoped
  prose container, and renders any supporting components named in the route
  specification.
- `runestone/src/styles/[rune]-prose.css` — rune-scoped prose stylesheet that
  styles every prose element present in the markdown source using Rune tokens
  and overrides third-party markdown or syntax-highlighting defaults as needed.

## Press rules the output must satisfy

Before writing the route and stylesheet, verify each inviolable rule:

1. Orange (`--pr-orange`) used only as a structural element — section eyebrow
   rules, link colour, or other load-bearing markers only; never decorative
2. No border-radius declarations — code blocks, table cells, blockquotes, and
   related prose chrome stay at 0
3. All explicit borders are `2px solid` — code blocks, tables, cells,
   blockquote rails, and rules never drop to 1px
4. Light-first surfaces — prose backgrounds, especially code blocks, remain
   light; dark Shiki themes are rejected
5. No second accent colour — syntax-highlighted code must not reintroduce a
   second accent; neutralise token colours when the Rune's Chromatism requires
   near-monochrome output

Concrete check from `press.md` section 10 anti-patterns:
- [ ] Syntax-highlighted markdown code blocks render with third-party theme colours or dark backgrounds → override Shiki/rehype code colours to `var(--pr-text)` and neutral Press surfaces before delivery

If the route specification or markdown pipeline would require violating any
rule, stop and report the conflict before writing any code.

## Failure modes

**Design Contract not found:** Stop. Report the exact path provided.
Do not proceed without the contract.

**Markdown source contains an element type the stylesheet does not cover:**
Stop. Report the missing element and add coverage before delivery.

**Shiki override missing:** Stop. Report that multicolour syntax highlighting
violates Press Rule 5 and Chromatism 2.

**Wrapper class missing on `<Content />`:** Stop. Report that the prose styles
will not apply at all.

**Stylesheet imported but wrapper class is wrong:** Stop. Report the mismatch
(`press-prose-content` instead of `press-prose`, for example) because the
styles will silently miss.

**Prose width exceeds the Design Contract max:** Stop. Report the measured or
configured width and the contract limit before delivering.

---

## Current implementation

**Framework:** Astro 6 (content collections + `render()` for markdown / MDX
routes)
**Route file format:** `.astro` page with TypeScript frontmatter
**CSS approach:** Rune-scoped external stylesheet imported into the route file
**Wrapper convention:** `[rune-namespace]-prose`
**Package manager:** pnpm

The canonical implementation is `runestone/src/styles/press-prose.css` paired
with `runestone/src/pages/runes/press/spec.astro`. The route imports the prose
stylesheet, renders the route header, renders any supporting components, and
wraps `<Content />` in the scoped wrapper shown below.

Wrapper pattern:
```astro
<div class="press-prose">
  <Content />
</div>
```

The wrapper class follows `[rune-namespace]-prose`. For Press, use
`press-prose`. For a future Neon Humaniste rune, use `neon-prose`.

Canonical Shiki override from `runestone/src/styles/press-prose.css`:
```css
/* Rule 5 — one structural colour.
   Override Shiki token colours to enforce
   Press Chromatism 2 near-monochrome. */
.press-prose pre.astro-code,
.press-prose pre.astro-code code,
.press-prose pre.astro-code span {
  --shiki-light: var(--pr-text-1, var(--pr-text));
  --shiki-dark: var(--pr-text-1, var(--pr-text));
  --shiki-light-bg: transparent;
  --shiki-dark-bg: transparent;
  color: var(--pr-text-1, var(--pr-text)) !important;
  background-color: transparent;
}

.press-prose pre.astro-code {
  box-shadow: inset 0 0 0 9999px var(--pr-surface);
}
```

**If the stack changes:** update this section only. The contract above
remains stable.

---

## Sources

- `sanctum/00 Inbox/harvest-frontend-phase-1-2026-05-02.md`
  (Section 4, proposal 1; Section 3, case 1)
- `sanctum/10 Knowledge/Design/runes/press.md`
  (section 04 inviolable rules; section 06 typography; section 10 anti-patterns)
- `runestone/src/styles/press-prose.css`
- `runestone/src/pages/runes/press/spec.astro`
- `bifrost/skills/frontend/frontend-component/SKILL.md` (template reference)
