# Approach

## Demo Scope

As a presentable demo, I considered building an E2E app that mimics what I imagine the Parspec ML pipeline is.

With some AI assistance, I derived the following plan:

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

## Anticipating Your Tech Stack

I assessed your tech stack by looking at the JDs. I arrived on the following:

### Application Layer

- React + TypeScript SPA
- Django or Go-based backend APIs
- RESTful services

### Core Data Layer

- MySQL / PostgreSQL – Primary relational databases
- DynamoDB – High-scale key-value lookups
- MongoDB – Flexible metadata storage
- Redis – Caching layer
- Elasticsearch / OpenSearch – Search & indexing

### AI Layer

- PyTorch + Hugging Face
- RAG pipelines
- Vector databases (Pinecone or Milvus)
- Custom ranking models
- Agentic orchestration frameworks

### Infrastructure

- AWS
- ECS / Kubernetes
- SQS + Lambda
- Docker
- GitHub Actions (CI/CD)

---

## Assessing My Skills and Experience

I've worked on frontends with (1) React+Vite, (2) NextJS, (3) Svelte. I've worked on the following backends — (1) NestJS (Node), (2) Sveltekit, (3) FastAPI. I'm most familiar with running PostgreSQL in Docker, and I have worked with hosted database services such as Supabase and Neon using Prisma and Drizzle as ORMs on separate projects. I've also done a short project using Flutter (Dart).

I've deployed the B2B app using Google Console, and the B2C app using Vercel.

For this project, I practiced setting up a Django backend and connected it with PostgreSQL running on Docker. I did this by directly following documentation instead of relying on AI assistance. I documented the concepts I needed to look up in a small vibe-coded concept card app — \<link\>.

> **Checkpoint:** At this point, I have a frontend app running using React+Vite, Django, and Postgres running on Docker, all locally.

---

## Document Sourcing

I grabbed 5 electrical and 5 plumbing PDFs from American manufacturers. From my prior experience working as a project engineer, I knew to look for PDFs that were:

- Multi-page
- A combination of images, text, tables, and drawings
- Had one-to-many or many-to-one product mappings, or 1-1 but confusing enough for AI

I recognize that there must be a periodic web crawling and indexing pipeline to enable this at Parspec. Challenges I can anticipate include:

- Limited access to specs behind paywalls or auth screens
- `robots.txt` files that reject scraping
- Scanned files
- Duplicate files with relatively similar content
- Marketing files that don't include product specifications
- Product files that remain online even when products are discontinued
- New product files added to sites that have already been scraped once
- Poor file naming standards from manufacturers
- Poor file versioning from manufacturers, making it hard to identify when a file was updated
- Finding small amd local manufacturers that have limited or non-existent online footprints
- Deciding whether data ingested from one company (Southland) can or should be made available to other M-P customers

Thinking outside the technology box, manufacturers have an incentive for their products to be discovered which creates incentives for them to share access to APIs or other data warehouses that create a win-win.

> **Question:** I'm curious if this avenue has been explored with technology forward manufacturers such as Hilti.
> **Question:** Do you also ingest the specification manuals/ project manuals from subcontractors the way Pype did to extract the product requirements to look them up against your database? Or are customers expected to bring that information from their existing PM software vendors such as Procore.

### Product considerations:

- Achieving >70% coverage of a given building code (for example, wash closets), before offering that to users as a feature
- Ensuring the coverage includes the most commonly used manufacturers, providing status quo or better product discovery

### Status:

- Working with sourced files.
  Todo:
- Explore web scraping pipeline, caching, etc.
- Tradeoffs include freshness of data, cost for new ingestions,

---

## Getting started

Creating ground truth examples and understanding what is SoTA:

Vendors:

- Landing.ai Agentic Document Extraction
- Claude (Opus 4.6)

- Used LandingAI ADE, to create one example of what good looks like
  - Can wire up API
- Used Opus in the Claude interface to interpret just the first page

Process:

- Used both vendors manually and eyeballed the outputs for page 1
  - Asked Claude Code to assess LandingAI's page 1 extraction treating Opus4.6 output as the ground truth
  - Reviewed the comparative report
-
- Saw the first two videos of the deeplearning.ai course on this to understand the tip of their process
  - TODO: Complete the rest of the course (3 hours)
-
