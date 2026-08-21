# HTML Rich Visual Architecture

**Status:** Implemented — First Increment  
**Designed:** 2026-08-21  
**Implemented:** 2026-08-21  
**Repository:** https://github.com/RPK-GIT/HTML-Renderer.git (commit `2f791d1`)  
**Tests:** 58 pass (39 original + 19 new in `tests/visual.test.js`)  
**Scope:** Optional HTML-specific visual specification layer for the HTML/SVG renderer

---

## 1. The Problem Being Solved

Experiment 2 proved that one canonical JSON produces valid, correct output from both renderers. However the static exports look broadly similar. The HTML renderer's real advantages — SVG interactivity, animations, hover, progressive disclosure, navigation — are barely visible in a PDF screenshot.

The question is: how can we unlock HTML-native capabilities without:

- Duplicating content
- Contaminating the canonical JSON with markup
- Breaking the PDF pipeline
- Creating a second source of truth for facts

---

## 2. Architecture Proposal

**Option B: Optional HTML visual specification embedded in the deck JSON.**

```
Canonical Deck JSON (Chapter_1_Slides_001_005.json)
│
│   "slides": [...content...]       ← both renderers read this
│   "_html_visual": {...hints...}   ← HTML renderer reads this; PDF ignores it
│
├─────────────────────┬───────────────────────
▼                     ▼
PDF Renderer     HTML Renderer
│                     │
▼                     ▼
Static PDF       Rich Interactive HTML
```

The visual specification lives under a single reserved key `_html_visual` at the **deck level**, outside the `slides` array. It is:

- Optional: absent = current HTML defaults apply, PDF unaffected
- Declarative: no HTML, CSS, SVG, or coordinates
- Additive: it describes HOW to render; the canonical slides describe WHAT to show
- Graceful: unknown modes fall back to current defaults without errors

---

## 3. Design Principles

**Principle 1 — Content and presentation remain separate.**  
The `slides` array is the single source of truth for content. The `_html_visual` block is pure presentation intent. It never redefines, summarises, or overrides content.

**Principle 2 — The visual spec enhances; it does not replace.**  
If `_html_visual` is absent, the HTML renderer produces exactly what it produces today. Adding `_html_visual` can only improve the output.

**Principle 3 — The PDF renderer is unaffected.**  
The PDF renderer already ignores unknown top-level keys. `_html_visual` never reaches it.

**Principle 4 — Modes are semantic, not mechanical.**  
`"mode": "sequential_reveal"` tells the renderer *what effect to achieve*. It does not say how many pixels to move, what CSS transition to use, or which SVG attributes to set.

**Principle 5 — Canonical `variant` fields take precedence.**  
If the canonical JSON says `"variant": "cycle"` on a process slide, the content is a cycle. The visual spec can say how to *animate* that cycle — not change it to something else.

---

## 4. The `_html_visual` Schema (Implemented)

```json
{
  "deck": { "title": "..." },
  "slides": [
    { "id": "my-slide-id", "type": "hierarchy", ... },
    { "type": "process", ... }
  ],
  "_html_visual": {
    "defaults": { ... },
    "by_type":  { ... },
    "by_slide": { ... }
  }
}
```

### 4.0 Slide IDs (new, implemented in Phase 1)

Any slide may carry an optional `id` field — a unique string key used to identify the slide in `by_slide` and navigation targets.

```json
{ "id": "chapter1-roadmap", "type": "hierarchy", ... }
```

- IDs must be unique when present. Duplicate IDs produce a validation **error**.
- Slides without an ID work exactly as before.
- `id` is ignored by the PDF renderer.

### 4.1 `defaults` — deck-wide visual behaviour

Reserved for future deck-wide settings. Currently accepted and merged as a baseline — no fields have renderer effect yet.

```json
"defaults": {
  "card_hover": true
}
```

### 4.2 `by_type` — visual mode for all slides of a given type

```json
"by_type": {
  "hierarchy": { "mode": "interactive_hierarchy" },
  "process":   { "mode": "sequential_reveal" }
}
```

Applies to every slide of that type unless overridden by `by_slide`. Unknown type keys produce a **warning**.

### 4.3 `by_slide` — override for a specific slide

Keys may be:
- A slide's `id` string (preferred — stable under reordering)
- A 1-based integer as a string: `"3"` means slide 3

**`by_slide` wins over `by_type` wins over `defaults`.** The merge is a simple object spread: `{ ...defaults, ...by_type_entry, ...by_slide_entry }`.

```json
"by_slide": {
  "chapter1-roadmap": {
    "mode": "interactive_hierarchy",
    "navigation": {
      "1 — Introduction": 3,
      "2 — Essential Concepts": "chapter1-concepts"
    }
  },
  "data-lifecycle": {
    "mode": "sequential_reveal"
  }
}
```

Unknown keys (neither a valid id nor a positive integer) produce a **warning**.

---

## 5. Visual Modes by Slide Type (Implementation Status)

**Implemented** modes are available in commit `2f791d1`. **Planned** modes are from the original design proposal and not yet implemented.

---

### 5.1 `hierarchy`

| Mode | Description | Status |
|---|---|---|
| `"interactive_hierarchy"` | Org-chart tree; CSS hover dims non-hovered branches; click a branch to focus it (dims others to 0.25); click again or click background to clear focus | **IMPLEMENTED** |
| `"radial_tree"` | Radial/circular layout — root at centre | **Planned** |
| `"collapsible_tree"` | Nodes can be clicked to collapse/expand children | **Planned** |

**Navigation (implemented):** A `navigation` map in the `by_slide` entry makes matching child/grandchild nodes clickable — clicking navigates to the target slide.

```json
"by_slide": {
  "my-roadmap-id": {
    "mode": "interactive_hierarchy",
    "navigation": {
      "Node Label": 3,
      "Another Label": "target-slide-id"
    }
  }
}
```

Navigation values: integer = 1-based slide index; string = slide `id` (resolved to index at render time). Unknown IDs produce a warning. The node label must match exactly (case-sensitive).

---

### 5.2 `process`

| Mode | Description | Status |
|---|---|---|
| `"sequential_reveal"` | Initial state: all steps visible at full opacity. First → or click focuses step 1 (others dim to 0.35). Each subsequent → advances focus. Visited steps remain at 0.65 opacity. After final step, next → advances the slide. | **IMPLEMENTED** |
| `"focused_step"` | One step enlarged; others dimmed | **Planned** |

The semantic content (step order, labels, detail text) is never changed. All steps remain visible at all times; only opacity changes.

```json
"by_slide": {
  "data-lifecycle-slide": {
    "mode": "sequential_reveal"
  }
}
```

**How sequential_reveal works technically:**
- Process SVG gains class `proc-seq` and each step card gains `data-step="N"` (1-based)
- The nav script installs a step controller on the slide
- Forward navigation (→ / click) is intercepted: if the controller has unfinished steps, it advances a step and returns early
- Going backward resets the step controller for the destination slide
- CSS: `svg.proc-seq.has-focus [data-step] { opacity: 0.35 }` / `.step-active { opacity: 1 }` / `.step-visited { opacity: 0.65 }`

---

### 5.3 `relationship`

| Mode | Description | Status |
|---|---|---|
| `"ellipse"` | Hub auto-detected, nodes on ellipse (current default) | **[existing]** |
| `"radial"` | Force-radial layout (`variant: "radial"` in canonical already triggers this) | **[existing]** |
| `"animated_edges"` | Edge lines draw in progressively on slide entry | **[new]** |
| `"force_directed"` | Physics simulation layout; nodes settle into position | **[new]** |

```json
"by_slide": {
  "8": {
    "mode": "animated_edges",
    "animation": {
      "edge_draw_ms": 400,
      "stagger_ms": 100
    },
    "interaction": {
      "hover": "highlight_connections"
    }
  }
}
```

**Interaction verbs for relationship:**

| Verb | Meaning |
|---|---|
| `"highlight_connections"` | Hover a node: highlight its edges and neighbour nodes; dim unconnected nodes | **[existing]** |
| `"expand_node"` | Click a node: show its `detail` text inline | **[new]** |
| `"navigate_slide"` | Click a node: jump to a designated slide | **[new]** |

---

### 5.4 `comparison`

| Mode | Description | Status |
|---|---|---|
| `"static"` | Both panels visible (current default) | **[existing]** |
| `"sequential_reveal"` | Left panel reveals first, right panel appears on advance | **[new]** |
| `"focus_side"` | Hover a panel to enlarge it; the other dims | **[new]** |

```json
"by_type": {
  "comparison": {
    "mode": "static",
    "interaction": {
      "hover": "focus_side"
    }
  }
}
```

---

### 5.5 `two_column` / `three_column`

| Mode | Description | Status |
|---|---|---|
| `"static"` | Cards fully visible (current default) | **[existing]** |
| `"expandable"` | Cards have `more` drawers (via canonical `more` field) | **[existing]** |
| `"staggered_reveal"` | Cards animate in with a staggered delay on slide entry | **[new]** |

```json
"by_type": {
  "two_column": {
    "mode": "staggered_reveal",
    "animation": {
      "entry_delay_ms": 150
    }
  }
}
```

---

### 5.6 `definition`

| Mode | Description | Status |
|---|---|---|
| `"static"` | Definition card centred (current default) | **[existing]** |
| `"term_highlight"` | Key terms in the definition text are underlined; hovering shows a brief tooltip | **[new]** |
| `"progressive_reveal"` | Definition card appears first; notes fade in after a delay | **[new]** |

```json
"by_type": {
  "definition": {
    "mode": "progressive_reveal",
    "animation": {
      "notes_delay_ms": 800
    }
  }
}
```

---

### 5.7 `section_summary`

| Mode | Description | Status |
|---|---|---|
| `"static"` | Numbered rows, clickable if `slide` field present | **[existing]** |
| `"staggered_reveal"` | Rows appear sequentially with animation | **[new]** |
| `"focus_current"` | As a learner navigates, the current section's row is highlighted | **[new]** |

---

### 5.8 `image_text`

| Mode | Description | Status |
|---|---|---|
| `"static"` | Image and text side by side (current default) | **[existing]** |
| `"zoomable_image"` | Clicking the image opens a lightbox/full-view | **[new]** |
| `"annotated"` | Hotspots on image reveal bullet points on hover | **[new]** |

---

### 5.9 Timeline (new type — currently absent)

A timeline is not currently in either renderer. If added, it would appear only in the canonical spec (new slide type), and the visual spec could enhance it:

```json
"by_type": {
  "timeline": {
    "mode": "animated_progression",
    "interaction": {
      "click": "expand_milestone"
    }
  }
}
```

---

## 6. Cross-Slide Navigation

This is a specific capability that the visual spec enables on any diagram node.

In the canonical JSON, a node or step has a label and optional detail. In the visual spec, it can be annotated with a navigation target:

```json
"by_slide": {
  "2": {
    "mode": "hover_tree",
    "navigation": {
      "1 — Introduction": 3,
      "2 — Essential Concepts": 7,
      "3 — DM Frameworks": 32
    }
  }
}
```

`"navigation"` is a map from node label (must match exactly) to slide number. The HTML renderer adds click handlers to matching nodes. The PDF renderer ignores the `navigation` map entirely.

This enables the hierarchy roadmap to become a clickable table of contents — without putting navigation intent into the canonical content.

---

## 7. Combination and Resolution

When the HTML renderer starts:

```
1. Load canonical JSON → parse slides array
2. Check for _html_visual key
3. If absent → use current renderer defaults (no change)
4. If present:
   a. Load "defaults" → merge into renderer context
   b. For each slide in the deck:
      i.  Check "by_slide" for this slide's 1-based index
      ii. If found → apply as visual context for this slide
      iii.If not found → check "by_type" for this slide's type
      iv. If found → apply as visual context for this slide
      v.  If neither → use renderer defaults for this type
5. Each slide template receives ctx.visual with resolved settings
6. Slide template checks ctx.visual.mode and renders accordingly
```

**Resolution priority (highest to lowest):**
1. `by_slide` entry for this slide index
2. `by_type` entry for this slide's type
3. Renderer defaults (current behaviour)

---

## 8. Fallback Model

| Scenario | Behaviour |
|---|---|
| `_html_visual` absent | Current renderer defaults apply exactly — no change |
| `by_type` references unknown slide type | Logged as a warning; ignored |
| `by_slide` references an out-of-range index | Logged as a warning; ignored |
| Unknown `mode` value for a type | Logged as a warning; renderer falls back to its default mode for that type |
| Unknown interaction verb | Logged as a warning; verb ignored; other verbs still applied |
| Unknown animation field | Logged as a warning; ignored |
| `navigation` maps to a non-existent slide | Logged as a warning; click handler not added for that node |

**The fallback rule:** any unrecognised instruction silently degrades to the default. The renderer never errors on a visual spec problem; it only errors on canonical content problems (missing required fields, etc.).

---

## 9. Validation

Visual spec validation is lighter than canonical spec validation:

- **Build-time:** Warn on unknown mode names. Warn on out-of-range `by_slide` indices. Never error.
- **No required fields:** Everything in `_html_visual` is optional.
- **No color policing:** Visual spec should not contain colors (colors are the theme's job). If colors appear, warn.
- **Report:** Visual spec warnings appear in `validation_report.json` as `source: "visual_spec"` warnings with severity `warning` (never `error`).

The visual spec validator runs only in the HTML renderer; the PDF renderer validator is never modified to inspect `_html_visual`.

---

## 10. Renderer Responsibilities

### PDF renderer
No change. Continues to ignore unknown top-level keys including `_html_visual`. No code change needed.

### HTML renderer

| Responsibility | Notes |
|---|---|
| Load and parse `_html_visual` if present | At deck load time, before slide rendering |
| Resolve visual context per slide | `by_slide` > `by_type` > defaults |
| Pass `ctx.visual` to each slide template | Templates check mode and apply appropriate SVG/CSS |
| Log visual spec warnings | Not errors; collected alongside content validation |
| Merge visual warnings into validation report | With `source: "visual_spec"` tag |
| Implement `mode` variants per type | In `src/svg/` modules and `src/slides/*.js` templates |
| Never modify content | The visual spec cannot change what `ctx.slide` contains |

---

## 11. Existing Components That Can Be Reused

| Component | Current capability | Extension path |
|---|---|---|
| `src/svg/hierarchy.js` | Org-chart tree + branch hover | Add `radial_tree` and `collapsible_tree` layout paths |
| `src/svg/process.js` | Horizontal / snake / cycle variants | Add JS-driven `sequential_reveal` step controller |
| `src/svg/relationship.js` | Ellipse/radial, edge hover | Add `animated_edges` draw-in and `force_directed` simulation |
| `src/slides/*.js` templates | Check `slide.variant` | Extend to also check `ctx.visual.mode` |
| `src/page.js` | Nav JS, overview grid | Add per-slide interaction controller hooks |
| `src/validation.js` | Error/warning collector | Add `source` tag; route visual spec issues to same reporter |
| `src/components.js` | `bulletList`, `bodyTop`, card rendering | No change needed |

---

## 12. Example: DAMA Slides 1–5 with Visual Spec

This is illustrative only. Nothing is being modified.

```json
{
  "deck": { "title": "DAMA-DMBOK Chapter 1 — Data Management Study Summary" },
  "slides": [
    ... (canonical slides 1–5 unchanged) ...
  ],
  "_html_visual": {
    "defaults": {
      "card_hover": true,
      "transition_ms": 250,
      "entry_animation": "fade_in"
    },
    "by_type": {
      "hierarchy": {
        "mode": "hover_tree",
        "interaction": {
          "hover": "highlight_branch"
        }
      },
      "comparison": {
        "mode": "static",
        "interaction": {
          "hover": "focus_side"
        }
      },
      "two_column": {
        "mode": "staggered_reveal",
        "animation": {
          "entry_delay_ms": 120
        }
      }
    },
    "by_slide": {
      "2": {
        "mode": "hover_tree",
        "interaction": {
          "hover": "highlight_branch",
          "click": "expand_collapse"
        },
        "animation": {
          "entry": "sequential_reveal",
          "delay_ms": 100
        },
        "navigation": {
          "1 — Introduction": 3
        }
      }
    }
  }
}
```

**What this produces:**
- Slide 1 (title): `entry_animation: fade_in` — subtle entry, same static output otherwise
- Slide 2 (hierarchy): branch hover-highlight + expand/collapse on click + animated entry + clicking "1 — Introduction" jumps to slide 3
- Slide 3 (two_column): staggered card reveal (left card, then right card, 120 ms offset)
- Slide 4 (two_column): same staggered reveal
- Slide 5 (comparison): hover dims the opposite panel (focus_side)

**What the PDF renderer sees:** The canonical `slides` array unchanged. `_html_visual` ignored.

---

## 13. What This Architecture Does NOT Do

- It does not duplicate the content (facts, definitions, relationships)
- It does not allow changing which nodes appear in a hierarchy
- It does not allow reordering steps in a process
- It does not allow adding new bullets to a slide
- It does not allow specifying font sizes, colors, or coordinates
- It does not require the PDF renderer to change

The visual spec is purely a rendering instruction. Content is immutable through it.

---

## 14. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Visual spec grows complex over time | Keep it shallow — modes, interactions, animations only; reject geometry/markup |
| `by_slide` indices break when slides are reordered | Add optional `id` to canonical slide spec; use `id` as the key instead of index |
| Teams add content to `_html_visual` ("just this once") | Strict schema validation at render time; content fields rejected with an error |
| Mode names proliferate inconsistently | Define a finite enum of verbs and modes in the canonical spec document |

**The index fragility problem (recommended fix):**  
Add an optional `id` field to canonical slides (a short kebab-case string, PDF renderer ignores it). Then `by_slide` keys become `"roadmap"` instead of `"2"`. This is backwards-compatible: existing specs without `id` continue to use index keys.

```json
{
  "type": "hierarchy",
  "id": "chapter-roadmap",
  "title": "Chapter Roadmap",
  ...
}
```

```json
"by_slide": {
  "chapter-roadmap": {
    "mode": "hover_tree",
    "navigation": { "1 — Introduction": 3 }
  }
}
```

---

## 15. Implementation Scope Estimate

**To implement a minimal useful version (hover + sequential_reveal + navigation):**

| Work item | Effort |
|---|---|
| Parse `_html_visual` in renderer startup | Small |
| Resolve visual context per slide, pass in `ctx.visual` | Small |
| Visual spec validation + warning reporting | Small |
| `sequential_reveal` on `process` slides | Medium |
| `expand_collapse` on `hierarchy` | Medium |
| `focus_side` on `comparison` | Small |
| `staggered_reveal` on column slides | Small |
| `navigate_slide` click handlers | Small |
| `animated_edges` on `relationship` | Medium |
| `radial_tree` for `hierarchy` | Large |
| `force_directed` layout | Large |

A useful first increment (all `Small` + `Medium` items) is achievable without large new SVG layout engines.

---

## 16. Recommendation

**Option B: Add an optional `_html_visual` specification to the canonical deck JSON.**

Reasons:
1. **One file, one source of truth.** Content and visual intent travel together. No sync problem.
2. **Zero PDF impact.** The PDF renderer ignores `_html_visual` today; no code change needed.
3. **Graceful degradation.** Absent visual spec = current output. Any unknown mode = current default. No new failure modes.
4. **Genuinely additive.** The architecture cannot degrade content quality; it can only enhance presentation.
5. **Small increment to start.** The first useful increment (hover, sequential reveal, navigation links) is a few days of work, not a redesign.
6. **Aligns with the canonical spec principle.** The visual spec is analogous to `_html_visual` the way CSS is analogous to HTML — a separate concern about presentation, not content.

**What to implement first:** The `navigation` map on `hierarchy` slides (clickable roadmap from a tree diagram) and `sequential_reveal` on `process` slides. These two additions cover the highest-value use cases in the DAMA deck and require no new layout engines.

**What NOT to implement:** `radial_tree`, `force_directed`, and `annotated` image modes. These require significant new SVG geometry work and add complexity before we have validated the overall approach.

---

## 17. Summary

| Question | Answer |
|---|---|
| Can HTML capabilities be added without breaking PDF? | Yes |
| Does it create a second source of truth? | No — content remains in `slides`; `_html_visual` is presentation-only |
| Does it require renderer code today? | No — this is a design proposal only |
| Recommended option | **B — optional `_html_visual` in deck JSON** |
| First useful increment | `navigation` on hierarchy + `sequential_reveal` on process |
| Index fragility fix | Add optional `id` field to canonical slides |
