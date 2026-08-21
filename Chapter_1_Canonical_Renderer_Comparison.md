# Chapter 1 Slides 1–5 — Canonical Renderer Comparison (Experiment 2)

**Date:** 2026-08-21  
**Canonical JSON:** `Chapter_1_Slides_001_005.json` (one file, fed to both renderers unchanged)  
**Change from Experiment 1:** Slide 3 gained `intro` field (+1 line). Slide 2 kept as `hierarchy` in both renderers.  
**PDF output:** `Chapter_1_Slides_001_005_Canonical.pdf`  
**HTML output:** `Chapter_1_Slides_001_005_Canonical.html` + `Chapter_1_Slides_001_005_Canonical_HTML.pdf`  
**Both renderers:** 0 errors, 0 warnings on the same input.

---

## Slide-by-Slide Comparison

---

### Slide 1 — Title: Data Management (`type: title`)

**Canonical content:** Identical in both outputs — same label, title, subtitle, author, date.

| Dimension | PDF | HTML |
|---|---|---|
| Background | Full-bleed navy (#0E3A66) | Same |
| Title weight | Very bold, ~88pt equiv — dominant visual mass | Bold, ~64px — slightly lighter due to browser font rendering |
| Label | "DAMA-DMBOK 2ND EDITION" — standard caps, light weight | Same text, wider letter-spacing (tracked caps) — more refined |
| Accent bar | Short, narrow blue line centred under title | Same bar, proportionally wider |
| Title vertical position | Block starts at ~42% down — good breathing room above | Block starts at ~38% down — slightly higher, less top air |
| Footer | Deck title · Source · no page number (title slide) | Same |

**HTML advantages:** Refined label tracking; proportionally wider accent bar.  
**PDF advantages:** Heavier title weight gives more visual drama; slightly better vertical balance.  
**Overall:** Tie. Both are professional title slides. Visual differences are purely renderer-driven — font rendering weight and label tracking.  
**Interaction value:** None (title slides are static in both).

---

### Slide 2 — Chapter Roadmap (`type: hierarchy`)

**Canonical content:** Identical in both outputs — same root, same 5 children, same grandchildren.  
**This is the key test:** In Experiment 1, slide 2 used different types in each renderer. Here both render `hierarchy` from the same JSON.

| Dimension | PDF | HTML |
|---|---|---|
| Tree structure | Root → 5 children → grandchildren — correct | Same tree — root, 5 children, grandchildren — correct |
| Root node | Navy rectangle, white bold text | Same navy, white bold — slightly smaller node (proportional to lower canvas) |
| Child nodes | Blue rounded rectangles, white bold text | Same colour, slightly smaller rectangles |
| Grandchild nodes | Light-blue rounded cards, navy text, stacked vertically under parent | Same — but grandchild cards sit slightly closer to parent; more compact vertical spacing |
| Connector style | Straight L-shaped bus connectors (horizontal run, then vertical drops) | Same elbow style — connectors appear slightly lighter (1px vs 1.4px stroke) |
| Vertical centering | Tree fills the content zone with good vertical centering | Tree sits slightly higher — a gap appears at the bottom below the deepest grandchildren |
| Hover (browser only) | N/A | Hovering a branch highlights it — child + grandchildren visually emphasised |

**HTML advantages:** Branch hover-highlighting is a genuine study aid — a learner can isolate each thematic group. Connectors and nodes are rendered in SVG (vector, crisp at any scale).  
**PDF advantages:** Slightly better vertical centering of the full tree; stroke weight on connectors is marginally heavier and more readable in print.  
**Overall:** HTML is better for interactive/browser use (hover). PDF is marginally better for static print (centering, connector weight).  
**Interaction value:** High — hover-highlighting on a 5-branch, 14-node tree is genuinely useful for a learner reviewing chapter structure.

---

### Slide 3 — The Case for Managing Data (`type: two_column`)

**Canonical content:** Identical in both — same intro, same two columns, same 4 bullets each.  
**New in Experiment 2:** The `intro` field is now in the shared JSON.

| Dimension | PDF | HTML |
|---|---|---|
| Intro sentence | **Not rendered.** PDF `two_column` template reads `title`/`section`/`subtitle` only; `intro` is silently ignored. Cards fill the content zone. | **Rendered** as a grey framing sentence above the cards: "Deriving value from data does not happen by accident — it requires management and leadership." |
| Column cards | Cards fill ~55% of slide height; vertically centred with good balance | Cards start below the intro line; slightly less vertical breathing room; cards fill ~45% of height |
| Card vertical position | Cards centred in content zone — balanced | Cards anchored below intro — leaves larger gap at bottom |
| Bullet text | Fills cards well; no overflow | Same bullets; all fit within card height |
| Hover | None | Cards lift slightly on hover (browser only) |

**This slide reveals the only actual renderer incompatibility in the canonical JSON.**  
The `intro` field is specified in `CANONICAL_SLIDE_SPEC.md` as a common optional field on all content slides. The HTML renderer renders it; the PDF renderer's `two_column` template does not.

**HTML advantages:** Intro sentence rendered correctly; adds framing context before the columns.  
**PDF advantages:** Without the intro, the cards have more vertical space and are better centred — the PDF output looks more balanced on this slide.  
**Overall:** A draw — HTML is semantically richer; PDF is visually more balanced. The incompatibility is a known gap in the PDF renderer's `columns.py` template, documented in `CANONICAL_SLIDE_SPEC.md` section 12.  
**Interaction value:** Low — hover lift is subtle; the main difference is the intro sentence.

---

### Slide 4 — Core Definitions (`type: two_column` with `body`)

**Canonical content:** Identical in both — same two columns, same verbatim definition text.

| Dimension | PDF | HTML |
|---|---|---|
| Definition text | Verbatim, 12.5pt body, wraps over 4 lines per column | Verbatim, 17px body — same text, wraps over 4 lines per column |
| Card positioning | Cards vertically centred in the content zone — good balance, significant space below | Cards anchored at top of content zone — large empty lower half |
| Card height | Sized to content; approximately half the slide height | Sized to content; cards sit in the upper third |
| Heading style | Bold navy, 14pt | Bold navy, 18px |
| Body text legibility | Clear at 12.5pt; larger font would be better | Clear at 17px — larger body size is an advantage for readability |

**HTML advantages:** Larger body text (17px vs 12.5pt) makes the verbatim definitions more readable, especially for study use.  
**PDF advantages:** Cards are vertically centred — the overall composition is better balanced; the large empty lower half in HTML looks unfinished.  
**Overall:** PDF wins on layout balance. HTML wins on text readability. This is purely renderer-driven — the JSON is identical.  
**Interaction value:** None — definition cards are static in both renderers.

---

### Slide 5 — Why Data Management Matters (`type: comparison`)

**Canonical content:** Identical in both — same two panels, same 4 points each, same headings.

| Dimension | PDF | HTML |
|---|---|---|
| Panel headers | Full-width navy bands, white bold text, same text | Same |
| VS medallion | Navy-outlined circle, "VS" in navy text, centred between panels | Same — medallion appears slightly smaller (proportional to canvas) |
| Bullet text | 12.5pt, round disc bullets; all 4 points fit without overflow | 17px, smaller disc bullets; all 4 points fit |
| Panel vertical fill | Panels fill ~55% of content zone height — well proportioned | Panels fill ~40% of content zone — more compact, larger empty space below |
| Hover (browser) | None | Panels lift slightly on hover |

**HTML advantages:** Larger bullet text; hover lift on panels.  
**PDF advantages:** Panels fill more of the vertical space — proportions are better for a static document.  
**Overall:** Tie to slight PDF advantage on proportioning. Content is identical.  
**Interaction value:** Minimal — hover lift is cosmetic on a comparison slide.

---

## Summary Table

| Slide | Content identical? | PDF visual | HTML visual | HTML interaction value | Winner (static) | Winner (live browser) |
|---|---|---|---|---|---|---|
| 1 Title | Yes | Bold, dramatic | Refined label tracking | None | Tie | Tie |
| 2 Hierarchy | Yes | Good centering, heavier connectors | SVG tree, branch hover | **High** | PDF (marginally) | **HTML** |
| 3 Two-column | Functionally yes; `intro` renders in HTML only | No intro, cards well-centred | Intro visible, cards displaced | Low | Tie | HTML (intro adds context) |
| 4 Definitions | Yes | Cards centred, smaller text | Cards top-anchored, larger text | None | PDF (balance) | Tie |
| 5 Comparison | Yes | Panels fill 55% height | Panels fill 40% height | Minimal | PDF (proportioning) | Tie |

---

## Answers to the Six Questions

### 1. Does the same JSON work successfully in both renderers?

**Yes — with one known partial exception.**  
All 13 slide types in the canonical spec are accepted by both renderers. The JSON was fed to both renderers without modification, and both reported 0 errors and 0 warnings. The one exception: the `intro` field on `two_column` is rendered by the HTML renderer and silently ignored by the PDF renderer. This is a documented gap in `columns.py`, not a failure — the PDF renderer doesn't error, it just omits the field.

### 2. Did either renderer require changes to the canonical JSON?

**No.** The JSON was fed unchanged. No renderer-specific workarounds were needed. The only change to the JSON from the previous version was adding `intro` to Slide 3 — a content decision, not a renderer requirement.

### 3. Are the visual differences now genuinely renderer-driven?

**Yes.** With the same JSON feeding both renderers, all remaining visual differences — card vertical positioning, font weight, connector stroke, panel proportions, title placement — are entirely attributable to renderer-level decisions about typography sizes, layout algorithms, and CSS box model behaviour. No JSON content differences contaminate the comparison.

### 4. Is HTML substantially better for any of these five slides?

**Yes — Slide 2 (hierarchy roadmap) in live browser use.**  
Branch hover-highlighting on a 14-node tree with 5 thematic groups is a genuine interactive advantage for a learner. For the other four slides, HTML and PDF are approximately equal; neither is substantially better.

### 5. Should the full Chapter 1 deck remain primarily PDF?

**Yes.** For the following reasons:
- Slides 1–15 are already validated at 0 errors through the PDF pipeline.
- For text-heavy study slides (tables, 13-item challenge lists, definition cards, image+text figures), the PDF renderer's vertical centring and consistent proportions produce better-balanced static pages.
- The `intro` field incompatibility in PDF `two_column` is a minor cosmetic gap.
- HTML's primary advantage — diagram interactivity — becomes significant only in slides with SVG diagrams (hierarchy, process cycle, relationship maps) that appear from Slide 13 onwards.

### 6. Which slide types should potentially get an HTML companion later?

| Slide type | HTML advantage | Priority |
|---|---|---|
| `hierarchy` | Branch hover-highlighting; visually identical SVG tree | **High** — every hierarchy slide benefits |
| `process` (cycle variant) | Circular layout with curved arrows not available in PDF | **High** — the Data Lifecycle (Slide 24) specifically |
| `relationship` | Radial layout, curved edges, node hover | **High** — any concept map slide |
| `section_summary` | Clickable roadmap links | **Medium** — useful for navigation in a long deck |
| `two_column` / `three_column` | Hover lift; `intro` field rendered | **Low** — visual improvement is marginal |
| `definition`, `comparison`, `table`, `image_text`, `takeaway` | None meaningful beyond text size | **None** — PDF is equal or better |

---

## Overall Conclusion

Experiment 2 confirms that **the canonical architecture works.** One JSON feeds both renderers correctly. All visual differences in these five slides are renderer-driven, not content-driven.

The two renderers are essentially equivalent for static study slides. HTML earns its keep specifically on **interactive diagram slides** — hierarchy trees, process cycles, and concept maps. For the full Chapter 1 deck, the recommended approach is:

1. **Complete Slides 16–50 using the PDF renderer** — the pipeline is validated, the JSON is canonical, and the content-heavy slides in this range do not benefit from HTML interactivity.
2. **After the full deck is complete**, produce an HTML companion for the diagram-heavy slides: the 12-principles hierarchy (Slide 13), the Data Lifecycle process cycle (Slide 24), and any relationship/concept map slides.
3. **Fix the `intro` gap in PDF `columns.py`** at the next opportunity — a one-line change that would make the PDF and HTML outputs identical on Slide 3.
