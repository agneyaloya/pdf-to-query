# PDF-to-Query Pipeline Plan

## Overview

Process product spec sheet PDFs (LED lighting + plumbing fixtures) into a searchable,
structured catalog with natural language querying and an interview-facing pipeline
evolution log.

---

## Pipeline Architecture

```
PDFs (spec sheets)
  └─▶ Step 1: Ingestion & OCR
        pdfplumber for text + table extraction (tables → markdown)
        pytesseract fallback for image-only pages
        → per-page output: {text_blocks, tables, ocr_used}

  └─▶ Step 2: Chunking
        tables treated as atomic chunks (never split mid-table)
        text split by section/heading
        → chunks with metadata: {source, page, section, category, chunk_type: text|table}

  └─▶ Step 3: Embeddings
        sentence-transformers (local) or OpenAI embeddings
        → vectors per chunk

  └─▶ Step 4: Vector DB
        pgvector extension on existing Postgres (no extra service)
        → stored chunks + vectors + metadata

  └─▶ Step 5: Query Retrieval
        embed user query → cosine similarity search → top-k chunks

  └─▶ Step 6: LLM Structured Extraction
        Claude API: extract {name, model, specs, category} from chunks
        → structured Product records saved to Postgres

  └─▶ Step 7: Evaluation & Iteration
        per-run metrics logged to Postgres + JSON
        drive intelligent improvements to each step
```

---

## Data

- `backend/data/electrical/` — 6 Lithonia/Acuity LED lighting spec sheets
- `backend/data/plumbing/` — 5 Kohler plumbing fixture spec sheets
- PDFs are primarily text-native with significant tabular data (spec tables, ordering codes)
- OCR fallback required for any image-rendered pages

---

## Frontend Views

| View | Route | Description |
|------|-------|-------------|
| Catalog | `/catalog` | Product grid with extracted specs, filterable by category |
| Search | `/search` | Natural language query → ranked matching products |
| Pipeline Log | `/pipeline` | Interview-facing: eval metric evolution across runs |

---

## Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| OCR | `pdfplumber` + `pytesseract` | pdfplumber handles text + tables natively; tesseract fallback for image pages |
| Table handling | `pdfplumber` `.extract_tables()` → markdown | preserves tabular structure for embeddings and extraction |
| Embeddings | `sentence-transformers` (local to start) | free, swappable — good eval story showing model upgrades |
| Vector DB | `pgvector` on existing Postgres | no new service; already dockerized |
| LLM Extraction | Claude API (`claude-haiku-4-5`) | structured output via tool_use; cost-efficient; API key confirmed |
| Backend | Django + Django REST Framework | extend existing setup |
| Frontend | React + Vite | already scaffolded |

---

## Django App Structure (backend/)

```
backend/
├── djangoApp/          # project settings
├── ingestion/          # Step 1–2: PDF parsing, OCR, chunking
├── embeddings/         # Step 3–4: embedding generation, pgvector storage
├── retrieval/          # Step 5: query handling, similarity search
├── extraction/         # Step 6: LLM structured extraction, Product model
├── evaluation/         # Step 7: metrics logging, run tracking
└── api/                # DRF endpoints consumed by frontend
```

---

## Database Schema (high-level)

```
PipelineRun
  id, created_at, config_snapshot (JSON), notes

Document
  id, filename, category, page_count, ocr_ratio, run_id

Chunk
  id, document_id, page, section, text, token_count, chunk_type (text|table)

ChunkEmbedding
  id, chunk_id, vector (pgvector), model_name

Product
  id, document_id, name, model_number, category, specs (JSON), run_id

EvaluationMetric
  id, run_id, step, metric_name, value, recorded_at
```

---

## Evaluation Metrics (per pipeline run)

| Step | Metric | How measured |
|------|--------|--------------|
| Ingestion | pages processed, OCR vs. native text ratio, tables extracted per doc | counted during parse |
| Chunking | avg chunk size (tokens), section coverage, text vs. table chunk ratio | computed post-split |
| Retrieval | precision@k | manually labeled small test set |
| Extraction | field fill rate, schema validation pass rate | checked against Product schema |
| End-to-end | query→correct product hit rate | labeled query/answer pairs |

Metrics are written to `EvaluationMetric` rows and optionally exported to
`plan/experiment_logs/` as JSON for version-controlled experiment history.

---

## Experiment Log Convention

Each significant pipeline change gets an entry in `plan/experiments/`:

```
plan/experiments/
  001_baseline_pdfplumber.md
  002_added_ocr_fallback.md
  003_switched_embeddings_model.md
  ...
```

Each file records: what changed, why, before/after metrics.

---

## Build Order

1. **Ingestion** — parse PDFs, detect OCR need, extract raw page text, log run metrics
2. **Chunking** — section-aware splitting, attach metadata
3. **Vector DB setup** — enable pgvector, define schema, migration
4. **Embeddings** — generate + store vectors
5. **Retrieval** — similarity search endpoint
6. **LLM Extraction** — structured product extraction, populate catalog
7. **API layer** — DRF endpoints for catalog, search, pipeline log
8. **Frontend** — Catalog, Search, Pipeline Log views
9. **Evaluation loop** — label test set, measure, improve, repeat

---

## Decisions Made

- [x] Anthropic API key available — Claude haiku for extraction, confirmed
- [x] PDFs are text-native with significant tabular data — pdfplumber table extraction is primary strategy
- [x] Embeddings: sentence-transformers (local) to start; upgrade path is an experiment opportunity

## Open Questions

- [ ] Are any PDFs password-protected or fully image-rendered?
- [ ] Target deployment: local only, or cloud eventually?
