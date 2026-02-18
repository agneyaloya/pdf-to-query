# Ground Truth Comparison: FEM LED — Page 1

**Ground truth:** `fem_led_opus.md` (Claude Opus 4.6, page 1 only)
**Challenger:** `fem_led_landingAI.md` (LandingAI, full 11-page document — page 1 extracted for comparison)

---

## Summary Verdict

| Dimension | Opus 4.6 | LandingAI |
|-----------|----------|-----------|
| Section header formatting | ✅ Bold (`**INTENDED USE**`, etc.) | ❌ Plain text |
| Inline table (Catalog/Notes/Type) | ✅ Markdown table | ❌ Unformatted plain text |
| Image handling | ✅ Rich alt-text descriptions | ⚠️ Brief `<:: ::>` custom tags |
| OCR accuracy | ✅ Correct | ❌ "SA+ Capable" (should be "A+") |
| Noise / artifacts | ✅ Clean | ❌ HTML `<a id='...'>` anchors throughout |
| Content completeness (page 1) | ✅ Complete | ✅ Complete |
| Text fidelity | ✅ High | ⚠️ Minor differences (see below) |
| Document coverage | ❌ Page 1 only | ✅ All 11 pages |

---

## Detailed Findings

### 1. Section Header Formatting

**Opus** correctly bolds all section headers using `**HEADER**` markdown:
```
**INTENDED USE** — A general purpose and energy-efficient...
**CONSTRUCTION** — One-piece 5VA fiberglass housing...
```

**LandingAI** extracts the same headers as plain text:
```
INTENDED USE — A general purpose and energy-efficient...
CONSTRUCTION — One-piece 5VA fiberglass housing...
```

> **Impact:** High. For downstream LLM extraction, bolded headers are a reliable signal for section boundaries. Losing this formatting degrades chunking and field extraction quality.

---

### 2. Inline Table (Right Column Header Box)

**Opus** correctly identifies the Catalog Number / Notes / Type fillable fields as a table:
```markdown
| Field | Value |
|---|---|
| Catalog Number | *(blank field)* |
| Notes | *(blank field)* |
| Type | *(blank field)* |
```

**LandingAI** renders them as unformatted adjacent text:
```
Catalog
Number

Notes

Type
```

> **Impact:** Medium. These fields are a UI affordance (not product data), so extraction impact is low — but it illustrates LandingAI's weaker handling of sparse/layout-heavy content.

---

### 3. OCR Error: A+ Capable Luminaire

**Opus:** `## ✦ Capable Luminaire` / `A+ capable luminaire`

**LandingAI:** `SA+ Capable Luminaire`

The "S" is a hallucination — likely caused by a decorative icon or badge preceding the "A+" text being misread as an "S".

> **Impact:** Medium. This is a branded certification program. An incorrect name would cause lookup failures if this value were used as a key.

---

### 4. Noise: HTML Anchor Tags

LandingAI wraps every content block in an HTML anchor:
```html
<a id='25bd4b29-bb3d-4da1-b3b7-38c2f96cad93'></a>
```

There are 20+ such tags on page 1 alone. These are LandingAI's internal element IDs, not part of the source document.

> **Impact:** Medium. Requires a stripping pass before ingestion. If fed raw into an LLM, these tokens add noise and cost without information value.

---

### 5. Image Handling

**Opus** produces rich, descriptive alt-text for each image:
```
![Deep Lens](Image: A photograph of the FEM LED fixture in the "Deep Lens"
configuration. It is a long, rectangular, low-profile industrial LED luminaire
with a raised/deeper translucent lens cover, shown from a three-quarter
perspective angle. The housing appears to be a light gray fiberglass material.)
```

**LandingAI** uses terse custom tags:
```
<::An image of a linear LED light fixture. Text "Deep Lens" points to the fixture's lens.: figure::>
```

> **Impact:** Low for structured extraction (images rarely carry spec data). High for semantic search — Opus image descriptions would embed meaningfully and could match queries like "low profile fixture with sensor".

---

### 6. Minor Text Differences

| Location | Opus (ground truth) | LandingAI |
|----------|---------------------|-----------|
| Intended Use | "Acrylic/Polycarbonate" | "Acrylic-Polycarbonate" (hyphen instead of slash) |
| Construction | Single paragraph | Split into two paragraphs at "Power connection..." |
| Intended Use | Clickable link text in **bold** | Plain text |
| Stray character | — | Stray `1` (page number) appears inline in the contaminants paragraph |

---

### 7. Design Select Block Ordering

**Opus** places the Design Select section at the bottom of the right column, after the A+ section — matching the visual reading order of the PDF layout.

**LandingAI** places it before the Catalog Number fields — reflecting a linearization order that doesn't match the visual layout.

> **Impact:** Low for this specific block, but suggests LandingAI may reorder content on more complex layouts (ordering tables, multi-column spec tables). Worth watching on pages 2–4.

---

## Conclusions for Pipeline Design

1. **LandingAI is better for coverage** — it processed all 11 pages including the dense ordering tables (pages 2–4) which contain the bulk of structured product data.

2. **Opus is better for page 1 quality** — richer image descriptions, correct formatting, no noise artifacts, accurate OCR.

3. **Recommended approach:** Use LandingAI for full-document ingestion, but apply a pre-processing pass to:
   - Strip `<a id='...'>` anchor tags
   - Convert `<:: ::>` image tags to standard markdown
   - Re-bold section headers (detectable by `ALL CAPS —` pattern)

4. **Key risk to monitor:** LandingAI content reordering on multi-column layout pages. Validate pages 2–5 against a manual read of the PDF before using ordering tables as ground truth.
