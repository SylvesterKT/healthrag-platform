# HealthRAG Platform

![Python](https://img.shields.io/badge/Python-3.11-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-0.111-green) ![LangChain](https://img.shields.io/badge/LangChain-0.2-green) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue) ![Chroma](https://img.shields.io/badge/ChromaDB-Vector-orange) ![License](https://img.shields.io/badge/License-MIT-yellow)

> Healthcare RAG platform — retrieval-augmented generation for clinical decision support, grounded in PubMed, clinical guidelines, drug databases, and de-identified patient records.

## Vision

HealthRAG bridges the gap between LLM reasoning and clinical evidence. It ingests authoritative medical sources (PubMed, NICE guidelines, BNF drug database), indexes them as vector embeddings, and serves clinically-grounded answers to physician queries — with source citations and confidence scores.

Every response is traceable back to source documents. No hallucinations without accountability.

## RAG Architecture

```
┌────────────────────────────────────────────────┐
│               Knowledge Sources                    │
│  PubMed │ NICE Guidelines │ BNF │ UpToDate         │
└───────────────────┬───────────────────────────┘
                    │ Ingestion Pipeline
┌───────────────────▼───────────────────────────┐
│         Chunking + Embedding + Indexing             │
│      (text-embedding-3-large, ChromaDB)             │
└───────────────────┬───────────────────────────┘
                    │
              Clinical Query
                    │
┌───────────────────▼───────────────────────────┐
│    Semantic Search (Top-K relevant chunks)          │
│      + Reranking (cross-encoder)                    │
└───────────────────┬───────────────────────────┘
                    │
┌───────────────────▼───────────────────────────┐
│     GPT-4o Generation with Grounded Context        │
│     Answer + Citations + Confidence Score          │
└───────────────────────────────────────────────┘
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| API | FastAPI, Python 3.11 |
| LLM | GPT-4o (Azure OpenAI) |
| Embeddings | text-embedding-3-large |
| Vector Store | ChromaDB (persistent) |
| Reranker | cross-encoder/ms-marco |
| Knowledge Sources | PubMed API, NICE, BNF |
| Database | PostgreSQL 16 (audit log) |
| Cache | Redis 7 |
| Auth | JWT, Role-based (Physician, Admin) |

## Key Features

- **PubMed ingestion** — automated daily sync of new publications for target medical domains
- **Semantic chunking** — sentence-aware splitting that preserves clinical context
- **Cross-encoder reranking** — retrieves top-K chunks then reranks by clinical relevance
- **Grounded generation** — GPT-4o answers strictly grounded in retrieved sources with inline citations
- **Confidence scoring** — each answer includes a RAG confidence score based on retrieval similarity
- **Full audit trail** — every query, retrieval, and generation logged for clinical accountability

## Project Structure

```
healthrag-platform/
├── ingestion/
│   ├── pubmed_sync.py        # Daily PubMed ingestion
│   ├── chunker.py            # Semantic text chunking
│   ├── embedder.py           # Generate and store embeddings
│   └── nice_guidelines.py    # NICE/BNF document ingestion
├── retrieval/
│   ├── vector_search.py      # ChromaDB semantic search
│   └── reranker.py           # Cross-encoder reranking
├── generation/
│   ├── rag_chain.py          # LangChain RAG pipeline
│   └── citation_formatter.py # Source attribution
├── api/
│   ├── main.py
│   ├── routes/
│   │   ├── query.py           # Clinical query endpoint
│   │   ├── documents.py       # Document management
│   │   └── auth.py
│   └── middleware/
│       └── audit_logger.py
├── tests/
├── requirements.txt
├── docker-compose.yml
└── README.md
```

## API Endpoints

```
POST /api/v1/query                  Submit clinical query
GET  /api/v1/query/{id}             Get answer with citations
POST /api/v1/documents/ingest       Trigger document ingestion
GET  /api/v1/documents/             List indexed documents
GET  /api/v1/audit/                 Query audit log (admin)
POST /api/v1/auth/login             JWT login
```

## Example Response

```json
{
  "query": "First-line treatment for newly diagnosed GBM?",
  "answer": "Standard of care is maximal safe surgical resection followed by Stupp protocol: concurrent temozolomide with radiotherapy (60 Gy / 30 fractions), then 6 cycles adjuvant temozolomide.",
  "confidence": 0.94,
  "citations": [
    {"pmid": "16166120", "title": "Stupp et al. NEJM 2005", "relevance": 0.97},
    {"pmid": "26296372", "title": "NICE guideline NG99", "relevance": 0.91}
  ]
}
```

## Quick Start

```bash
git clone https://github.com/SylvesterKT/healthrag-platform
cd healthrag-platform
cp .env.example .env  # Add OPENAI_API_KEY
docker-compose up --build
python ingestion/pubmed_sync.py --query "glioblastoma treatment" --limit 1000
```

## Roadmap

- [x] PubMed ingestion pipeline
- [x] ChromaDB vector indexing
- [x] Cross-encoder reranking
- [x] GPT-4o grounded generation
- [ ] FHIR R4 patient record integration
- [ ] Drug interaction checker
- [ ] Multi-lingual query support

## Resume Impact

Built a production-grade healthcare RAG platform integrating PubMed ingestion, ChromaDB vector search, cross-encoder reranking, and GPT-4o grounded generation — demonstrating clinical AI engineering with full source attribution and audit trails.

## License

MIT © Sylvester KT
