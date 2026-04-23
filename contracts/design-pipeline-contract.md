# Design Pipeline Contract

**Location:** `Bifrost/contracts/design-pipeline-contract.md`  
**Version:** 0.1.0  
**Status:** Phase 0 — Validated. Ready to build.  
**Last updated:** 2026-04-22

---

## Overview

This contract governs how aesthetic intent travels from a client brief into executable code through agents that have zero prior context. It defines the vocabulary, the file formats, the pipeline stages, the quality gates, and the skill responsibilities for all design work performed by agents within this system.

The pipeline is the first fully-specced instance of a general pattern that will recur across all technical domains in Bifrost. The pattern: deep research → conceptual modelling → iterative validation → spec. The design domain required this investment because agents without constraints default to generic output. The same investment applies to any domain where agent failures are specific, recurring, and consequential.

The core insight the pipeline encodes: agents do not generate taste. They execute yours. A Rune is your taste, codified. The pipeline is the mechanism by which that codification travels reliably from brief to deployed interface.

---

## Vocabulary

These terms are canonical throughout this system. Do not substitute synonyms.

**Rune** — A named, versioned design identity. Encodes visual DNA, motion system, routing governance, and the annotated radar for a specific archetype profile. Lives in Sanctum. Reusable across projects that share the same archetype profile. The agent reads the Rune to understand creative intent. Plural: Runes.

**Runestone** — The browsable web interface that displays the Rune library. Renders each Rune as a live experience: type specimens, colour swatches, motion moment previews, radar visualisation, routing guide. Built using the design pipeline itself (dogfood principle). Not deferred — the Runestone validates the pipeline as it is built.

**Design Contract** — The project-specific, agent-executable file instanced from a Rune for a given project. Lives at the project root as `DESIGN_[PROJECT].md`. Contains YAML frontmatter (Google spec compliant) and markdown body. The single source of truth every agent reads before touching any UI in that project. Complete and self-contained — design/builder requires no other input.

**Axes** — Eight visual/perceptual dimensions that define the measurable DNA of a Rune. Each axis has a score (1–10) and a one-sentence interpretation specific to how that Rune expresses it. Directly executable: an agent reading axis scores and interpretations can produce CSS without intermediate translation.

**Archetype** — A Carol Pearson archetype label (1–2 per Rune) indicating the psychological and cultural register the Rune operates in. Governs creative direction — *why* decisions are made. The axes govern *what* those decisions look like. Archetypes are a pointer, not a measurement system.

**Moments** — Named, timed animation events that are part of a Rune's identity. 2–5 per Rune. Each moment expresses a specific aspect of the archetype profile. Not decorative polish — semantic expression through motion.

**Routing Guide** — A structured set of questions that map brief characteristics to Rune candidates. Lives in Sanctum. Used by the human before dispatching design/curator. Not an agent step — the human consults the routing guide and selects the Rune.

**Run Report** — A structured markdown file produced by design/builder alongside every HTML output. Documents which Rune was used, which Design Contract version, which sections were generated, which anti-patterns were detected and suppressed, which inviolable rules were verified. Feeds design/harvester.

---

## The Eight Axes

These are the eight visual/perceptual dimensions used to score every Rune. The names and pole labels are fixed. The scores and interpretations are authored per Rune.

| Axis | Code | Low pole | High pole |
|---|---|---|---|
| Density | DEN | Airy — generous white space | Dense — information-first |
| Temperature | TEM | Cold — clinical, corporate | Warm — human, sensorial |
| Luminosity | LUM | Dark-first — deep backgrounds | Light-first — bright surfaces |
| Energy | ENE | Static — calm, measured | Expressive — kinetic, vivid |
| Geometry | GEO | Organic — curves, fluidity | Geometric — precision, angles |
| Chromatism | CHR | Monochrome — one accent | Polychrome — many accents |
| Depth | DEP | Flat — no layers or shadows | Layered — elevation, halos |
| Posture | POS | Utility — tool recedes | Experience — interface asserts |

**Reading scores:** A score of 2 on LUM means dark-first. A score of 8 on TEM means warm. A score of 9 on POS means the interface is the experience — it does not recede behind content. Each score is followed by a one-sentence interpretation written specifically for this Rune.

**Axis scores are not emotional descriptions.** They are directly executable CSS decisions. An agent reading DEN 3 knows: generous spacing, breathing room, never cramped. An agent reading CHR 2 knows: one accent colour, strict roles, no mixing.

---

## Token Format

All Rune and Design Contract colour tokens are written in OKLCH with a two-letter namespace prefix derived from the Rune slug. Hex values are shown as secondary reference only.

```css
/* Format: --[namespace]-[token-name]: oklch([L%] [C] [H]); */
--sb-surface-bg:     oklch(8% 0.005 240);    /* #0d0d0d */
--sb-accent-yellow:  oklch(95% 0.25 110);    /* #E2FF00 */
--sb-accent-pink:    oklch(55% 0.25 10);     /* #FF1F5B */
```

**Why OKLCH:** Perceptual uniformity — luminosity values map directly to the LUM axis score. `oklch(8% ...)` corresponds to LUM 2. `oklch(95% ...)` corresponds to LUM 9. Colour system and measurement system share vocabulary. Accessibility calculations (contrast ratios) are more reliable in OKLCH than in sRGB hex.

**Namespace convention:** Two-letter prefix from the Rune slug.

| Rune | Slug | Namespace |
|---|---|---|
| Argent Décisif | argent-decisif | `--ae-` |
| Néon Humaniste | neon-humaniste | `--nh-` |
| Signal Brut | signal-brut | `--sb-` |
| Encre Éditoriale | encre-editoriale | `--ee-` |

New Runes follow the same pattern. Two-letter prefixes must be unique across the Rune library. Collision check: run `grep -r "^--[prefix]-"` across all Design Contracts before registering a new prefix.

---

## Rune Anatomy

A Rune is a markdown file in `Sanctum/10 Knowledge/Design/runes/[slug].md`. It is a compiled knowledge artifact — authored by a human, versioned, and evolved through the distillation loop. Agents read it. Humans author and review it.

### Required sections in order

**YAML Frontmatter**
```yaml
---
name: Signal Brut
slug: signal-brut
version: 1.0.0
status: stable          # stable | draft | deprecated
archetypes: [Outlaw, Creator]
namespace: sb
radar: "4-3-7-5-9-6-8-2"
test_sentence: "Does this look like an underground concert poster printed by someone who knows typography?"
instances: 3            # incremented by design/curator on each instancing
first_instanced: 2026-04-21
---
```

**01 — Essence**
Three to five sentences. The emotional and cultural register of the Rune. Who is this for. What it refuses to be. The test sentence explained. This section is what the human reads to decide if a Rune fits a brief.

**02 — Archetypes**
One paragraph per archetype (primary, secondary). Describes specifically how each archetype manifests visually in this Rune — not generic archetype theory, but this Rune's specific expression.

**03 — Radar**
Eight axis scores with one-sentence interpretation each. Format:

```
Density     3 — Airy — information has room to breathe
Temperature 3 — Cold — no warmth, no comfort, deliberate harshness
Luminosity  2 — Dark-first — #0d0d0d background, no light-first variant
Energy      7 — Kinetic — hard shadows, blinking cursors, sweep animations
Geometry    5 — Mixed — 0px radius on all containers, curves reserved for type
Chromatism  7 — Two accents in tension — yellow and pink, never mixed
Depth       6 — Physical — hard drop shadows, 4px offset, no blur
Posture     8 — Assertive — the interface demands attention
```

**04 — Inviolable Rules**
3–5 binary, testable conditions. If any one is absent, the output is not this Rune. Written as decision trees, not prose preferences:

```
Rule 1 — Background:
Is the hero section background yellow (#E2FF00)?
  Yes → proceed
  No  → STOP. The hero must be yellow. Fix before continuing.

Rule 2 — Shadows:
Does every CTA and interactive card have box-shadow: 4px 4px 0 #000000?
  Yes → proceed
  No  → STOP. Shadows must be hard offset with no blur. Fix before continuing.

Rule 3 — Radius:
Is border-radius 0 on every container, button, and badge?
  Yes → proceed
  No  → STOP. No rounded corners in this Rune. Fix before continuing.

Rule 4 — Borders:
Are all borders 2px solid?
  Yes → proceed
  No  → STOP. 1px borders are invisible. Fix before continuing.
```

**05 — Canvas**
Background colour hierarchy, text colours, and accent colour roles. Each value in OKLCH with namespace prefix. Hex shown as secondary reference. Each colour has one sentence defining its role and one sentence defining what it is never used for.

**06 — Typography**
Three-font triad (display · body · mono) with exact weights, scale, tracking, and leading per level. Usage rules — which font appears in which context, and which contexts are explicitly forbidden for each font.

**07 — Spatial**
Base unit, page margins, section padding, component padding, gap conventions. One rule per line. No prose.

**08 — Motion**
Named moments (2–5). Each moment:
```
Name: Sweep Flare
Trigger: CTA button hover or focus
Timing: 400ms ease-expressive
Effect: Accent colour sweeps across button surface left to right
Active state: translateY(1px) — physical press feedback
Semantic meaning: Deployment is a decisive act, not a passive click
Exclusive use: Primary CTA only — never on secondary buttons or links
```

UI state machine if applicable (3 states with named transitions and timing).

Generic motion rules that apply throughout: easing family for entrances, easing for on-screen transitions, hover feedback standard, what must never animate.

**09 — Components**
Spec for the 5–8 most critical components: button variants, navigation, cards, badges, data tables (if applicable), form inputs (if applicable). Each component has: CSS properties with token references, hover/active/focus states, what this component must never do.

**10 — Anti-Patterns**
Binary checklist. Each item is a statement of what must not exist in any output generated from this Rune. Written so design/validator can check them mechanically:

```
- [ ] Hero section on dark background → must be yellow
- [ ] Soft box-shadow with blur > 0 on interactive elements → hard shadow only
- [ ] border-radius > 0 on any container → radius must be 0
- [ ] Inter, Roboto, Space Grotesk, or system-ui → Bebas Neue / DM Sans only
- [ ] Yellow and pink used in the same visible section → never simultaneous
- [ ] Gradient of any kind → no gradients
```

**11 — Routing**
When to use this Rune. When to use a different one instead, and which one.

```
Use Signal Brut for:
- Products that explicitly reject algorithmic curation
- Media monitoring, investigative tools, signal-heavy interfaces
- Brands that position adversarially against mainstream platforms
- Dark-first, dense, high-energy contexts

Use another Rune instead when:
- Audience is non-tech, 40-60, 6-8h/day → consider Aube Opérative
- Dashboard with density 9, polychrome → consider Prisme Analytique
- WCAG AAA required, accessible-first → consider Flux Bienveillant
- Light-first, institutional long-form → consider Encre Éditoriale
```

### Version semantics

`patch` (0.0.x) — Copywriting examples refined, anti-patterns clarified, spatial values tweaked. No visual change.
`minor` (0.x.0) — New component rules, colour value adjusted within same hue, new anti-pattern added. Existing Design Contracts remain valid.
`major` (x.0.0) — Archetype profile changes, test sentence changes, axis scores shift by more than 2, new inviolable rule added. Existing Design Contracts must be re-instanced.

On major version bump: design/curator appends a migration note to `Sanctum/00 Inbox/` listing affected projects.

---

## Design Contract Anatomy

A Design Contract is `DESIGN_[PROJECT].md` at the project root. Instanced by design/curator from a Rune + brief. Stands alone — design/builder requires no other file.

### YAML Frontmatter

Google `design.md` spec compliant. Parseable by `npx @google/design.md lint`.

```yaml
---
version: alpha
name: Signal — Media Monitoring Platform
description: Professional media surveillance tool for journalists and editorial teams
rune: signal-brut@1.0.0
namespace: sb
instanced: 2026-04-21
instanced_by: design/curator
test_sentence: "Does this look like an underground concert poster printed by someone who knows typography?"

colors:
  surface-bg:      "oklch(8% 0.005 240)"     # --sb-surface-bg
  surface-raised:  "oklch(11% 0.006 240)"    # --sb-surface-raised
  surface-white:   "oklch(96% 0.003 100)"    # --sb-surface-white
  accent-yellow:   "oklch(95% 0.25 110)"     # --sb-accent-yellow
  accent-pink:     "oklch(55% 0.25 10)"      # --sb-accent-pink
  text-primary:    "oklch(96% 0.003 100)"    # --sb-text-primary
  text-secondary:  "oklch(55% 0.01 240)"     # --sb-text-secondary

typography:
  display:
    fontFamily: Bebas Neue
    fontSize: clamp(80px, 14vw, 200px)
    fontWeight: 400
    letterSpacing: -0.01em
    lineHeight: 0.88
  h2:
    fontFamily: Bebas Neue
    fontSize: clamp(48px, 7vw, 96px)
    fontWeight: 400
    letterSpacing: 0.01em
    lineHeight: 0.92
  body:
    fontFamily: DM Sans
    fontSize: 15px
    fontWeight: 400
    lineHeight: 1.65
  mono:
    fontFamily: JetBrains Mono
    fontSize: 10px
    fontWeight: 400

spacing:
  base: 4px
  page-margin: 40px
  section-v: 80px
  component: 16px

rounded:
  default: 0px
  none: 0px

components:
  button-primary:
    backgroundColor: "{colors.surface-bg}"
    textColor: "{colors.accent-yellow}"
    rounded: "{rounded.none}"
    padding: "13px 24px"
  button-primary-hover:
    backgroundColor: "{colors.accent-yellow}"
    textColor: "{colors.surface-bg}"
  button-ghost:
    backgroundColor: "transparent"
    textColor: "{colors.text-primary}"
    rounded: "{rounded.none}"
    padding: "12px 23px"
---
```

### Markdown Body Sections

The markdown body contains content that the Google spec cannot encode: emotional context, layout compositions, motion specs, canonical copy, and anti-patterns.

**Required sections in order:**

**## Identity**
Product name, product description in one sentence, target user in one sentence. Rune reference. Test sentence. Three values the product embodies.

**## Axes**
Eight axes reproduced from the Rune with scores and interpretations. Human-readable confirmation that the Rune maps correctly to this project.

**## Inviolable Rules**
The Rune's inviolable rules, reproduced verbatim. If the project requires an exception to any rule, it is stated explicitly here with justification. Exceptions require human authorship — design/curator does not generate exceptions.

**## Page Structure**
Section-by-section layout specification for the primary deliverable (landing page, dashboard, component). For each section:
- Background colour (token reference)
- Padding
- Layout composition (exact grid, proportions, what goes where)
- Typography applied
- Any section-specific rules

Background alternation map — the full sequence of section backgrounds for the page, top to bottom.

**## Components**
Project-specific component specs that extend or override the Rune's component section. Same format as Rune component specs.

**## Motion**
Rune's named moments reproduced. Project-specific motion additions if any. The UI state machine if applicable.

**## Canonical Copy**
4–6 verbatim phrases that are locked — not paraphrasable, not variant-able. Extracted from the brief by design/curator. Examples:

```
Hero headline:   "The feed without the filter."
Primary CTA:     "Request access."
Status line:     "Deployed. Tested. Live."
Pricing note:    "No free tier. No algorithm."
```

**## Anti-Patterns**
The Rune's anti-pattern checklist, reproduced verbatim. Project-specific additions if any.

**## Do's and Don'ts**
Google spec compliant. Practical guardrails in the imperative:
```
- Do use Bebas Neue only at 20px minimum
- Do apply the hard shadow to every CTA: box-shadow: 4px 4px 0 #000
- Don't mix yellow and pink accents in the same visible viewport
- Don't use border-radius on any element
```

---

## The Pipeline

Five stages in phase order. Each stage has: inputs, outputs, who runs it, and what it reads.

### Stage 0 — Routing (human)

**Who:** You.
**Input:** Client brief (natural language, minimum 5 sentences covering: product, audience, tone, anti-patterns, values).
**Reference:** `Sanctum/10 Knowledge/Design/routing-guide.md`
**Output:** Selected Rune (slug + version).

**The routing guide** has six questions in a deliberate order. Audience → posture → register eliminate the majority of candidates. Density → luminosity → sector discriminate between the remaining 1–3. Each answer shows which Runes qualify. You choose from what remains.

The routing guide is not an agent step. Design/curator does not select Runes. You select. Curator instances.

---

### Stage 1 — design/curator (agent, Phase 0)

**Reads:** Selected Rune file from Sanctum + client brief.
**Writes:** `DESIGN_[PROJECT].md` at project root.
**Trigger:** Manual dispatch — `claude /design-curator --rune [slug] --brief [brief-file]`

**What it does:**
1. Reads the Rune in full.
2. Reads the client brief.
3. Generates YAML frontmatter: tokens from the Rune, namespaced and OKLCH.
4. Generates markdown body: axes, inviolable rules, page structure from brief, motion from Rune, canonical copy extracted from brief, anti-patterns from Rune.
5. Increments `instances` counter in the Rune file.
6. Writes `DESIGN_[PROJECT].md`.
7. Reports: "Rune: [name] v[version] · Instanced: DESIGN_[PROJECT].md · Canonical copy: [6 phrases]."

**What it does not do:**
- Does not select a Rune (that's yours).
- Does not override inviolable rules.
- Does not add sections not defined in this contract.
- Does not generate exceptions to any Rune rule.

**Failure modes:**
- Brief too vague (fewer than 3 extractable values) → request clarification before proceeding.
- Rune version conflict (brief requirements contradict a Rune rule) → flag the specific conflict, pause, await instruction.

---

### Stage 2 — design/builder (agent, Phase 0)

**Reads:** `DESIGN_[PROJECT].md` only. No other inputs required.
**Writes:** `[deliverable].html` + `run-report.md` (both to `Sanctum/00 Inbox/` for review, then to project if approved).
**Trigger:** Manual dispatch — `claude /design-builder --contract DESIGN_[PROJECT].md`

**Execution order:**
1. Read `DESIGN_[PROJECT].md` in full before writing any code. Not mid-read.
2. Identify: test sentence, YAML tokens, page structure, inviolable rules, anti-patterns, canonical copy.
3. Set up `:root {}` with every YAML token. No deviations.
4. Import fonts exactly as specified.
5. Implement each section in the order listed in `## Page Structure`. No reordering, no skipping.
6. Write all canonical copy verbatim — never paraphrase.
7. Run mental anti-pattern checklist before closing `</html>`.
8. Apply the test sentence. If the output fails, identify the weakest section and fix it.
9. Write `run-report.md`.

**Run report format:**
```markdown
# Run Report — DESIGN_SIGNAL.md
Date: 2026-04-22
Rune: signal-brut@1.0.0
Sections generated: 9/9
Anti-patterns caught: 2
  - Soft shadow on CTA → replaced with 4px 4px 0 #000000
  - border-radius: 4px on badge → corrected to 0
Inviolable rules verified: 4/4
Test sentence result: PASS — hero is yellow, shadows are hard, no rounded corners
Notes: Section 6 (Testimonials) used Bebas Neue at 18px — below 20px threshold. Corrected to 22px.
```

**What it does not do:**
- Does not deviate from the Design Contract for aesthetic reasons.
- Does not add sections not defined in `## Page Structure`.
- Does not use Claude's default aesthetic (dark SaaS, purple gradients, generic cards).
- Does not use placeholder copy.

---

### Stage 3 — design/validator (agent, Phase 0)

**Reads:** Generated HTML + `DESIGN_[PROJECT].md`.
**Writes:** Validation report to `Sanctum/00 Inbox/`.
**Trigger:** Runs automatically after design/builder completes. Can also be dispatched standalone.

**Three check layers:**

**Layer 1 — Structural lint (Google spec)**
```bash
npx @google/design.md lint DESIGN_[PROJECT].md
```
Checks: broken token references, missing primary colour, contrast ratios, orphaned tokens, section order violations. Exit code 1 on errors.

**Layer 2 — Inviolable rule verification**
Each inviolable rule in `## Inviolable Rules` is checked as a binary pass/fail against the generated HTML. Checks are mechanical: CSS property values, font-family declarations, background-color on hero section, box-shadow values.

**Layer 3 — Anti-pattern scan**
Each item in `## Anti-Patterns` is checked against the generated HTML. Any failure is flagged with the exact line number and a corrective instruction.

**Output format:**
```
design/validator — DESIGN_SIGNAL.md
Structural lint:    PASS (0 errors, 1 warning: orphaned token --sb-surface-3)
Inviolable rules:   4/4 PASS
Anti-patterns:      0 failures
Test sentence:      PASS (manual — requires human review)
Recommendation:     Approve for review
```

---

### Stage 4 — design/dialogue (agent, Phase 1)

**Reads:** A single Rune file from Sanctum.
**Writes:** Nothing — conversational only.
**Trigger:** Manual dispatch — `claude /design-dialogue --rune [slug]`

Opens a conversation scoped to the selected Rune's files. The Rune answers questions about its own decisions by reading its own spec files. Tool calls are shown — the Rune shows its work. Replaces the need for a browsable documentation interface for individual Runes during Phase 1.

Example interaction:
```
You: What makes this Rune different from Signal Brut?
Rune: [reads axes section] The key difference is Posture (4 vs 8) and Luminosity (8 vs 2).
      Signal Brut is dark-first and assertive — the interface demands attention.
      This Rune is light-first and utility-oriented — it recedes behind content.
      [reads inviolable rules] Signal Brut has four hard visual rules.
      This Rune has different constraints: no hard shadows, generous white space.
```

---

### Stage 5 — design/harvester (agent, Phase 1)

**Reads:** All run reports in `Sanctum/00 Inbox/` since the last harvest.
**Writes:** Harvest report to `Sanctum/00 Inbox/harvest-[date].md`.
**Trigger:** Manual dispatch after every 3–5 design runs.

**What it produces:**
- Patterns in anti-pattern failures (which rules are repeatedly violated → candidate for stronger wording or new inviolable rule)
- Sections that consistently required manual correction → candidate for clearer spec in the Design Contract template
- Canon copy that consistently needed revision → candidate for tone rule update
- Emerging patterns not covered by existing Rune rules → candidate for new Rune or Rune update

The harvest report is a proposal document. It does not modify any Rune. You review and decide what to commit.

---

### Stage 6 — design/distill (agent, Phase 2)

**Reads:** Harvest report + target Rune file.
**Writes:** Proposed Rune update to `Sanctum/00 Inbox/`.
**Trigger:** Manual dispatch after human review of harvest report.

Applies approved harvest findings to the Rune file. Increments version appropriately (patch/minor/major per the version semantics defined in Rune Anatomy). If a major version bump is required, flags all affected Design Contracts for re-instancing.

The human commits the proposed update. design/distill never commits directly to a Rune.

---

## Runestone — The Browsable Interface

The Runestone is the web interface that renders the Rune library as a live experience. It is not documentation — it is a working demonstration of each Rune's identity. Navigating the Runestone is how you choose a Rune for a new project.

**Phase 1 deliverable.** Built using the design pipeline itself (dogfood principle). The Runestone is the first real project that runs the full Phase 0 pipeline.

### What each Rune page shows

Every Rune page has the following sections, in this order:

1. **Header** — Rune name, version, status, archetype tags, radar summary (8 scores), test sentence.
2. **Identity** — The essence section prose + 4–5 principle cards.
3. **Radar** — Eight axes with gradient visualisation bars and per-axis interpretations.
4. **Colour** — All tokens with OKLCH swatches, namespace prefixes, hex reference.
5. **Typography** — Live specimens at each scale level. Real copy from the canonical phrases.
6. **Motion** — Named moments with live interactive previews. Each moment playable inline.
7. **Components** — Key components rendered live: buttons in all states, cards, badges.
8. **Routing** — When to use / when not to use, with links to alternative Runes.

### The selection guide

At `/guide` — six questions that filter the Rune library. Questions in order: audience → posture → register → density → luminosity → sector. Each answer eliminates candidates and shows which remain. Links directly to Rune pages. Used before dispatching design/curator.

### Technical spec

- **Framework:** Next.js (App Router) or Astro — decision at build time.
- **Data source:** Rune markdown files from Sanctum, read at build time.
- **Design Contract:** The Runestone has its own Design Contract. The Runestone picks the Rune that best fits a design-tool-for-designers context at the time of build.
- **Deployment:** Vercel or static hosting. Not Tachyon.
- **Motion:** Framer Motion for named moment previews. No other animation library.

---

## Sanctum Storage

```
Sanctum/
  10 Knowledge/
    Design/
      runes/
        argent-decisif.md
        neon-humaniste.md
        signal-brut.md
        encre-editoriale.md
        [slug].md            ← future Runes
      routing-guide.md       ← six-question selection guide
      design-principles.md   ← cross-Rune principles (Emil's motion grammar, etc.)
  40 Sources/
    Design/
      sessions/
        design-pipeline-session-2026-04-21.md  ← this conversation summary
      references/
        google-design-md-spec.md
        animations-dev-skill-notes.md
        ropau-forge-analysis.md
```

---

## Bifrost Storage

```
Bifrost/
  contracts/
    design-pipeline-contract.md    ← this file
  skills/
    design/
      _index.md
      design-curator/
        SKILL.md
      design-builder/
        SKILL.md
      design-validator/
        SKILL.md
      design-dialogue/
        SKILL.md
      design-harvester/
        SKILL.md
      design-distill/
        SKILL.md
```

---

## Phase Gates

### Phase 0 — Now (validated, ready to build)
- [ ] `design/curator` SKILL.md written
- [ ] `design/builder` SKILL.md written
- [ ] `design/validator` SKILL.md written
- [ ] First four Rune files written in Sanctum (Argent Décisif, Néon Humaniste, Signal Brut, Encre Éditoriale)
- [ ] `routing-guide.md` written in Sanctum
- [ ] Google design.md CLI integrated into design/validator

### Phase 1 — Post first real project run
- [ ] First run report produced and reviewed
- [ ] `design/dialogue` SKILL.md written
- [ ] Runestone v1 built and deployed (dogfood: Runestone is the first design pipeline project)
- [ ] Routing guide available at `/guide` in Runestone

### Phase 2 — Post first harvest
- [ ] First harvest report produced
- [ ] `design/harvester` SKILL.md written
- [ ] `design/distill` SKILL.md written
- [ ] First Rune version bump based on real run data
- [ ] Component builder with registry (for projects with existing components)

### Phase 3 — Optional (only if this becomes a client-facing product)
- [ ] Persona validator (simulated user task completion tests)
- [ ] Runestone v2 with full interactive motion previews
- [ ] System-spec self-generated using design pipeline

---

## Quality Rules

**An output is not this system if:**
- The Rune was not selected by a human before design/curator ran
- The Design Contract does not pass `npx @google/design.md lint`
- Any inviolable rule failed and was not explicitly noted in the run report
- design/builder used fonts, colours, or layouts not specified in the Design Contract
- The run report was not produced

**A Rune update is not valid if:**
- design/distill committed it directly (human must commit all Rune changes)
- The version bump level was lower than required by the version semantics
- Affected Design Contracts were not flagged for re-instancing on a major bump

**A Design Contract is not valid if:**
- It was generated from a Rune version that has since received a major bump without re-instancing
- The YAML frontmatter does not match the Rune's current colour tokens
- The inviolable rules section differs from the Rune's inviolable rules without explicit justification

---

## Relationship to the Broader System

This contract is the design domain instance of Bifrost's general domain-deepening pattern. Every technical domain in the system follows the same progression:

1. **Research** — Find who has already solved this rigorously. Learn from them.
2. **Model** — Identify the conceptual vocabulary. Name things.
3. **Validate** — Run real work through lightweight specs. Observe what breaks.
4. **Spec** — Write the contract from evidence, not theory.

The design pipeline was the first domain to complete this cycle. It produced: four validated Runes, three working packages tested against cold instances, and this contract. Other domains (frontend, backend, database, testing) follow when they qualify — when agent failures in that domain become specific, recurring, and consequential enough to warrant the investment.

The design pipeline is not a special case. It is the template.
