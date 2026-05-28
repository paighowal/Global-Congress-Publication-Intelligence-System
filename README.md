# ðŸ› Congress Intelligence System

AI-powered medical congress publication tracking with RAG-based querying.

**26 major congresses Â· 15 therapeutic areas Â· 2021â€“2028**

---

## Architecture

```
PubMed API â”€â”€â”€â”€â”€â”€â”
Semantic Scholar â”€â”¼â”€â”€â–º scrapers/pubmed_scraper.py
EuropePMC â”€â”€â”€â”€â”€â”€â”€â”˜
                          â”‚
                   core/pipeline.py  (orchestrator)
                          â”‚
           â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
           â–¼              â–¼                           â–¼
  core/ai_enrichment  database/models.py        rag/vector_store.py
  (Claude API)        (SQLAlchemy ORM)           (ChromaDB + RAG)
  â”€ Summaries         â”€ SQLite / PostgreSQL      â”€ Embeddings
  â”€ Drug mentions     â”€ Publications             â”€ Semantic search
  â”€ Study design      â”€ Authors / Tags           â”€ Claude synthesis
           â”‚              â”‚                           â”‚
           â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                          â–¼
                  api/api.py (FastAPI)
                  GET  /congresses
                  GET  /publications
                  POST /query  (RAG)
                  POST /scrape
                  GET  /stats
                          â”‚
                  core/scheduler.py
                  (APScheduler cron jobs)
```

---

## Congresses Covered

| Area | Congresses |
|------|-----------|
| **Oncology** | ASCO, ESMO, AACR, SABCS |
| **Hematology** | ASH |
| **Cardiology** | ESC, ACC, AHA, HFSA |
| **Neurology** | AAN, ECTRIMS, AAIC |
| **Diabetes & Metabolic** | ADA, EASD |
| **Rheumatology** | ACR, EULAR |
| **Respiratory** | ATS, ERS |
| **GI / Hepatology** | DDW, UEGW |
| **Infectious Disease** | IDWeek, CROI |
| **Dermatology** | AAD, EADV |
| **Nephrology** | ASN |
| **Psychiatry & CNS** | APA |

---

## Quick Start

### 1. Install dependencies
```bash
pip install -r requirements.txt
```

### 2. Configure
```bash
copy config\.env.example .env
# Edit .env and fill in ANTHROPIC_API_KEY
```

### 3. Initialize the database
```bash
python database/models.py
# Output: "Database initialized successfully."
```

### 4. Test a single congress (dry run)
```bash
python core/pipeline.py --congress ASCO --year-from 2024 --year-to 2024 --dry-run
```

### 5. Run with AI enrichment
```bash
python core/pipeline.py --congress ASCO --year-from 2021 --year-to 2026
```

### 6. Run all 26 congresses (2021-2028)
```bash
python core/pipeline.py
# Estimated: 2-8 hours, 50k-200k records, ~50k Claude API calls
```

### 7. Run a RAG query
```bash
python core/pipeline.py --query "What Phase 3 NSCLC trials at ASCO 2024 showed OS benefit?"
```

### 8. Start the REST API
```bash
uvicorn api.api:app --host 0.0.0.0 --port 8000 --reload
# Docs: http://localhost:8000/docs
```

### 9. Start the scheduler (automated updates)
```bash
python core/scheduler.py
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check |
| GET | `/congresses` | List all congresses |
| GET | `/congresses/registry` | Full registry (no DB) |
| GET | `/publications` | Search / filter publications |
| POST | `/query` | RAG natural-language query |
| POST | `/scrape` | Trigger background scrape |
| GET | `/stats` | System-wide statistics |

### Example RAG query
```bash
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What Phase 3 NSCLC trials were presented at ASCO or ESMO in 2023?",
    "year_from": 2023,
    "year_to": 2023,
    "n_results": 10
  }'
```

### Example publication search
```bash
curl "http://localhost:8000/publications?congress=ASCO&year=2024&drug=pembrolizumab&limit=20"
```

---

## Project Structure

```
pharmacongress/
├── core/
│   ├── congress_registry.py   # 26-congress master registry
│   ├── pipeline.py            # End-to-end orchestrator
│   ├── ai_enrichment.py       # Multi-provider LLM enrichment (Groq/Gemini/Anthropic)
│   └── scheduler.py           # APScheduler cron jobs
├── scrapers/
│   └── pubmed_scraper.py      # PubMed + SemanticScholar + EuropePMC
├── database/
│   ├── models.py              # SQLAlchemy ORM schema
│   └── migrate_legacy.py      # Legacy DB migration (SQLite or PostgreSQL)
├── rag/
│   └── vector_store.py        # ChromaDB + RAGQueryEngine
├── api/
│   └── api.py                 # FastAPI REST endpoints
├── ui/                        # React UI components
│   ├── GlobalPublicationPlan.jsx   # Full dashboard: all years, areas, filters
│   ├── PublicationPlan2026.jsx     # Legacy Dato-DXd 2026 plan (reference)
│   └── planData.js                 # Auto-generated DB data (run gen_jsx_data.py)
├── scripts/
│   ├── enrich_migrated.py     # Batch AI enrichment of publications
│   ├── vectorize_publications.py  # Build ChromaDB vector store
│   └── gen_jsx_data.py        # Export DB data → ui/planData.js
├── context/                   # Reference documents (not runtime)
│   ├── CongressIntelligence_ClaudeFeed.pdf
│   └── CongressIntelligence_ClaudeFeed.pdf.txt
├── data/
│   ├── db.sqlite              # Legacy prototype data (4,611 publications)
│   └── congress_intel.db      # New SQLite DB (mirrors PostgreSQL for local dev)
├── config/
│   └── .env.example
├── logs/                      # Runtime log output
├── requirements.txt
└── README.md
```

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | **Yes** | â€” | Claude AI enrichment & RAG |
| `DATABASE_URL` | No | `sqlite:///data/congress_intel.db` | DB connection string |
| `CHROMA_DIR` | No | `./chroma_db` | ChromaDB persistence dir |
| `NCBI_API_KEY` | No | â€” | PubMed 10 req/s (vs 3) |
| `SEMANTIC_SCHOLAR_API_KEY` | No | â€” | Higher S2 rate limits |

---

## Scheduler

Automated updates via APScheduler (`core/scheduler.py`):

| Schedule | Job |
|----------|-----|
| Every Monday 02:00 UTC | Incremental scrape (last 2 years) |
| 1st of each month 03:00 UTC | Full refresh 2021â€“2028 |
| Jun 15 & Dec 15 04:00 UTC | Oncology + Hematology deep-dive |

---

## Model Used

Claude model: **`claude-sonnet-4-6`** (current production default).  
Prompt caching (`cache_control: ephemeral`) is enabled on system prompts to reduce costs on repeated enrichment calls.

---

## Global Publication Plan UI

[ui/GlobalPublicationPlan.jsx](ui/GlobalPublicationPlan.jsx) is a React component that
displays all **4,611 tracked publications** across **26 congresses × 15 therapeutic areas × 2022–2026**
with interactive filters (year, quarter, therapeutic area, search) and three views: Calendar, Table, Summary.

Data is sourced from `congress_intel.db` and embedded in [ui/planData.js](ui/planData.js).
Regenerate the data file whenever the database is updated:

```bash
python scripts/gen_jsx_data.py
```

The original focused plan ([ui/PublicationPlan2026.jsx](ui/PublicationPlan2026.jsx)) covers
Dato-DXd 2026 only and is retained as a reference artifact.
