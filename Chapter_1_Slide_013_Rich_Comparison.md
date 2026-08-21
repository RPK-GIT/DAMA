# Chapter 1 Slide 13 — PDF vs HTML-Rich Comparison

**Date:** 2026-08-21  
**Slide:** 13 — "The 12 Data Management Principles" (`type: hierarchy`)  
**Source material:** DAMA-DMBOK 2nd Edition, pp. 50–53

## Files

| Version | File | Renderer |
|---|---|---|
| A — Original PDF | `Chapter_1_Slides_011_015.pdf` (page 3) | Deterministic PDF Renderer |
| B — HTML-rich | `Chapter_1_Slide_013_Rich.html` + `Chapter_1_Slide_013_Rich.pdf` | HTML/SVG Renderer + `_html_visual` |

## Content Fidelity

**Identical.** The slide JSON is character-for-character the same as the canonical Slide 13 in `Chapter_1_Slides_011_015.json`. No content was added, removed, paraphrased, or reorganised. The only additions are:
- An `id` field: `"slide13-principles"` (ignored by PDF renderer)
- A `_html_visual` block specifying `"mode": "interactive_hierarchy"` (ignored by PDF renderer)

---

## Side-by-Side Assessment

### 1. Static Readability

| Aspect | PDF | HTML (export) |
|---|---|---|
| Title and section label | Clear, navy bold, 26pt | Clear, navy bold, 32px |
| Root node | Navy rectangle, "12 Data Management Principles" centred | Same — slightly smaller due to lower canvas resolution |
| Five principle groups | Blue rectangles, white bold text, readable | Same — branches slightly more compact vertically |
| Grandchild nodes | Light-blue cards, navy text, readable | Same |
| Connectors | 1.4pt elbow connectors — slightly heavier | 1px SVG connectors — marginally thinner |
| Vertical balance | Tree centred well within content zone | Tree sits slightly higher; modest gap at bottom |
| Footer | Source reference + page counter | Same |
| Page dimensions | 959.76 × 540 pt | 960 × 540 pt |

**Verdict:** Static readability is essentially equal. The PDF has a marginally better vertical balance (tree more centred). The HTML has a marginally larger title text. Neither advantage is significant.

---

### 2. Hierarchy Clarity

Both versions convey the same tree structure: 1 root → 5 children → 2–4 grandchildren per child. All 14 nodes are visible, labelled, and readable in both renderers.

The PDF version's connector stroke is slightly heavier (1.4pt), which may make the parent-child relationships marginally more legible at small sizes. The HTML version's SVG connectors are 1px but render crisply on screen.

**Verdict:** Equal.

---

### 3. Visual Attractiveness

Both versions use the identical palette (#0E3A66 navy, #2E75B6 blue, #D9E8F5 light-blue, #FFFFFF white) and the same colour-per-level coding. Corner radii are very close. Card shadows are absent in PDF; the HTML renderer adds a subtle box-shadow on hover via the `.hoverable` CSS class — this is not visible in the export but present in the live browser.

**Verdict:** Equal in static export. HTML is marginally more polished in a browser (card shadows, smooth hover transitions).

---

### 4. Learning Value

This is where the comparison diverges.

**PDF (static):**  
The learner sees all 5 principle groups at once. This is good for overview comprehension — you can read the entire structure on one page. For a first read of a 14-node tree, seeing everything simultaneously is appropriate.

The limitation: with 14 nodes on screen simultaneously, the eye has no natural focus point after the root. A learner studying "DM is a Business Requirement" has to visually filter out the 4 other groups and their 10+ grandchildren.

**HTML (live browser):**  
When the learner hovers over "DM is a Business Requirement":
- That branch stays at full opacity
- The other 4 groups dim to 35% opacity (via CSS `svg.hier:hover .h-branch:not(:hover) { opacity: 0.35 }`)
- The visual attention narrows naturally to the group being studied

When the learner clicks "DM is a Business Requirement":
- The focus locks (JS click-to-focus: other branches dim to 25%)
- The learner can read all 4 grandchild principles at full attention without visual noise
- Clicking again (or clicking the SVG background) releases the focus

This is a meaningful difference for a self-study learner working through 5 thematic groups. The hover/focus effectively turns a 14-node overview into an on-demand single-group study view, without hiding any content permanently.

**Verdict: HTML is genuinely better for learning** — specifically for the study use case where a learner wants to concentrate on one principle group at a time.

---

### 5. Interactivity

**PDF:** None. All 14 nodes are static. The learner must mentally isolate one branch while reading others.

**HTML (browser):**

| Interaction | Behaviour |
|---|---|
| Hover a branch group | Non-hovered branches dim to 35% — natural, smooth, 180ms transition |
| Click a branch group | Clicked branch stays at 100%; others dim to 25% — stronger focus lock |
| Click same branch again | Focus released; all branches return to 100% |
| Click SVG background | Focus released |
| Keyboard navigation | ← / → navigate across slides; O shows overview grid; no slide-internal keyboard interactions for hierarchy |
| Overview grid (O key) | Single-slide deck shows the one slide — less useful here but works correctly |
| Resize/scale | Canvas scales proportionally to any viewport via CSS transform; SVG is resolution-independent — crisp at any size |

**Verdict: HTML adds meaningful interactivity.** The hover/click-to-focus is not a gimmick — it serves a genuine study purpose for a dense 14-node diagram.

---

### 6. Information Density

Both versions contain exactly the same 14 nodes. Neither hides or reveals additional content. Information density is identical.

The key difference is that HTML allows the learner to *manage* perceived density on demand — hovering reduces visual complexity from 14 active nodes to the hovered branch's 3–5 nodes. The PDF always presents full density.

**Verdict: Advantage HTML for learner-managed density.**

---

### 7. Does Interaction Genuinely Help Understanding?

Yes — specifically for this slide.

The 12 Principles hierarchy is one of the most information-dense slides in Chapter 1. It has:
- 5 principle groups
- 12 individual principles
- 1 additional "Leadership Commitment" node
- 14 nodes total in a tree that spans the full slide width

For a learner who already knows the 5 groups exist and wants to study one at a time, the hover/click behaviour provides genuine focus control. For a learner seeing the slide for the first time, the full static view is a good overview, after which the interactivity helps consolidation.

The interaction does not add noise: the dimming is subtle, the transitions are smooth, and no content is ever hidden — which means the PDF export (which captures the un-hovered state) still communicates the full structure.

**Verdict: Yes, interaction helps.** It turns a reference slide into a study tool for learners who want to drill into one principle group.

---

### 8. Is HTML Substantially Better Than PDF for This Slide?

**For static reading / printing / sharing:** No — equivalent quality.

**For active self-study in a browser:** Yes — meaningfully better. The hover/click-to-focus maps directly onto the natural human behaviour of "I want to study just this one group." The PDF requires the learner to perform this mentally; the HTML performs it automatically.

The advantage is proportional to slide density. Slide 13 is the densest hierarchy in Chapter 1 (14 nodes). Sparser hierarchy slides (e.g., the Chapter Roadmap with 5 children and no grandchildren for sections 4 and 5) would benefit less from the interaction.

---

## Summary Table

| Dimension | PDF | HTML | Winner |
|---|---|---|---|
| Static readability | Excellent | Excellent | Tie |
| Hierarchy clarity | Very good | Very good | Tie (PDF: slightly heavier connectors) |
| Visual attractiveness (static) | Professional | Professional | Tie |
| Visual attractiveness (browser) | N/A | Hover lift + shadows | HTML |
| Learning value (first read) | Good | Good | Tie |
| Learning value (study/drill) | Good | **Very good** | **HTML** |
| Interactivity | None | Hover + click focus | **HTML** |
| Density management | Fixed (full) | Learner-controlled | **HTML** |
| Information fidelity | Complete | Complete | Tie |
| Print quality | Excellent | Good (export) | PDF |
| File portability | Universal PDF | Self-contained HTML | Tie |

---

## Recommendation

**For this specific slide type (dense hierarchy, many nodes):** produce both versions.

- **PDF version:** stays as canonical deliverable for printing, sharing, offline reference.
- **HTML-rich version:** companion study file for browser-based learners. No content changes needed — just add `id` and `_html_visual` to the deck JSON.

The overhead of producing the HTML companion for hierarchy slides is extremely low: a 3-line `_html_visual` block and one render command. The learning benefit for dense diagrams is real and immediate.

**For sparser hierarchy slides (fewer than 6 total nodes):** the interaction adds less value. Reserve the HTML companion for slides where visual complexity warrants managed focus.

**For the full Chapter 1 deck:** the following slides would benefit most from an HTML-rich companion:
1. Slide 13 — 12 Principles hierarchy (this experiment) ← demonstrated
2. Slide 2 — Chapter Roadmap hierarchy (already partially demonstrated in Experiment 2)
3. Any relationship/concept map slides (once added to the deck)
4. Slide 24 — Data Lifecycle (process cycle variant, sequential reveal would add educational value)
