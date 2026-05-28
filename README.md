# Global Congress & Publication Intelligence System

> **I'm currently open to new opportunities.**  
> I built this to show what's possible when you combine pharma domain knowledge with data engineering and AI automation. If your team is working on problems like this - medical intelligence, life sciences data, or AI in life sciences - I'd genuinely love to chat.  
> **[Nitin Paighowal on LinkedIn](https://www.linkedin.com/in/nitinpaighowal/)**

---

## What is this and why does it exist?

Medical affairs and competitive intelligence teams in pharma spend a surprising amount of time doing something that should be automated: tracking what gets published at medical congresses.

Every year, 50+ major congresses - ASCO, ASH, ESC, ESMO, AHA and dozens more - collectively produce tens of thousands of publications, abstracts, and trial readouts. For most teams, monitoring this means manual PubMed searches after each event, spreadsheets passed around by email, and a constant feeling that something important was missed.

I built this system to replace that workflow entirely.

It automatically collects publications from **53 major medical congresses** across **18 therapeutic areas**, classifies the type of evidence (RCT, meta-analysis, review, etc.), and presents everything through an interactive web interface - no database access or technical knowledge required for the end user.

# **[See it live here](https://paighowal.github.io/Global-Congress-Publication-Intelligence-System/)**
<img width="1900" height="887" alt="image" src="https://github.com/user-attachments/assets/62adb918-3184-445b-a837-5d0eb4d3afe9" />

<img width="1870" height="900" alt="image" src="https://github.com/user-attachments/assets/1c8d23c5-976f-4fa2-ae16-12d8df5ec41c" />

---

## The numbers

| | |
|---|---|
| Congresses tracked | 53 |
| Therapeutic areas | 18 |
| Publications indexed | 32,000+ |
| Years covered | 2018 – 2027 |
| Data source | PubMed / NCBI Entrez API |

---

## How the data is collected - the Python pipeline

This is the part I'm most proud of. No manual downloads, no copy-paste. The entire data collection process runs automatically through a Python pipeline.

### Step 1 - Congress registry

Every congress is defined in a JSON config file with its name, acronym, therapeutic areas, typical timing, and a PubMed search string tuned to that congress. Adding a new congress is just adding a new JSON block.

```json
{
  "name": "American Society of Clinical Oncology Annual Meeting",
  "acronym": "ASCO",
  "therapeutic_areas": ["Oncology", "Hematology"],
  "pubmed_search_term": "(\"ASCO\"[tiab] OR \"J Clin Oncol\"[journal]) AND (\"Annual Meeting\"[tiab])",
  "typical_month": [5, 6],
  "frequency": "annual"
}
```

### Step 2 - Automated PubMed search

For each congress and year, the pipeline hits the **NCBI Entrez API** (`esearch` + `efetch`) to pull every matching publication. It handles rate limits, batches requests in groups of 200, and retries on transient errors.

```python
# Simplified - search PubMed for ASCO 2024 publications
search_results = Entrez.esearch(
    db="pubmed",
    term=f"{congress.pubmed_search_term} AND 2024[pdat]"
)
records = Entrez.efetch(db="pubmed", id=pmids, rettype="xml")
```

### Step 3 - Publication type classification

PubMed assigns publication type tags to every paper (things like *Randomized Controlled Trial*, *Meta-Analysis*, *Review*, *Case Reports*). The pipeline reads those tags and maps them to one of 9 meaningful categories:

| PubMed tag | Classified as |
|---|---|
| Randomized Controlled Trial | `rct` |
| Meta-Analysis | `meta_analysis` |
| Systematic Review | `systematic_review` |
| Review | `review` |
| Clinical Trial (Phase I–IV) | `clinical_trial` |
| Case Reports | `case_report` |
| Letter / Editorial | `letter` / `editorial` |
| Everything else | `manuscript` |

This matters because not all publications carry the same weight. A congress that generates 500 RCTs tells a very different story than one generating 500 editorials.

### Step 4 - Storage and UI generation

Processed data lands in **PostgreSQL**. From there, two Python scripts generate static JavaScript data files that the frontend loads directly:

```
python scripts/gen_jsx_data.py   # → ui/planData.js  (congress/edition/year data)
python scripts/gen_pubs_data.py  # → ui/pubsData.js  (32K publications, indexed)
```

---

## Architecture

```
config/congress_config.json
        │  53 congresses defined here
        ▼
  Python Pipeline (core/pipeline.py)
  │
  ├── NCBI Entrez API (esearch + efetch)
  │     └── title, abstract, pub_type tags, PMID, DOI
  │
  ├── Publication type classifier
  │     └── RCT / meta-analysis / review / clinical trial / ...
  │
  └── PostgreSQL
        │
        ▼
  Data generators (scripts/)
  ├── gen_jsx_data.py   →  ui/planData.js   (~274 KB)
  └── gen_pubs_data.py  →  ui/pubsData.js   (~4.4 MB, indexed format)
        │
        ▼
  Static web app (ui/)
  ├── index.html          Congress calendar + year summary
  └── publications.html   Full publications browser
        │
        ▼
  GitHub Pages - zero infrastructure, no server required
```

The frontend is intentionally serverless. React 18 runs directly in the browser (no Node.js, no build step), data is pre-generated into JavaScript files, and the whole thing deploys as four static files. The only thing a user needs is a URL.

---

## What the interface does

### Congress calendar view

A year-by-quarter calendar showing all 53 congresses. Filter by therapeutic area to focus on what's relevant. Each congress card shows publication count and links to the published evidence. You can see at a glance how a congress has grown or shrunk over time.

### Publications browser

Search and filter across 32,000+ publications in real time. The filters are faceted - selecting "Oncology" as the therapeutic area automatically narrows the Congress dropdown to only show congresses with Oncology publications. Same logic applies across year, quarter, and publication type. Export any filtered view to CSV.

### Summary view

For a selected year, see total publications by congress, a breakdown by publication type (how many RCTs vs reviews vs case reports), and publication activity by therapeutic area - in one screen.

### 2027 planning

Congress stubs for 2027 are pre-populated based on historical timing, so planning teams can start mapping coverage before the year begins.

---

## Therapeutic areas covered

Oncology · Hematology · Cardiology · Neurology · Psychiatry & CNS · Diabetes & Metabolic · Rheumatology · Respiratory · Gastroenterology · Hepatology · Immunology · Dermatology · Infectious Disease · Nephrology · Urology · Endocrinology · Health Economics · Rare Disease

---

## Why this matters for business

**Medical affairs** can stop manually tracking congresses and spend that time on actual analysis. The system gives a structured, searchable record of what was published, where, and what type of evidence it represents.

**Competitive intelligence** teams can see publication patterns across years and therapeutic areas - spotting when a competitor's drug starts generating a surge of RCT-level evidence at specific congresses before it becomes common knowledge.

**Congress planning** - mapping 53 congresses across 4 quarters makes resource allocation tangible. You can see where the biggest evidence windows are and plan accordingly.

**Health economics and market access** - ISPOR and outcomes-focused congresses are tracked alongside clinical congresses, so the real-world evidence and HTA landscape lives in the same system as the clinical data.

**The scale argument:** a team manually tracking 10 congresses a year is probably spending 200–400 hours on PubMed searches, spreadsheet maintenance, and chasing down links. This system cuts that to near zero and covers 5× as many congresses.

---

## Tech stack

| Layer | Technology |
|---|---|
| Data collection | Python 3, NCBI Entrez API |
| Storage | PostgreSQL |
| DB driver | psycopg2 |
| Frontend framework | React 18 (UMD - runs in browser, no build needed) |
| JSX compilation | Babel Standalone |
| Styling | Inline React styles, Inter font |
| Deployment | GitHub Pages (static, no backend) |

---

## Running it locally

The frontend works immediately - just open `ui/index.html` in a browser. Data is already baked into the JS files.


---

## Project structure

```
pharmacongress/
├── config/
│   ├── congress_config.json    ← master congress registry (53 entries)
│   └── ui_config.json          ← colours, icons per therapeutic area
├── core/
│   ├── congress_registry.py    ← Python dataclasses + TherapeuticArea enum
│   └── pipeline.py             ← orchestrates scraping, classification, storage
├── scripts/
│   ├── gen_jsx_data.py         ← DB → planData.js (congress/edition data)
│   ├── gen_pubs_data.py        ← DB → pubsData.js (full publications, indexed)
│   ├── _sync_congresses.py     ← adds new congresses from config to DB
│   └── _splice_data.py         ← inlines planData.js into index.html
└── ui/
    ├── index.html              ← congress calendar + summary
    ├── publications.html       ← publications browser
    ├── planData.js             ← auto-generated (do not hand-edit)
    └── pubsData.js             ← auto-generated (do not hand-edit)
```

---

*Built by [Nitin Paighowal](https://www.linkedin.com/in/nitinpaighowal/) - open to roles in pharma tech, advanced analytics, data engineering, and AI in life sciences.*
