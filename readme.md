# pdf-to-query

An end-to-end pipeline that ingests product spec sheet PDFs, extracts structured data, and exposes it via natural language search and a catalog UI.

```
PDFs → OCR + layout parsing → chunking → embeddings → vector DB → retrieval → LLM extraction → API → UI
```

**Data source — v1:** Static PDFs in `backend/data/`
**Data source — v2:** Web scraping pipeline *(planned)*

See `plan/` for full architecture and approach.
