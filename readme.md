# pdf-to-query

An end-to-end pipeline that ingests product spec sheet PDFs, extracts structured data, and exposes it via natural language search and a catalog UI.

```
PDFs → OCR + layout parsing → chunking → embeddings → vector DB → retrieval → LLM extraction → API → UI
```

**Data source — v1:** Static PDFs in `backend/data/`
**Data source — v2:** Web scraping pipeline _(planned)_

Check out [approach.md](plan/approach.md) for my in-process notes including technical trade-offs and product considerations.

## Checklist

### PDF Parsing

- [x] Source representative documents
- [x] Generate ground truth files using SoTA vendors (LandingAI, Anthropic)
- [ ] Compare SoTA vendors _(ongoing)_
- [ ] Benchmark SoTA vendors (cost, latency)
- [ ] Implement and iterate on native PDF parsing pipeline (pdfplumber, tesseract, other)
- [ ] Evaluate native pipeline against SoTA

### Database

- [ ] Expand data model to support core app features and experiment outputs
- [ ] Assess pgvector vs. other vector DB vendors

### Chunking + Embedding

- [ ] To do

### Hybrid Search + Reranking

- [ ] To do

### Querying

- [ ] To do

### Evaluation

- [ ] To do

---

## Local Development

Copy `.env.example` to `.env` and fill in your values before starting.

### 1. Postgres (Docker)

```bash
docker compose up -d
```

### 2. Backend (Django)

```bash
cd backend
source ../.venv/bin/activate
python manage.py runserver
```

### 3. Frontend (React + Vite)

```bash
cd frontend
npm run dev
```
