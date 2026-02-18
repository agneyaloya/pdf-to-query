# Ground Truth Comparison: FEM LED — Page 2

**Ground truth:** `fem_led_opus_page2.md` (Claude Opus 4.6)
**Challenger:** `fem_led_landingAI.md` lines 124–302 (LandingAI, page 2 extracted)

Page 2 is the primary ordering/options page — the highest-value page for structured extraction. Differences here have direct impact on usability.

---

## Summary Verdict

| Dimension | Opus 4.6 | LandingAI |
|-----------|----------|-----------|
| Ordering tree structure | ✅ Single unified 9-column table | ❌ Split into 3 separate HTML tables |
| Options section structure | ✅ Separate `###` section per category | ❌ Mixed: some embedded as rows, some as separate tables |
| ‡ symbol rendering | ✅ Consistent `‡` throughout | ❌ Inconsistent: `‡`, `(red asterisk)`, `(visual of asterisk)`, `(red symbol)`, `(plus sign)` |
| Option code accuracy | ✅ Correct | ❌ 3 transcription errors (see below) |
| Heading misclassification | ✅ Plain paragraph | ❌ "Lead times will vary..." promoted to `##` heading |
| Noise / artifacts | ✅ Clean | ❌ HTML anchors + cell IDs throughout |
| Content completeness | ✅ Complete | ✅ Complete |

---

## Detailed Findings

### 1. Ordering Tree: Split vs. Unified Table

This is the most significant structural difference on the page. The PDF presents a single wide ordering tree spanning all parameters. Opus correctly represents this as **one 9-column markdown table**:

```markdown
| Series | Length | Nominal Lumens | Diffuser | Distribution | Voltage | Driver | Color temperature | CRI |
|---|---|---|---|---|---|---|---|---|
| FEM | L24 24" ‡ | 2000LM 2,000 lumens | IMAFL Acrylic, lineal ribbed frosted lens | MD Medium | MVOLT 120-277V | GZ10 0 - 10V dimming | 30K 3000K | 80CRI 80 CRI |
...
```

LandingAI splits it into **three separate HTML tables** by column group:
- Table 1: Series, Length, Nominal Lumens
- Table 2: Diffuser, Distribution, Voltage
- Table 3: Driver, Color Temperature, CRI

> **Impact:** Critical. A split ordering tree breaks the relationship between columns. A query like "what lumen options are available for the 48" fixture at MVOLT?" requires joining all three tables. An LLM receiving these as separate chunks would need to reason across them without any explicit join key.

---

### 2. ‡ Symbol Rendering

The ‡ symbol flags ordering restrictions throughout the page. Opus renders it consistently. LandingAI cannot decide what it is and uses five different representations:

| Location | Opus | LandingAI |
|----------|------|-----------|
| Ordering tree (L24, L48, L96 lengths) | `‡` | `(plus sign)`, `(plus symbol)`, `(red cross icon)` |
| Options table (most entries) | `‡` | `(visual of asterisk)`, `(red asterisk)` |
| Options table (nLight last 2 entries) | `‡` | `(plus sign)` |
| Cordset / Reloc® entries | `‡` | `‡` (correct here) |

> **Impact:** High. The ‡ flag is semantically meaningful — it tells specifiers that an option has ordering restrictions. Inconsistent rendering means a cleaning step must catch five different strings to normalise this to a single flag. More importantly, the symbol also appears embedded in cell text with no surrounding whitespace, making regex tricky.

---

### 3. Transcription Errors in Option Codes

Each model makes errors — neither is fully reliable for option codes:

| Field | Opus | LandingAI | Correct | Error |
|-------|------|-----------|---------|-------|
| Emergency option | `247OP` | `2470P` | `2470P` | Opus wrong — "O" misread as "0" |
| Mounting option | `QMB` | `OMB` | `QMB` | LandingAI wrong — "Q" misread as "O" |
| Mounting option | `WLFPMP4X` | `WLFMP4X` | `WLFPMP4X` | LandingAI wrong — "P" dropped |

> **Impact:** Critical. Option codes are exact-match lookup keys. Errors are silent — there is no marker indicating a value is wrong. Notably, both models make character-level OCR mistakes on codes, just on different entries. This means **neither output can be trusted for option codes without validation against a known code list** — a ground-truth lookup or cross-model consensus check is needed.

---

### 4. Options Section Structure

**Opus** separates each option category into its own `###` section with a dedicated two-column markdown table:

```markdown
### Emergency:
| Code | Description |
|---|---|
| BE6WCP | ... |

### Mounting:
| Code | Description |
|---|---|
| ANGBKT | ... |
```

**LandingAI** collapses Emergency, Mounting, and Other Options into a single HTML table, with category names embedded as plain rows:

```html
<tr><td>Options</td><td></td></tr>
<tr><td>Emergency: BE6WCP</td><td>6W internal cold battery...</td></tr>
...
<tr><td>Mounting:</td><td></td></tr>
<tr><td>ANGBKT</td><td>Angle bracket...</td></tr>
```

Additionally, Reloc® entries are embedded as rows inside the Cordsets table rather than as a separate section.

> **Impact:** High. Flattening category headers into table rows makes category-level filtering harder. A query for "mounting options" requires scanning for the row where the first cell equals "Mounting:" rather than matching a section heading.

---

### 5. Heading Misclassification

**Opus** correctly treats "Lead times will vary..." as a plain informational paragraph.

**LandingAI** promotes it to an `##` heading:
```markdown
## Lead times will vary depending on options selected. Consult with your sales representative.
```

> **Impact:** Low in isolation, but symptomatic of LandingAI over-applying heading detection to text that is visually prominent (bold/large) but semantically a note, not a section title.

---

### 6. Footer Fragmentation

**Opus** combines the page footer into a single clean line:
```
**INDUSTRIAL:** 1 Acuity Way, Decatur, GA 30035 Phone: 800-705-SERV (7378) www.lithonia.com © 2012-2025 Acuity Brands Lighting, Inc. All rights reserved. Rev. 12/18/25
```

**LandingAI** fragments it across four separate anchor-wrapped blocks: brand name, address+phone, copyright, revision date.

> **Impact:** Negligible for extraction. Footer content is typically excluded from chunks anyway.

---

## Conclusions

1. **The ordering tree split is the most serious issue** for this project. Reconstructing the logical table from three physical fragments is non-trivial and would require custom post-processing keyed on column-group proximity.

2. **Both models make option code errors** — Opus gets `247OP` wrong (correct: `2470P`), LandingAI gets `OMB` and `WLFMP4X` wrong (correct: `QMB`, `WLFPMP4X`). Errors are silent and character-level. Neither model can be trusted for option codes without cross-validation.

3. **The ‡ symbol inconsistency** is fixable with a normalisation regex but requires enumerating all five variants.

4. **Recommended pre-processing additions** (extending page 1 findings):
   - Detect and merge split ordering tree fragments by matching column group headers
   - Normalise all ‡ variants to a single token (e.g. `‡`)
   - Promote embedded category-header rows back to `###` section headings
   - Validate extracted option codes against a known code list where available
