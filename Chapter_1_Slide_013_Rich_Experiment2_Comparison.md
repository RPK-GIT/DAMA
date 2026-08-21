# Chapter 1 Slide 13 — Experiment 2 Comparison Report

**Date:** 2026-08-21  
**Slide:** "The 12 Data Management Principles" (Section 2.4, pp. 50–54)

## Three Versions Compared

| Version | File | Renderer | Visual type |
|---|---|---|---|
| A — Original PDF | `Chapter_1_Slides_011_015.pdf` p.3 | Deterministic PDF | Hierarchy tree (SVG) |
| B — HTML Experiment 1 | `Chapter_1_Slide_013_Rich.html` | HTML + `interactive_hierarchy` | Same tree with hover/click-to-focus |
| C — HTML Experiment 2 | `Chapter_1_Slide_013_Rich_Experiment2.html` | HTML + `principles_explorer` | Split-panel: group cards + source drill-down |

---

## 1. Visualization Concepts Considered

**Option A — Radial wheel / sunburst**  
Each group = one arc sector; principles = sub-arcs. Inspired by the DAMA Wheel visual. Rejected because: the 5 groups have unequal numbers of principles (2, 4, 3, 3, 1), making arc sizes misleading — a larger arc would imply more importance, not more principles. SVG arc geometry is also complex to implement at slide scale.

**Option B — Layered stack (architectural metaphor)**  
Groups stacked bottom-to-top suggesting a foundation-to-pinnacle progression. Rejected because: the source does not imply this ordering. Groups 1–4 are parallel thematic categories, not a dependency stack. This metaphor would misrepresent the source.

**Option C — Split-panel grouped card explorer (selected)**  
Left: 5 group cards always visible. Right: detail panel shows principles of the selected group, then source text on principle click. Chosen because: it preserves the 5-group organization prominently, makes all principles readable at a glance (as pill chips), and adds a genuine third level (source text) that the PDF cannot show.

---

## 2. Selected Visualization: Why

The **split-panel card explorer** was chosen for these reasons:

1. **Groups are made primary.** The PDF tree gives all 14 nodes roughly equal visual weight. The card layout gives the 5 groups clear prominence as large labelled cards — the learner sees the organizational structure immediately, not just a list of labels.

2. **Principles are secondary but always visible.** The pill chips under each group show all principles at a glance, without requiring hover or click to discover them.

3. **Source text is a genuine third level.** Clicking a principle shows the verbatim DAMA source text and page reference. The learner moves: Overview → Group → Principle → Source — a genuine progression that the PDF cannot support.

4. **No visual metaphor is imposed that isn't in the source.** The cards are a neutral organizational container, not an architectural metaphor, circular metaphor, or dependency stack.

5. **Architecturally clean.** Source text comes from `_html_visual.source_annotations` in the JSON — the renderer is generic, not DAMA-specific.

---

## 3. Interaction Model

**Default state:** Left panel shows 5 group cards with principle chips. Right panel shows instruction text.

**Click a group card:** 
- Clicked card gets a thicker blue left border (`selected` state)
- Other cards dim to 45% opacity
- Right panel shows the group name + numbered list of principle chips

**Click a principle chip (in the right panel):**
- Chip gets navy background (`selected` state)
- Source text box appears below the principle list
- Shows verbatim DAMA-DMBOK text + page reference in a navy panel

**Click same group again:** Clears selection; all cards return to full opacity; right panel returns to default.

**"← All groups" button:** Resets to default state.

**Keyboard navigation:** ← / → advance slides; O shows overview. No keyboard navigation within the explorer (mouse/touch only for the panel interaction — a known limitation for this experiment).

---

## 4. Source Drill-Down Implementation

Source text is stored in `_html_visual.by_slide["slide13-principles-exp2"].source_annotations`:
- Keys: principle labels (must match exactly)
- Values: `{ "text": "...", "page": "..." }`

All text is verbatim from `_ch1_extract.txt` (extracted from DAMA-DMBOK PDF pp. 50–54). No paraphrasing.

The renderer embeds annotations as a JSON data island (`<script type="application/json">`) which the page JS reads. No content is stored in the renderer code — the renderer is fully generic.

---

## 5. Source Pages Used

| Node | Source page(s) |
|---|---|
| Data is Valuable (group) | p. 50 |
| Unique asset properties | pp. 50–51 |
| Value expressible in economic terms | p. 51 |
| DM is a Business Requirement (group) | p. 50 |
| Managing data = managing quality | pp. 51–52 |
| Metadata is required to manage data | p. 51 |
| Planning is required to manage data | p. 53 |
| DM requirements must drive IT decisions | pp. 53–54 |
| DM Needs Diverse Skills (group) | p. 53 |
| Cross-functional by nature | p. 53 |
| Requires enterprise perspective | p. 53 |
| Accounts for multiple perspectives | p. 53 |
| DM is Lifecycle Management (group) | p. 53 |
| Data has a lifecycle to manage | p. 53 |
| Different data types, different requirements | p. 53 |
| Risk management is part of DM | pp. 53–54 |
| Leadership Commitment Required | p. 54 |

All page references come from the existing project analysis (`Chapter_1_Content_Analysis.md`) and source extraction (`_ch1_extract.txt`). No page numbers were guessed.

---

## 6. Output Files

| File | Description |
|---|---|
| `Chapter_1_Slide_013_Rich_Experiment2.json` | Canonical slide content + `_html_visual` overlay |
| `Chapter_1_Slide_013_Rich_Experiment2.html` | Interactive HTML presentation |
| `Chapter_1_Slide_013_Rich_Experiment2.pdf` | Static PDF export (default/uninteracted state) |

---

## 7. Validation Result

HTML renderer: **0 errors, 0 warnings**  
PDF export: **Success** (64 kB, 960 × 540 pt)  
Test suite: **61/61 pass** (3 new tests for `principles_explorer`)

---

## 8. Comparison with Experiment 1 (interactive_hierarchy)

| Dimension | Experiment 1 (hover tree) | Experiment 2 (explorer) |
|---|---|---|
| Visual structure | Same SVG tree as PDF | New HTML card layout |
| Groups visible | Always (as blue tree nodes) | Always (as prominent card headers) |
| Principles visible | Always (as light-blue leaf nodes) | Always (as small pill chips) |
| Hover behaviour | Branch dims; others dim to 35% | Card hover lifts (subtle) |
| Click behaviour | Branch focus locked; others dim to 25% | Group selected; principles listed on right |
| Source text | Not available | Verbatim text on principle click |
| "All groups" recovery | Click same branch or background | "← All groups" button |
| Static export | Tree (identical to PDF) | Card layout (genuinely different) |
| Distinctiveness from PDF | Low (same layout, added CSS effects) | High (completely different layout) |
| Educational progression | See all → focus one group | See all → select group → select principle → read source |

**Conclusion:** Experiment 2 is substantially more educationally rich than Experiment 1. Experiment 1 enhanced the PDF's static layout; Experiment 2 replaced it with a different interaction paradigm.

---

## 9. Comparison with the PDF

### Static readability

| Aspect | PDF | Exp 2 HTML (export/static) |
|---|---|---|
| All groups visible | Yes (as blue boxes) | Yes (as card headers) |
| All principles visible | Yes (as small nodes in tree) | Yes (as pill chips under each group) |
| Vertical hierarchy clear | Yes (connectors show parent-child) | Yes (chips are indented within cards) |
| Text size | Small (tree nodes at ~13pt) | Better — group headers at ~16px; chips at ~13px |
| Whitespace | Compact tree fills content zone | Groups are separated with clear padding |
| Leadership Commitment | Single box, equal visual weight to groups | Card at bottom, clearly different (no chips) |
| Page reference in footer | pp. 50–53 | pp. 50–54 |

The static export of Experiment 2 is slightly more readable than the PDF — clearer group separation, larger group headers, better whitespace. The right panel is empty in static export (it's interactive-only).

### Interactive value

| Capability | PDF | Exp 2 HTML (browser) |
|---|---|---|
| Focus one group | Mental isolation required | Click card → other cards dim |
| Read principles in detail | All visible but small | Right panel shows focused group's principles clearly |
| Access source text | Not available | Click principle → verbatim DAMA text + page number |
| Navigate Overview → Principle | Not supported | Supported: group cards → principle list → source panel |

**This is a qualitatively different learning experience from the PDF.** The source drill-down alone — moving from summary label to verbatim DAMA text with a single click — is a capability that fundamentally changes how a learner can use the slide.

---

## 10. Honest Assessment

### Where HTML Experiment 2 is genuinely better

1. **Source traceability** — the learner can verify every principle against the DAMA PDF without leaving the slide. This is the single most valuable feature.

2. **Group/principle hierarchy** — the card layout makes the 5 groups visually primary in a way the tree does not. A learner immediately understands that there are 5 thematic groups before reading the individual principles.

3. **Density management** — the explorer pattern allows a learner to focus on one group at a time without losing sight of the others. The PDF requires mental effort to isolate one branch.

4. **Learning progression** — the three levels (overview → group → source) map naturally to different stages of learning: first exposure, study, and verification.

### Where PDF is still competitive

1. **Scanning** — the PDF's tree is faster to scan for a specific label when you already know what you're looking for. The card layout requires clicking to drill into a group before seeing its principles clearly.

2. **Printing / sharing** — the PDF is universally shareable; the HTML requires a browser and loses the interactive features when exported.

3. **Connector visibility** — the tree's elbow connectors make parent-child relationships explicit graphically. In the card layout, the group-to-principle relationship is implied by card containment (which is equally clear, but different).

### Net verdict

**For self-study in a browser: HTML Experiment 2 is substantially better.** The source drill-down is not a cosmetic feature — it enables a learner to move from summary to primary source in one click, which supports deeper learning and fact-checking.

**For static reference / printing: PDF remains better.** The tree layout fits more naturally on a printed page; the explorer's right panel adds nothing in a static context.

---

## 11. Is This Worth Applying to Other Diagram-Heavy Slides?

**Yes — specifically for slides with multi-level structure where the user wants to trace content back to the source.**

Priority candidates in Chapter 1:

| Slide | Type | `principles_explorer` value | Reason |
|---|---|---|---|
| Slide 13 (this) | Hierarchy — 5 groups, 12 principles | **High** | ✓ Demonstrated |
| Slide 39 — Context Diagram 12 Components | Table | **Medium** | Each row could drill to its definition |
| Slide 44–45 — 11 Knowledge Areas | Table | **High** | Each KA definition could link to source pages 89–90 |
| Slide 15 — 13 Challenges overview | section_summary | **Medium** | Each challenge could link to its sub-section source |

The pattern generalises well to any slide where:
- Content is organized into named groups or items
- Each item has a corresponding source text passage
- The learner needs to verify or study individual items in depth

---

## New Renderer Capability

`principles_explorer` is now a generic renderer mode, not a DAMA-specific feature.

Any hierarchy slide can use it by adding:
```json
"_html_visual": {
  "by_slide": {
    "<slide-id>": {
      "mode": "principles_explorer",
      "source_annotations": {
        "<node label>": { "text": "<verbatim source>", "page": "<page ref>" }
      }
    }
  }
}
```

The annotations are optional per-node — if a label has no annotation, the panel shows a graceful fallback message.
