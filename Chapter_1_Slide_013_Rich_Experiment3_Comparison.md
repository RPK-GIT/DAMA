# Chapter 1 Slide 13 — Experiment 3 Comparison Report

**Date:** 2026-08-21  
**Slide:** "The 12 Data Management Principles" (Section 2.4, pp. 50–54)  
**Mode:** `annotated_hierarchy` — hierarchy tree with floating source-text popover  
**HTML output:** `Chapter_1_Slide_013_Rich_Experiment3.html`  
**Validation:** 0 errors, 0 warnings

---

## Three Versions Compared

| Version | Visual structure | Source text | Interaction |
|---|---|---|---|
| A — PDF Slide 13 | Static hierarchy tree (SVG) | Footer reference only | None |
| B — Experiment 1 (`interactive_hierarchy`) | Same tree | Footer reference only | Branch hover dim + click-to-focus |
| C — Experiment 3 (`annotated_hierarchy`) | Same tree | Floating popover (verbatim) | Hover dim + click node → popover |

---

## What was built

**The hierarchy tree is visually identical to the PDF.** The same SVG layout, the same colours (navy root, blue children, light-blue grandchildren), the same elbow connectors, the same font sizes. The `annotated_hierarchy` mode adds nothing to the tree geometry.

**What is added:**
- Every node (root + 5 children + 12 grandchildren = 18 nodes) that has a `source_annotation` entry gets a `cursor: pointer` affordance and a hover brightness effect.
- Clicking any annotated node opens a floating navy popover positioned beside the node.
- The popover shows: node label (uppercase, light-blue), verbatim DAMA-DMBOK source text (italic, white), page reference (light-blue footer).
- A × close button dismisses the popover. Clicking elsewhere also dismisses it.
- Changing slides also dismisses the popover.

**What the tree still does:**
- CSS hover still dims non-hovered branches (35% opacity), unchanged from Experiment 1.
- The branch click-to-focus was deliberately disabled for annotated slides — clicking a node now opens the source popover instead of locking branch focus (the two interactions were mutually incompatible since the focus handler called `stopPropagation`).

---

## Interaction verification

| Action | Result |
|---|---|
| Hover "DM is a Business Requirement" branch | Other 4 branches dim to 35% — CSS hover intact ✓ |
| Click "Data is Valuable" (group node) | Popover appears: "DATA IS VALUABLE" / verbatim source / p. 50 ✓ |
| Click "Unique asset properties" (grandchild) | Popover: "UNIQUE ASSET PROPERTIES" / principle text / p. 50–51 ✓ |
| Click "Leadership Commitment Required" (standalone) | Popover: full principle text / p. 54 ✓ |
| Click "12 Data Management Principles" (root) | Popover: overview text / p. 50 ✓ |
| Click × close | Popover dismisses ✓ |
| Click-outside (on slide background) | Popover dismisses ✓ |
| Arrow key navigation | Popover dismisses on slide change ✓ |
| Viewport resize | Slide scales proportionally; popover repositions on next click ✓ |

---

## Visual Assessment

### Default state (no interaction)

**The hierarchy looks better than the PDF** in one specific way: the slide is rendered at a native 1280×720 browser resolution rather than 959.76 pt PDF resolution, so the SVG curves and text are marginally crisper. The layout, proportions, and colour are otherwise identical. A side-by-side comparison would require careful measurement to spot the difference.

### Popover design

- **Navy background with light-blue label**: matches the design system without introducing off-palette colours
- **Italic body text** in quotation marks: visually distinguishes source text from paraphrase
- **Page reference** in light-blue at the bottom: small and unobtrusive, directly citable
- **Positioning**: popover appears to the right of the clicked node when space allows, clamped to the viewport otherwise. The Leadership Commitment node (rightmost) correctly positions the popover to the left.
- **Size**: 340px wide — substantial enough to read, compact enough not to obscure the full tree

The popover is **professional and non-distracting**. It appears and disappears cleanly. There is no animation (by design — the text appears immediately). A 120–160ms fade-in could be added in future if desired.

---

## Comparison: PDF vs Experiment 1 vs Experiment 3

### Static readability

| Dimension | PDF | Exp 1 | Exp 3 |
|---|---|---|---|
| Tree layout | ✓ | ✓ (identical) | ✓ (identical) |
| All nodes visible | ✓ | ✓ | ✓ |
| Text legibility | Good | Good | Good (slightly crisper on screen) |
| Source reference | Footer only | Footer only | Popover on click |

### Interaction quality

| Dimension | PDF | Exp 1 | Exp 3 |
|---|---|---|---|
| Hover branch focus | None | CSS dim (35%) | CSS dim (35%) |
| Click branch focus | None | Lock focus (25%) | Removed — popover takes precedence |
| Source text access | None | None | Click any node |
| Source text fidelity | N/A | N/A | Verbatim from DAMA-DMBOK |
| Page reference | N/A | N/A | Per node, pp. 50–54 |
| Dismiss mechanism | N/A | Click again | × button or click-outside |

### Learning value

| Scenario | PDF | Exp 1 | Exp 3 |
|---|---|---|---|
| First exposure — see the full structure | ✓ | ✓ | ✓ (identical) |
| Study one principle group | Mental isolation | Visual isolation (dim) | Visual isolation (dim) |
| Verify against source | Open PDF, find page | Not available | Click node — one step |
| Read verbatim wording | Open PDF | Not available | Popover |
| Trace to exact page | Open PDF | Not available | Popover shows page |

---

## Honest Assessment

**Is Experiment 3 better than Experiment 1?** Yes, substantially.

Experiment 1 was a valid first step — it made branch focus interactive — but it did not add source traceability. Experiment 3 adds the genuinely new capability: clicking a node shows you what the DAMA-DMBOK says about that principle, verbatim, in one action. This changes the slide from a summary into a navigable reference.

**Is Experiment 3 better than the PDF?** For self-study in a browser: yes.

The single most valuable feature is the popover. A learner studying "Metadata is required to manage data" can click the node and immediately read: *"Managing any asset requires having data about that asset... The data used to manage and use data is called Metadata. Because data cannot be held or touched, to understand what it is and how to use it requires definition and knowledge in the form of Metadata."* — with the page reference (p. 51) visible. Without the HTML slide, reaching the same information requires switching to the PDF, navigating to page 51, and finding the passage.

**What was sacrificed vs Experiment 1?** The click-to-focus-branch behaviour (locking one branch at 25% dimming) was removed because it conflicted with the ann-node click handler. This is a reasonable trade-off — the source popover is the more educationally valuable feature. The CSS hover dimming (35%) is preserved and works correctly.

**Limitations:**
1. The popover covers nearby nodes when positioned over the centre of the tree. On a 5-column tree, the middle column positions overlap with adjacent columns. This is inherent to floating popovers on a dense diagram — acceptable for study use.
2. The popover has no keyboard access — Tab/Enter do not open it. Keyboard users must use a mouse/touch. A future improvement would add `tabindex` to ann-nodes.
3. Closing the popover by clicking the tree background area (not a node) requires clicking on a specific white area — the SVG fill is transparent, so clicking on the SVG outside a node hits the slide background and correctly dismisses.

---

## Bug fixed during this experiment

**Root cause of the popover not triggering:** The existing `interactive_hierarchy` JS attached a `click` listener to every `.h-branch` group that called `e.stopPropagation()`. Since `g.ann-node` is nested inside `g.h-branch`, the `stopPropagation` prevented the document-level ann-node click listener from ever firing.

**Fix:** One-line guard in `src/page.js` — the h-branch focus loop now checks `if (svg.hasAttribute('data-annotated')) return;` and skips annotated SVGs entirely.

**Lesson:** When two interaction systems (branch focus + popover) share the same event target tree, the outer system's `stopPropagation` silently breaks the inner system. The fix is correct and minimal — it preserves branch focus behaviour for `interactive_hierarchy` slides and disables it for `annotated_hierarchy` slides.

---

## Source annotations: accuracy

All 18 annotated nodes use text directly extracted from `_ch1_extract.txt` (PDF pages 50–54). No paraphrasing. Page references are from `Chapter_1_Content_Analysis.md`. None were guessed or invented.

---

## Should this approach be applied to other slides?

**Yes — specifically to any hierarchy or process slide where the learner needs to trace a label back to its source definition.**

Best candidates in Chapter 1:

| Slide | Type | Nodes to annotate | Value |
|---|---|---|---|
| Slide 13 (this) | Hierarchy | 18 (all) | **High** — demonstrated |
| Slide 44–45 — 11 Knowledge Areas | Table | 11 rows | **High** — each KA has a definition on pp. 89–90 |
| Slide 15 — 13 Challenges | section_summary | 13 rows | **Medium** — each challenge links to a sub-section |
| Slide 9 — Why representation is hard | Process | 4 steps | **Medium** — each step has a source sentence |
| Slide 2 — Chapter Roadmap | Hierarchy | 5 groups | **Low** — group descriptions are summaries, not definitions |

The `annotated_hierarchy` mode generalises to `annotated_process` or `annotated_table` with minor renderer additions — the popover JS and CSS are already generic.

---

## Files produced

| File | Description |
|---|---|
| `Chapter_1_Slide_013_Rich_Experiment3.json` | Canonical hierarchy + `_html_visual` with 18 source annotations |
| `Chapter_1_Slide_013_Rich_Experiment3.html` | Interactive HTML — 0 errors, 0 warnings |
| `Chapter_1_Slide_013_Rich_Experiment3_Comparison.md` | This report |
| `_s13_exp3_final_default.png` | Screenshot: default state |
| `_s13_exp3_popover_working.png` | Screenshot: group popover (Data is Valuable) |
| `_s13_exp3_popover_principle_working.png` | Screenshot: principle popover (Unique asset properties) |
| `_s13_exp3_final_leadership.png` | Screenshot: Leadership Commitment Required popover |

**HTML-Renderer commits:**
- `10fe0e4` — `annotated_hierarchy` mode + data island + ann-node wrapping + popover CSS/JS
- `f353b32` — bugfix: skip h-branch stopPropagation on annotated SVGs
