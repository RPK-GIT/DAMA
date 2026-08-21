# Canonical Slide JSON Specification

Version: 1.0  
Date: 2026-08-21

---

## 1. Architecture

```
               Canonical Slide JSON
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
      PDF Renderer          HTML Renderer
            │                     │
            ▼                     ▼
           PDF                 HTML/SVG
```

One JSON file describes WHAT a deck contains and HOW concepts relate to each
other semantically. Each renderer independently decides HOW to draw it — page
geometry, typography sizes, colors, SVG layout, HTML interactivity. Neither
renderer modifies the content it receives.

---

## 2. Design Principles

1. **Content, not coordinates.** The JSON must never contain pixel positions,
   CSS properties, SVG markup, or HTML elements. It describes semantic
   structure only.

2. **Renderer neutrality.** Every field must be meaningful regardless of which
   renderer processes it. Renderer-specific extensions are represented as
   optional semantic annotations that the other renderer ignores safely.

3. **Unknown fields are safe.** A renderer encountering an unrecognized field
   must silently ignore it and never error. This is how forward-compatible
   extensions work: an HTML-aware feature (e.g. `slide` navigation links) is
   present in the shared JSON but ignored by the PDF renderer.

4. **Theme is not the spec's job.** Individual slides must never hard-code
   colors, font names, or sizes. Those belong to the renderer's theme layer.
   The default palette (`#0E3A66` / `#2E75B6` / `#D9E8F5` / `#FFFFFF`) is a
   renderer default, not a slide property.

5. **Definitions are immutable.** Any `definition` field value must be passed
   through to the output verbatim — no paraphrasing, truncation, or
   reformatting. Both renderers enforce this contractually.

6. **One spec, both outputs.** Feeding the same JSON to both renderers must
   always produce valid, meaningful output. If a feature makes sense only in
   one renderer, represent it as an optional annotation the other ignores.

---

## 3. Top-Level Document Structure

```json
{
  "deck":   { "title": "Deck title shown in the footer" },
  "theme":  { "body_size": 13.0 },
  "slides": [ { "type": "...", ... } ]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `deck` | object | Yes | Deck-level metadata |
| `deck.title` | string | Yes | Shown in the footer of every slide |
| `theme` | object | No | Override renderer theme defaults; any Theme key |
| `slides` | array | Yes | Ordered list of slide objects |

### Common optional fields on every content slide

| Field | Type | Notes |
|---|---|---|
| `section` | string | Small label above the title (section or chapter context) |
| `subtitle` | string | Secondary line below the title |
| `intro` | string | Lead-in sentence rendered between the title and the main body |
| `source` | string | Rendered verbatim in the footer (e.g., "Source: p. 46") |

---

## 4. Slide Type Catalogue

### 4.1 `title`

**Purpose:** Opening slide for a deck or major section. Full-bleed background.

**Required:** `title`

**Optional:** `label`, `subtitle`, `author`, `date`

```json
{
  "type": "title",
  "label": "DAMA-DMBOK 2nd Edition",
  "title": "Data Management",
  "subtitle": "Chapter 1 — Principles, Frameworks, and Knowledge Areas",
  "author": "Study Summary",
  "date": "Source: pp. 45–95"
}
```

---

### 4.2 `section_summary`

**Purpose:** Numbered overview of topics, sections, or key points. Functions
as a clickable roadmap in HTML.

**Required:** `title`, `summary`

**`summary` items** — each item is either:

- A plain string: `"Plain item text"` — rendered as a numbered row
- An object: `{ "text": "...", "detail": "...", "slide": N }` — richer form;
  `detail` is a subtitle line; `slide` (integer) is the target slide number
  for navigation (HTML only — PDF ignores it)

**Limits:** ≤ 10 items. Items 7–10 trigger a two-column layout automatically.

```json
{
  "type": "section_summary",
  "section": "Chapter Overview",
  "title": "Chapter Roadmap",
  "summary": [
    { "text": "1 — Introduction", "detail": "Business Drivers · Goals", "slide": 3 },
    { "text": "2 — Essential Concepts", "detail": "Data · Principles · Challenges" },
    "3 — Frameworks"
  ],
  "source": "Source: pp. 45–95"
}
```

**Cross-renderer note:** The PDF renderer converts each item to a string via
`str(item)`, so it renders plain-string items correctly. It ignores `detail`
and `slide`. To retain readable text in both renderers, always populate `text`
as the primary label and treat `detail` and `slide` as additive annotations.

---

### 4.3 `content`

**Purpose:** General-purpose slide. Supports a body paragraph, a bullet list,
and/or a callout highlight box.

**Required:** `title`

**Optional (content, choose as needed):** `body` (string), `bullets` (array),
`callout` (string)

```json
{
  "type": "content",
  "section": "2.5 — Challenges",
  "title": "Data Quality",
  "body": "Poor data quality costs organisations 10–30% of revenue.",
  "bullets": [
    "Trust once lost is difficult to regain",
    "IBM estimated US cost of poor quality data at $3.1 trillion (2016)"
  ],
  "source": "Source: pp. 56–58"
}
```

---

### 4.4 `two_column`

**Purpose:** Two equal-width cards side by side. For contrasting perspectives,
before/after comparisons, or two related topics.

**Required:** `title`, `columns` (array of exactly 2 column objects)

Each column:

| Field | Type | Notes |
|---|---|---|
| `heading` | string | Card title |
| `body` | string | Paragraph text |
| `bullets` | array | Bullet list (strings or sub-bullet objects) |
| `more` | string | Expandable detail (HTML only — PDF ignores) |

Use `body` OR `bullets` per column (not both), unless intentionally combining.

```json
{
  "type": "two_column",
  "section": "1 — Introduction",
  "title": "The Case for Managing Data",
  "intro": "Deriving value from data does not happen by accident.",
  "columns": [
    {
      "heading": "What Data Represents",
      "bullets": ["Vital enterprise asset", "Currency, life blood, new oil"]
    },
    {
      "heading": "What Management Requires",
      "bullets": ["Intention, planning, coordination", "Technical and business skills"],
      "more": "Additional detail visible on click in interactive presentations."
    }
  ],
  "source": "Source: pp. 45–46"
}
```

---

### 4.5 `three_column`

**Purpose:** Three equal-width cards. For three parallel themes, dimensions,
or perspectives.

**Required:** `title`, `columns` (array of exactly 3 column objects)

Column fields are the same as `two_column`. Use smaller bullet text (renderers
reduce font size automatically for three-column layouts).

Recommended: ≤ 4 bullets per column; ≤ 3 lines per bullet.

```json
{
  "type": "three_column",
  "title": "Selected Principles in Depth",
  "columns": [
    { "heading": "Unique Properties", "bullets": ["Not consumed when used", "Simultaneous multi-user access"] },
    { "heading": "Metadata is Essential", "bullets": ["Data cannot be touched", "Requires definition and context"] },
    { "heading": "Lifecycle Management", "bullets": ["Data has a lifecycle", "Different types, different requirements"] }
  ],
  "source": "Source: pp. 50–54"
}
```

---

### 4.6 `definition`

**Purpose:** Present a formal definition with emphasis and visual prominence.
The definition text is contractually immutable — both renderers pass it through
exactly as provided.

**Required:** `term`, `definition`

**Optional:** `label` (default: "Definition"), `notes` (array of strings),
`title` (default: derived from `term`)

```json
{
  "type": "definition",
  "section": "2.3 — Data as an Organizational Asset",
  "label": "Core Definition",
  "term": "Asset",
  "definition": "An asset is an economic resource, that can be owned or controlled, and that holds or produces value. Assets can be converted to money.",
  "notes": [
    "Data is widely recognised as an enterprise asset",
    "Monetisation of data is increasingly common"
  ],
  "source": "Source: p. 49"
}
```

**Verbatim rule:** Never paraphrase, shorten, or reformat the `definition`
value. If a definition is too long for a slide, choose a different layout (e.g.
`two_column` with body text) rather than modifying the definition.

---

### 4.7 `process`

**Purpose:** Sequential flow of steps with optional detail text per step.
Used for procedures, cycles, and causal chains.

**Required:** `title`, `steps` (2–8 items)

Each step: `{ "label": "...", "detail": "..." }` — `detail` is optional.

**Optional:** `intro` (string), `variant`

`variant` controls the diagram layout:

| Value | Behaviour | PDF | HTML |
|---|---|---|---|
| `auto` (default) | ≤ 5 steps → horizontal; 6–8 → vertical/snake | ✓ | ✓ |
| `horizontal` | Force horizontal regardless of count | ignored | ✓ |
| `snake` | Horizontal then wraps with a U-turn | ignored | ✓ |
| `cycle` | Circular arrangement; optional `center` label | ignored | ✓ |

The PDF renderer always uses its own auto-layout algorithm and ignores
`variant`. Include `variant` for HTML when a specific layout is needed; the
PDF version will still render correctly.

```json
{
  "type": "process",
  "variant": "cycle",
  "section": "2.5.9 — The Data Lifecycle",
  "title": "Data Lifecycle Key Activities",
  "center": "Data",
  "steps": [
    { "label": "Plan" },
    { "label": "Design & Enable" },
    { "label": "Create / Obtain" },
    { "label": "Store / Maintain" },
    { "label": "Use" },
    { "label": "Enhance" }
  ],
  "source": "Source: p. 62"
}
```

---

### 4.8 `hierarchy`

**Purpose:** Org-chart / tree structure. Root → children → grandchildren.
Canonical representation for any parent-child or classification structure,
including chapter roadmaps.

**Required:** `title`, `root` (string), `children` (array)

Children: each item is a plain string OR `{ "label": "...", "children": [...] }`.
Grandchildren must be plain strings.

**Limits (cross-renderer safe):**
- ≤ 6 children
- ≤ 4 grandchildren per child *(HTML hard limit; PDF renders more but may overflow)*

```json
{
  "type": "hierarchy",
  "section": "Chapter Overview",
  "title": "Chapter Roadmap",
  "root": "Chapter 1 — Data Management",
  "children": [
    { "label": "1 — Introduction", "children": ["Business Drivers", "Goals"] },
    { "label": "2 — Essential Concepts", "children": ["Data & Information", "12 Principles", "13 Challenges", "Data Strategy"] },
    { "label": "3 — DM Frameworks", "children": ["SAM & AIM", "DAMA Wheel", "Aiken Pyramid", "Framework Evolved"] },
    "4 — DAMA and the DMBOK",
    "5 — Works Cited"
  ],
  "source": "Source: pp. 45–95"
}
```

**Note for roadmap slides:** Use `hierarchy` (not `section_summary`) as the
canonical type when the content is genuinely hierarchical. The HTML renderer
adds hover-highlighting and branch interactivity without needing a different
slide type.

---

### 4.9 `relationship`

**Purpose:** Concept map / network of related nodes and labelled edges. A node
connected to every other node is automatically placed in the hub (center)
position.

**Required:** `title`, `nodes` (2–8 items), `edges` (array)

Each node: `{ "id": "unique-string", "label": "Display label" }`

Each edge: `{ "from": "id", "to": "id", "label": "optional", "directed": true/false }`

**Optional:** `directed` (boolean, deck-wide default), `variant`

`variant` values:

| Value | Behaviour | PDF | HTML |
|---|---|---|---|
| *(absent)* | Auto hub-detection | ✓ | ✓ |
| `"radial"` | Force radial layout regardless of hub | ignored | ✓ |

```json
{
  "type": "relationship",
  "variant": "radial",
  "title": "How the Core Concepts Relate",
  "nodes": [
    { "id": "dq", "label": "Data Quality" },
    { "id": "meta", "label": "Metadata" },
    { "id": "gov", "label": "Data Governance" }
  ],
  "edges": [
    { "from": "meta", "to": "dq", "label": "enables" },
    { "from": "gov", "to": "meta", "label": "enforces" }
  ],
  "source": "Source: pp. 50–65"
}
```

---

### 4.10 `comparison`

**Purpose:** Two-panel side-by-side contrast with a VS divider. Use for
trade-offs, before/after, managed vs. unmanaged.

**Required:** `title`, `left`, `right`

Each side: `{ "heading": "...", "points": ["...", ...] }`

**Optional:** `divider` (string, replaces "VS" label)

```json
{
  "type": "comparison",
  "section": "1.1 — Business Drivers",
  "title": "Why Data Management Matters",
  "left": {
    "heading": "Effective Data Management",
    "points": [
      "Enables organisations to get value from their data assets",
      "Delivers reliable, high-quality data for better decisions"
    ]
  },
  "right": {
    "heading": "Failure to Manage Data",
    "points": [
      "Similar to failure to manage capital",
      "Results in waste and lost opportunity"
    ]
  },
  "source": "Source: p. 46"
}
```

---

### 4.11 `table`

**Purpose:** Structured reference data. Navy header band, zebra rows,
auto-proportional column widths.

**Required:** `title`, `columns` (2–6 strings), `rows` (array of arrays)

**Optional:** `align` (array of `"left"` / `"center"` / `"right"` per column),
`emphasize_first_column` (boolean)

```json
{
  "type": "table",
  "title": "Context Diagram Components",
  "columns": ["Component", "Description"],
  "align": ["left", "left"],
  "rows": [
    ["Definition", "Concisely defines the Knowledge Area"],
    ["Goals", "Purpose and guiding principles"],
    ["Activities", "Actions classified as Plan / Develop / Operate / Control"]
  ],
  "source": "Source: pp. 77–78"
}
```

---

### 4.12 `image_text`

**Purpose:** Source figure or diagram on one side with explanatory text on
the other.

**Required:** `title`, `image`

`image` object:

| Field | Type | Notes |
|---|---|---|
| `path` | string | Relative to the JSON file's directory |
| `fit` | string | `"contain"` (default), `"cover"`, `"center"` |
| `caption` | string | Small caption below the image |
| `alt` | string | Accessibility description (HTML only; PDF ignores) |

**Optional:** `image_side` (`"left"` default, or `"right"`), `body` (string),
`bullets` (array)

```json
{
  "type": "image_text",
  "section": "2.5.9 — The Data Lifecycle",
  "title": "Figure 2 — Data Lifecycle Key Activities",
  "image": {
    "path": "_fig_p62.png",
    "fit": "contain",
    "caption": "Figure 2, DAMA-DMBOK p. 62",
    "alt": "The data lifecycle: Plan, Design & Enable, Create/Obtain, Store/Maintain, Use, Enhance, Dispose"
  },
  "image_side": "left",
  "bullets": [
    "Creation and usage are the most critical points in the lifecycle",
    "Data Quality, Metadata Quality, and Data Security must be managed throughout",
    "Focus on critical data; minimise data ROT (Redundant, Obsolete, Trivial)"
  ],
  "source": "Source: pp. 60–63"
}
```

---

### 4.13 `takeaway`

**Purpose:** Closing or summary slide. A single prominent statement with
optional supporting points.

**Required:** `statement`

**Optional:** `title`, `points` (≤ 3 strings)

```json
{
  "type": "takeaway",
  "section": "Chapter 1 — Summary",
  "title": "Key Takeaway",
  "statement": "Data management is neither easy nor simple — but because few organisations do it well, it is a source of largely untapped opportunity.",
  "points": [
    "Managing data is managing an asset throughout its lifecycle",
    "Both technical and business skills are required",
    "Committed leadership is the enabling condition"
  ],
  "source": "Source: pp. 45–95"
}
```

---

## 5. Bullet Item Format

A bullet item is either:
- A plain string: `"Item text"`
- An object with sub-bullets: `{ "text": "Parent item", "sub": ["Child one", "Child two"] }`

Both renderers support sub-bullets one level deep.

---

## 6. Optional Semantic / Interactive Annotations

These fields are semantic hints that do not alter the content meaning. The
PDF renderer ignores fields it does not recognise. The HTML renderer uses
them to add navigational or interactive behaviour.

| Annotation | Where | Type | HTML behaviour | PDF behaviour |
|---|---|---|---|---|
| `slide` | `section_summary` item | integer | Makes the row a hyperlink to slide N | Ignored |
| `detail` | `section_summary` item | string | Rendered as a subtitle under the item text | Ignored |
| `variant` | `process`, `relationship` | string | Selects diagram layout (`cycle`, `snake`, `radial`, …) | Ignored |
| `center` | `process` (cycle variant) | string | Label placed at the centre of a cycle diagram | Ignored |
| `more` | column in `two_column`/`three_column` | string | Expandable drawer (click to reveal) | Ignored |
| `alt` | `image_text` image object | string | Accessibility alt text for the `<img>` element | Ignored |

**Rule:** Do not add fields that carry renderer-specific markup (CSS, SVG,
HTML tags, PDF coordinates). Semantic annotations only.

---

## 7. Renderer Responsibilities (shared)

Both renderers must:
- Accept any JSON conforming to this specification.
- Silently ignore unrecognised fields (forward-compatibility).
- Never modify `definition` text — pass it through verbatim.
- Enforce the four-color palette; reject any hard-coded hex color in the spec
  that is not in the current theme.
- Validate required fields and report errors with a 1-based slide number.
- Write `validation_report.json` and `validation_report.md` alongside every
  output.
- Produce output at exactly 16:9 aspect ratio.
- Never hard-code a specific color or size from an individual slide spec —
  all visual constants come from the theme.

---

## 8. PDF Renderer Responsibilities

In addition to shared responsibilities:

- Produce exactly 959.76 × 540 pt pages.
- Handle text overflow with wrap → shrink-to-fit → visible truncation with
  ellipsis; report overflow as a validation error.
- Register every drawn element's bounding box; check for out-of-canvas
  placement and severe overlap (> 15% of the smaller element's area).
- Use ReportLab for drawing; Helvetica for all text.

---

## 9. HTML Renderer Responsibilities

In addition to shared responsibilities:

- Produce a self-contained single HTML file (inline CSS, JS, image data URIs;
  no external network dependencies).
- Logical canvas: 1280 × 720 px (scaled to viewport via CSS `transform`).
- Use SVG for all diagrams (process, hierarchy, relationship).
- Support keyboard navigation (←/→, PageUp/Down, Home/End), click-to-advance,
  overview grid (O key), and URL hash deep-linking (`#N`).
- Implement hover-highlighting on hierarchy branches and relationship edges.
- Apply expandable `more` drawers on column slides when the field is present.
- Respect `slide` navigation links on `section_summary` items.
- Apply `variant` on `process` and `relationship` diagrams.
- Merge build-time and runtime (browser-measured) overflow errors into a
  single validation report when generating a PDF export.
- Embedded images: PNG, JPG, SVG only. Warn if > 3 MB.

---

## 10. Rules for AI Agents Generating Slide JSON

1. **Pick the right type.** Use the semantic guide:
   - Sequential steps or causal chains → `process`
   - Parent/child or classification → `hierarchy`
   - Networks of related concepts → `relationship`
   - Two opposing sides → `comparison`
   - Formal definition → `definition`
   - Numbered overview or roadmap → `section_summary`
   - Source figure with explanation → `image_text`
   - Closing insight → `takeaway`
   - Default for narrative or mixed content → `two_column` or `content`

2. **Content only.** Never emit pixel positions, sizes, colors, CSS, SVG, or
   HTML. Only semantic fields.

3. **Respect structural limits.**
   - `process`: 2–8 steps
   - `hierarchy`: ≤ 6 children; ≤ 4 grandchildren per child
   - `relationship`: 2–8 nodes
   - `two_column`/`three_column`: ≤ ~5 bullets per column
   - `section_summary`: ≤ 10 items
   - `table`: 2–6 columns; ≤ ~12 rows

4. **Keep text presentation-short.** Renderers shrink fonts within a readable
   range and flag overflow as an error. Prefer concise bullets over long
   sentences.

5. **Preserve definitions verbatim.** Never paraphrase a `definition` field
   value. Use a different layout if the text is long.

6. **Use one JSON for both renderers.** Do not create separate HTML and PDF
   versions of the same content. If HTML-specific behaviour is needed (cycle
   variant, `detail` subtitles, `slide` links), add the annotation to the
   shared JSON — the PDF renderer ignores it.

7. **Source references.** Always populate `source` with the exact page
   reference for study/reference material.

8. **After rendering:** read `validation_report.json`. If `status` is not
   `"ok"`, shorten or split the offending slides and re-render.

---

## 11. Rules Preventing Renderer-Specific JSON

The following patterns are **forbidden** in canonical slide JSON:

```
// FORBIDDEN — PDF coordinates
"x": 200, "y": 150, "width": 400

// FORBIDDEN — CSS
"style": "font-size: 14px; color: #red"

// FORBIDDEN — HTML markup
"body": "<b>Bold text</b> and <i>italic</i>"

// FORBIDDEN — SVG
"svg": "<circle cx='50' cy='50' r='40'/>"

// FORBIDDEN — off-palette colors
"color": "#FF0000"

// FORBIDDEN — renderer-brand fields
"reportlab_font": "Times-Roman"
"html_class": "my-card"
```

The following patterns are **allowed** as semantic annotations:

```json
"variant": "cycle"
"slide": 3
"detail": "Sub-topic text"
"more": "Expanded detail (HTML reveals; PDF ignores)"
"alt": "Accessibility description for image"
```

---

## 12. Known Schema Gaps

The following differences between the two renderers have been identified
through source inspection. None require spec changes; they are documented
here for renderer maintainers.

| Gap | PDF renderer | HTML renderer | Impact |
|---|---|---|---|
| `hierarchy` grandchild limit | No hard limit (space-constrained; may overflow silently) | 4 per child (hard error) | Keep ≤ 4 grandchildren per child for cross-renderer safety |
| `section_summary` max items | No hard limit (rows shrink) | 10 (hard error) | Keep ≤ 10 items |
| `process` `variant` | Auto-layout only (≤5 horizontal, 6–8 vertical) | `auto`/`horizontal`/`snake`/`cycle` | PDF will not cycle; for cycle diagrams, note that the PDF version renders as vertical |
| `relationship` `variant` | Hub auto-detected only | `radial` forces radial layout | PDF ignores `"variant": "radial"` |
| `two_column` `intro` field | Not rendered in `columns.py` (header only reads `subtitle`) | Rendered as a lead-in sentence above the cards | Add `intro` to the `two_column` template in the PDF renderer to close this gap |
| Column `more` field | Ignored | Expandable drawer | By design — PDF cannot be interactive |
| `alt` on image | Ignored | `<img alt="...">` | By design — semantic accessibility hint |

---

## 13. Example Complete Deck JSON

```json
{
  "deck": { "title": "DAMA-DMBOK Chapter 1 — Data Management Study Summary" },
  "slides": [
    {
      "type": "title",
      "label": "DAMA-DMBOK 2nd Edition",
      "title": "Data Management",
      "subtitle": "Chapter 1 — Principles, Frameworks, and Knowledge Areas",
      "author": "Study Summary",
      "date": "Source: pp. 45–95"
    },
    {
      "type": "hierarchy",
      "section": "Chapter Overview",
      "title": "Chapter Roadmap",
      "root": "Chapter 1 — Data Management",
      "children": [
        { "label": "1 — Introduction", "children": ["Business Drivers", "Goals"] },
        { "label": "2 — Essential Concepts", "children": ["Data & Information", "12 Principles", "13 Challenges", "Data Strategy"] },
        { "label": "3 — DM Frameworks", "children": ["SAM & AIM", "DAMA Wheel", "Aiken Pyramid", "Framework Evolved"] },
        "4 — DAMA and the DMBOK",
        "5 — Works Cited"
      ],
      "source": "DAMA-DMBOK PDF pp. 45–95"
    },
    {
      "type": "definition",
      "section": "1 — Introduction",
      "term": "Data Management",
      "definition": "Data Management is the development, execution, and supervision of plans, policies, programs, and practices that deliver, control, protect, and enhance the value of data and information assets throughout their lifecycles.",
      "source": "DAMA-DMBOK PDF p. 45"
    },
    {
      "type": "process",
      "variant": "cycle",
      "section": "2.5.9 — The Data Lifecycle",
      "title": "Data Lifecycle Key Activities",
      "center": "Data",
      "steps": [
        { "label": "Plan" },
        { "label": "Design & Enable" },
        { "label": "Create / Obtain" },
        { "label": "Store / Maintain" },
        { "label": "Use" },
        { "label": "Enhance" }
      ],
      "source": "DAMA-DMBOK PDF p. 62"
    },
    {
      "type": "takeaway",
      "title": "Key Takeaway",
      "statement": "Data management is neither easy nor simple — but because few organisations do it well, it is a source of largely untapped opportunity.",
      "points": [
        "Managing data is managing an asset throughout its lifecycle",
        "Committed leadership is the enabling condition"
      ],
      "source": "DAMA-DMBOK PDF pp. 45–95"
    }
  ]
}
```

---

## 14. Companion Renderers

| Renderer | Repository | Language | Output |
|---|---|---|---|
| Deterministic PDF Renderer | https://github.com/RPK-GIT/Deterministic-Renderer | Python / ReportLab | PDF (959.76 × 540 pt, 16:9) |
| HTML/SVG Renderer | https://github.com/RPK-GIT/HTML-Renderer | Node.js / SVG | HTML + optional PDF via Playwright |

Both renderers accept the same canonical JSON. Theme overrides use the
renderer's own key names (see each renderer's `theme.py` / `theme.js`).
