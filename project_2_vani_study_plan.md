# 🔥 Project 2: Vani — AI-Native Document & Media Ingestion Engine
# Complete Study & Build Plan

> **Goal:** Build an open-source, air-gap-ready, containerized document and media ingestion engine with an MCP server, hybrid search, and full audit logging — targeting the Backend Engineer (Chanakya) role at Sarvam AI.

---

## Table of Contents

1. [Project Summary](#project-summary)
2. [Backend Phases & Skills Required](#backend-phases--skills-required)
3. [AI Engineering Phases & Skills Required](#ai-engineering-phases--skills-required)
4. [Skills You DON'T Need for This Project](#skills-you-dont-need-for-this-project)
5. [Recommended Study Order (The Flow)](#recommended-study-order-the-flow)
6. [Week-by-Week Execution Plan](#week-by-week-execution-plan)
7. [Key Deliverables Checklist](#key-deliverables-checklist)

---

## Project Summary

Vani is NOT a chatbot, and it's NOT a generic RAG tutorial. It's a **self-contained backend service** that ingests unstructured data (PDFs, audio, images, geospatial feeds) from multiple sources, processes it into structured, searchable knowledge, and exposes it through clean APIs — including an **MCP server** for LLM agent tool-use. It's designed for **air-gapped, operationally constrained environments** (defence/government) where nothing phones home to any cloud API.

### Core Components You're Building

| Component | What It Is |
|---|---|
| **Multi-Format Ingestion** | Parsers for PDFs, DOCX, images (OCR), audio (Whisper), CSV, JSON, GeoJSON/KML |
| **Chunking Engine** | Semantic chunking, sliding window, table extraction, metadata preservation |
| **Embedding & Vector Store** | Local embedding models → Qdrant (embedded mode) for vector search |
| **Hybrid Search** | Reciprocal Rank Fusion (RRF) combining Qdrant vectors + SQLite FTS5 keyword search |
| **MCP Server** | Model Context Protocol server exposing ingested knowledge to LLM agents as tools |
| **NL-to-Action API** | Natural language queries → structured database operations |
| **Audit Engine** | Append-only SQLite with hash chain — tamper-evident, full provenance logging |
| **Air-Gap Mode** | All models local, zero API calls, single Docker container, no telemetry |

### Tech Stack

- **API Server:** FastAPI + asyncio + Pydantic
- **PDF Processing:** PyMuPDF + unstructured
- **Audio Transcription:** Whisper.cpp (local)
- **OCR:** Tesseract + EasyOCR
- **Geospatial:** GeoPandas + Shapely
- **Embeddings:** sentence-transformers (local)
- **Vector Store:** Qdrant (embedded mode)
- **Full-Text Search:** SQLite FTS5
- **Metadata Store:** PostgreSQL (or SQLite for single-node)
- **MCP Protocol:** mcp-python SDK
- **Containerization:** Docker (multi-stage build)
- **Audit Log:** Append-only SQLite with hash chain

---

## Backend Phases & Skills Required

These are the chapters from [Backend-from-first-Principle](file:///c:/Lab/Backend-from-first-Principle) you need to study.

### 🔴 Critical (Must Do Before Building)

| # | Chapter | Why You Need It for Vani | Specific Skills |
|---|---|---|---|
| **1** | [HTTP and CORS](file:///c:/Lab/Backend-from-first-Principle/1.HTTP-AND-CORS) | Vani's FastAPI server communicates via HTTP. The MCP server and all REST endpoints need proper HTTP lifecycle understanding. CORS for when agents access Vani from web UIs | HTTP methods, status codes, headers, CORS policy |
| **2** | [Routing in Backend](file:///c:/Lab/Backend-from-first-Principle/2.Routing-in-backend) | Vani exposes multiple endpoints: `POST /ingest`, `GET /search`, `POST /query`, `GET /documents/{id}`, `GET /audit-log`, `WebSocket /mcp`. Route design is core | Route handlers, path params, query params, route organization |
| **3** | [Serialization and Deserialization](file:///c:/Lab/Backend-from-first-Principle/3.Serialization-and-Deserialization-or-backend-engineers) | Documents are parsed into chunks → embedded → serialized to JSON for API responses. MCP protocol uses structured JSON messages. Multi-format data crosses many boundaries | JSON parsing, schema validation, Pydantic models, binary data handling |
| **5** | [Validations and Transformations](file:///c:/Lab/Backend-from-first-Principle/5.%20Validations%20and%20transformations%20for%20backend%20engineers) | File uploads need validation (is it a real PDF? is the file size within limits? is the GeoJSON valid?). Query parameters need sanitization | Pydantic validators, file type validation, custom validation rules, data sanitization |
| **6** | [Controllers, Services, Repositories, Middlewares](file:///c:/Lab/Backend-from-first-Principle/6.controllers-services-repositories-middlewares-and-request-context) | Vani's codebase needs clean separation: API controllers → ingestion services → parser modules → storage repositories. Middleware for audit logging, request context tracking | Layered architecture, dependency injection, middleware chains, request context |
| **7** | [API Design — REST API](file:///c:/Lab/Backend-from-first-Principle/7.API-DESIGN-RestAPI) | Designing Vani's API surface: document CRUD, search endpoints, ingestion triggers, MCP protocol endpoint, audit log queries | RESTful conventions, resource naming, versioned APIs, error responses, file upload endpoints |
| **8** | [Database with Backend](file:///c:/Lab/Backend-from-first-Principle/8.Database-with-backend) | Vani stores document metadata, chunk records, audit logs, and search indices. PostgreSQL for metadata, SQLite for audit + FTS5. Understanding schemas, queries, and migrations is essential | DB connections, SQL queries, ORM basics, migrations, PostgreSQL + SQLite dual usage |

### 🟡 Important (Need During Building)

| # | Chapter | Why You Need It for Vani | Specific Skills |
|---|---|---|---|
| **9** | [Caching](file:///c:/Lab/Backend-from-first-Principle/9.Caching,%20the%20secret%20behind%20it%20all) | Caching embedding computations (same document re-ingested), caching search results, caching parsed document chunks to avoid re-processing | Cache strategies (TTL, LRU), embedding cache, result caching |
| **12** | [Error Handling and Fault-Tolerant Systems](file:///c:/Lab/Backend-from-first-Principle/12.%20Error%20Handling%20and%20Building%20Fault%20Tolerant%20Systems) | Ingestion pipelines WILL fail — corrupt PDFs, invalid audio files, OCR timeouts. Vani needs graceful error handling per-document without crashing the batch. Retry logic for flaky parsers | Error propagation, retry strategies, partial failure handling, graceful degradation |
| **14** | [Production-grade Configuration Management](file:///c:/Lab/Backend-from-first-Principle/14.Production-grade%20Configuration%20Management) | Vani needs configurable pipeline settings: embedding model path, chunk sizes, OCR language, Qdrant config, FTS5 settings. All without hardcoding, especially for air-gap deployments where env vars differ | Config loading, env vars, config validation, environment-specific configs |
| **15** | [Logging, Monitoring, and Observability](file:///c:/Lab/Backend-from-first-Principle/15.Logging,%20Monitoring%20and%20Observability) | The audit engine needs structured logging. Production Vani needs health checks, ingestion metrics (docs/sec, failures, queue depth), and observability for air-gap operators | Structured logging, JSON log format, health check endpoints, metrics |
| **4** | [Authentication and Authorization](file:///c:/Lab/Backend-from-first-Principle/4.Authentication%20and%20authorization%20for%20backend%20engineers) | Multi-tenant access to the knowledge base. API key validation for MCP clients. Tenant isolation — one tenant cannot search another's documents | API keys, tenant identification, RBAC basics, resource isolation |

### 🟢 Nice to Have (Polish Phase)

| # | Chapter | Why You Need It for Vani | Specific Skills |
|---|---|---|---|
| **16** | [Graceful Shutdown](file:///c:/Lab/Backend-from-first-Principle/16.Graceful%20Shutdown) | When shutting down Vani, active ingestion jobs need to complete or checkpoint. Embedded Qdrant needs clean shutdown to avoid index corruption | Signal handlers, drain connections, cleanup, checkpoint & resume |
| **20** | [Concurrency & Parallelism](file:///c:/Lab/Backend-from-first-Principle/20.Concurrency%20&%20Parallelism%20-%20IO%20Bound%20vs%20CPU%20Bound) | Ingesting multiple documents concurrently (asyncio for I/O-bound parsing, process pool for CPU-bound OCR/Whisper). Understanding when to use threads vs processes vs async is critical for throughput | asyncio, ProcessPoolExecutor, IO-bound vs CPU-bound, concurrent ingestion |
| **21** | [Containerization — Docker, K8s, CI/CD](file:///c:/Lab/Backend-from-first-Principle/21.Containerization-and-Deployment-Docker-Kubernetes-and-CICD) | Vani's air-gap design requires a single Docker container with all models baked in. Multi-stage builds, model caching layers, deterministic builds | Dockerfile, multi-stage builds, model baking, docker-compose |
| **22** | [Automated Testing](file:///c:/Lab/Backend-from-first-Principle/22.Automated-Testing-Unit-Integration-and-E2E) | Testing parsers (does PDF extraction work?), testing search quality (does hybrid search return relevant results?), testing MCP protocol compliance | pytest, fixtures, mocking file uploads, integration tests |
| **24** | [WebSockets and Real-Time Communication](file:///c:/Lab/Backend-from-first-Principle/24.Web-Sockets-And-Real-time-Communication-with-WebSockets) | The MCP server uses WebSocket transport (`WebSocket /mcp`). Understanding the WebSocket protocol is needed for the MCP server implementation | WebSocket protocol, connection management, message framing |

---

## AI Engineering Phases & Skills Required

These are the phases from [ai-engineering-from-scratch](file:///c:/Lab/ai-engineering-from-scratch/phases) you need to study.

### 🔴 Critical — Foundations (Must Do First)

| Phase | Name | Why You Need It for Vani | Key Lessons |
|---|---|---|---|
| **Phase 00** | [Setup and Tooling](file:///c:/Lab/ai-engineering-from-scratch/phases/00-setup-and-tooling) | Dev environment, Python envs, Docker for AI, API key management (even in air-gap, you need local model config) | `01-dev-environment`, `06-python-environments`, `07-docker-for-ai` |
| **Phase 10** (partial) | [LLMs from Scratch](file:///c:/Lab/ai-engineering-from-scratch/phases/10-llms-from-scratch) | Understanding tokenizers helps with chunking strategies (token-aware chunk boundaries). Understanding how embeddings work at a low level | `01-tokenizers`, `02-building-a-tokenizer` |

### 🔴 Critical — Core Vani Skills (The Heart of the Engine)

| Phase | Name | Why You Need It for Vani | Key Lessons |
|---|---|---|---|
| **Phase 11** (selected) | [LLM Engineering](file:///c:/Lab/ai-engineering-from-scratch/phases/11-llm-engineering) | **Embeddings, RAG, and Advanced RAG are the backbone of Vani.** Chunking strategies, embedding model selection, vector search, hybrid retrieval, re-ranking — this IS the ingestion-to-search pipeline. Structured outputs for NL-to-Action API. Guardrails for query safety | `04-embeddings` → embedding pipeline, `06-rag` → basic retrieval pipeline, `07-advanced-rag` → chunking strategies/hybrid search/re-ranking, `03-structured-outputs` → NL-to-Action parsing, `12-guardrails` → query safety, `14-model-context-protocol` → MCP awareness |
| **Phase 13** (selected) | [Tools and Protocols](file:///c:/Lab/ai-engineering-from-scratch/phases/13-tools-and-protocols) | **Vani IS an MCP server.** Understanding MCP fundamentals, building MCP servers, tool schema design, and how agents consume tools is directly required | `05-tool-schema-design` → MCP tool definitions, `06-mcp-fundamentals` → MCP protocol, `07-building-an-mcp-server` → **THE most relevant lesson**, `04-structured-output` → response formatting |

### 🟡 Important — Deepening Skills

| Phase | Name | Why You Need It for Vani | Key Lessons |
|---|---|---|---|
| **Phase 05** (selected) | [NLP Foundations](file:///c:/Lab/ai-engineering-from-scratch/phases/05-nlp-foundations-to-advanced) | Understanding embeddings deeply (for choosing models), semantic similarity (for search quality), multilingual NLP (Hindi document support) | `03-word-embeddings-word2vec` → embedding intuition, `22-embedding-models-deep-dive` → choosing embedding models, `18-multilingual-nlp` → Hindi document support |
| **Phase 14** (selected) | [Agent Engineering](file:///c:/Lab/ai-engineering-from-scratch/phases/14-agent-engineering) | Understanding how agents consume tools (Vani is the tool provider, not the agent). Memory architectures inform Vani's storage design | `06-tool-use-and-function-calling` → how agents call Vani, `07-memory-virtual-context-memgpt` → storage patterns, `09-hybrid-memory-mem0` → hybrid search inspiration |
| **Phase 06** (selected) | [Speech and Audio](file:///c:/Lab/ai-engineering-from-scratch/phases/06-speech-and-audio) | Vani ingests audio files using Whisper for transcription. Understanding ASR pipeline, audio preprocessing, and transcription quality | `01-audio-fundamentals` → audio preprocessing, `03-automatic-speech-recognition` → Whisper pipeline, `07-text-to-speech` → understanding audio formats |

### 🟢 Nice to Have — Advanced Topics

| Phase | Name | Why You Need It for Vani | Key Lessons |
|---|---|---|---|
| **Phase 17** (selected) | [Infrastructure & Production](file:///c:/Lab/ai-engineering-from-scratch/phases/17-infrastructure-and-production) | Production deployment of Vani, observability setup, containerized AI model serving | `13-llm-observability` → monitoring setup, `20-shadow-canary-progressive` → deployment strategies |
| **Phase 18** (selected) | [Ethics, Safety, Alignment](file:///c:/Lab/ai-engineering-from-scratch/phases/18-ethics-safety-alignment) | Audit trail design, data handling ethics for classified documents, PII detection in ingested content | `29-moderation-systems-openai-perspective-llamaguard` → content moderation in ingested docs |

---

## Skills You DON'T Need for This Project

These phases are **NOT required** for Vani. Don't waste time on them:

| Phase | Name | Why You Can Skip It |
|---|---|---|
| Phase 01 | Math Foundations | Not building ML models from scratch |
| Phase 02 | ML Fundamentals | Using pre-trained embedding models, not training |
| Phase 03 | Deep Learning Core | Same — using, not building |
| Phase 04 | Computer Vision | Vani uses OCR libraries, not building CV models |
| Phase 07 | Transformers Deep Dive | Using transformers via libraries, no need to build from scratch |
| Phase 08 | Generative AI (images) | No image generation |
| Phase 09 | Reinforcement Learning | Not training agents with RL |
| Phase 10 (most of it) | LLMs from Scratch (advanced) | Don't need to build an LLM, just use embedding models |
| Phase 12 | Multimodal AI | Vani processes each modality separately, no cross-modal reasoning |
| Phase 15 | Autonomous Systems | Vani is a backend service, not an autonomous agent |
| Phase 16 | Multi-Agent & Swarms | Single service, not multi-agent |
| AI Phase 11 (most) | LLM Engineering (prompt eng, few-shot, function calling, LangGraph) | Vani doesn't orchestrate LLM conversations — it serves data |
| Backend #10 | Task Queues | Could be useful later but not for core Vani v1 |
| Backend #11 | Elasticsearch | Vani uses SQLite FTS5 for full-text search, not ES |
| Backend #13 | gRPC | REST + WebSocket is sufficient for Vani v1 |
| Backend #17 | Backend Security (deep) | Basic API key auth is enough |
| Backend #18-19 | Scaling & Performance | Premature for v1 |
| Backend #23 | Message Brokers / Kafka | Not needed for single-container deployment |

---

## Recommended Study Order (The Flow)

> **Strategy:** Learn → Build incrementally. Don't study everything first. Study a block, then build the corresponding Vani component.

```
📚 STUDY BLOCK 1: "Foundations"                          ⏱️ ~3-4 days
├── Backend #1: HTTP and CORS
├── Backend #2: Routing
├── Backend #3: Serialization/Deserialization
├── AI Phase 00: Setup and Tooling (selected lessons)
└── AI Phase 10: Tokenizers (01-02 only)
         │
         ▼
📚 STUDY BLOCK 2: "Embeddings & RAG Core"               ⏱️ ~4-5 days
├── AI Phase 11: Embeddings (04)
├── AI Phase 11: RAG (06)
├── AI Phase 11: Advanced RAG (07)
├── AI Phase 05: Word Embeddings (03) — intuition
├── AI Phase 05: Embedding Models Deep Dive (22)
└── AI Phase 11: Structured Outputs (03) — for NL-to-Action
         │
         ▼
🔨 BUILD CHECKPOINT 1: "Hello Ingestion"                 ⏱️ ~2-3 days
│   Build: Basic PDF parser → chunking engine → embed
│   chunks with sentence-transformers → store in Qdrant
│   (embedded mode) → simple semantic search endpoint.
│   No MCP, no audit, no audio, no OCR yet.
│
         ▼
📚 STUDY BLOCK 3: "Backend Architecture"                 ⏱️ ~3-4 days
├── Backend #5: Validations
├── Backend #6: Controllers/Services/Middlewares
├── Backend #7: API Design (REST)
├── Backend #8: Database with Backend
└── Backend #14: Configuration Management
         │
         ▼
📚 STUDY BLOCK 4: "Hybrid Search & Data Layer"           ⏱️ ~2-3 days
├── Backend #8: Database deep dive (PostgreSQL + SQLite)
├── SQLite FTS5 — study documentation / tutorials
└── Reciprocal Rank Fusion (RRF) — algorithm study
         │
         ▼
🔨 BUILD CHECKPOINT 2: "Hybrid Search + REST API"        ⏱️ ~3-4 days
│   Build: Add SQLite FTS5 keyword search alongside
│   Qdrant vector search. Implement RRF fusion. Build
│   FastAPI endpoints: POST /ingest, GET /search,
│   GET /documents/{id}. Proper layered architecture.
│
         ▼
📚 STUDY BLOCK 5: "MCP & Tool Protocol"                  ⏱️ ~3-4 days
├── AI Phase 13: Tool Schema Design (05)
├── AI Phase 13: MCP Fundamentals (06)
├── AI Phase 13: Building an MCP Server (07)
├── AI Phase 14: Tool Use & Function Calling (06)
├── Backend #24: WebSockets (for MCP transport)
└── AI Phase 11: Model Context Protocol (14)
         │
         ▼
🔨 BUILD CHECKPOINT 3: "MCP Server"                      ⏱️ ~2-3 days
│   Build: Implement MCP server with tool definitions
│   (search_documents, ingest_document, get_stats).
│   WebSocket transport. Test with an MCP client.
│
         ▼
📚 STUDY BLOCK 6: "Multi-Format Parsers"                 ⏱️ ~3-4 days
├── AI Phase 06: Audio Fundamentals (01)
├── AI Phase 06: ASR / Whisper pipeline (03)
├── OCR libraries — Tesseract + EasyOCR docs/tutorials
├── GeoPandas + Shapely — geospatial data tutorials
└── Backend #12: Error Handling (for parser failures)
         │
         ▼
🔨 BUILD CHECKPOINT 4: "Multi-Format Ingestion"          ⏱️ ~3-4 days
│   Build: Add parsers for audio (Whisper.cpp local),
│   images (OCR), DOCX, CSV, GeoJSON/KML. Parser
│   orchestrator that detects document type and routes
│   to appropriate parser. Graceful failure per-doc.
│
         ▼
📚 STUDY BLOCK 7: "Audit, Security & NL-to-Action"       ⏱️ ~2-3 days
├── Backend #4: Authentication & Authorization
├── Backend #15: Logging, Monitoring, Observability
├── AI Phase 18: Moderation Systems (29) — PII detection
├── AI Phase 11: Guardrails (12) — query safety
└── Hash chain / tamper-evident log design study
         │
         ▼
🔨 BUILD CHECKPOINT 5: "Audit Trail + NL-to-Action"      ⏱️ ~3-4 days
│   Build: Append-only audit log with hash chain.
│   Every ingest, search, and query logged with full
│   provenance. POST /query NL-to-Action endpoint.
│   API key auth, tenant isolation, health checks.
│
         ▼
📚 STUDY BLOCK 8: "Air-Gap & Production Polish"          ⏱️ ~2-3 days
├── Backend #21: Docker (multi-stage builds, model baking)
├── Backend #16: Graceful Shutdown
├── Backend #20: Concurrency (asyncio + ProcessPool)
├── Backend #22: Automated Testing
└── AI Phase 05: Multilingual NLP (18) — Hindi docs
         │
         ▼
🔨 BUILD CHECKPOINT 6: "Air-Gap Ready Vani"              ⏱️ ~3-4 days
│   Build: Multi-stage Docker build with all models
│   baked in. Zero external API calls. Deterministic
│   builds with pinned versions + SHA-verified weights.
│   Health checks, structured JSON logs, graceful
│   shutdown. Concurrent ingestion pipeline.
│
         ▼
🔨 FINAL: "Ship It"                                      ⏱️ ~2-3 days
│   README with architecture diagram, Quick Start guide
│   (ingest a sample PDF + query via MCP in < 5 min),
│   demo recording, Design Decisions doc, test suite.
```

---

## Week-by-Week Execution Plan

### Week 1: Foundations & First Search Pipeline
**Study:** Blocks 1 + 2 (HTTP/Routing/Serialization + Embeddings/RAG Core)

| Day | Activity | Output |
|---|---|---|
| Day 1 | Backend #1 (HTTP/CORS) + Backend #2 (Routing) | Notes: HTTP lifecycle, route patterns |
| Day 2 | Backend #3 (Serialization) + AI Phase 00 (Setup) | Working dev environment, Python envs, Docker |
| Day 3 | AI Phase 10: Tokenizers (01-02) | Understand tokenization, token-aware chunking |
| Day 4 | AI Phase 11: Embeddings (04) + Phase 05: Word Embeddings (03) | Embedding vectors, similarity metrics, model intuition |
| Day 5 | AI Phase 11: RAG (06) + Advanced RAG (07) | Chunking strategies, retrieval pipeline, re-ranking |
| Day 6 | AI Phase 05: Embedding Models Deep Dive (22) + AI Phase 11: Structured Outputs (03) | Choosing embedding models, JSON mode outputs |
| Day 7 | **BUILD:** Checkpoint 1 — PDF → chunks → embed → Qdrant → search | Basic ingestion + semantic search working |

**Milestone:** Can ingest a PDF, chunk it intelligently, embed it, and search it semantically.

---

### Week 2: Backend Architecture + Hybrid Search + REST API
**Study:** Blocks 3 + 4 (Backend Architecture + Hybrid Search)
**Build:** Checkpoint 2

| Day | Activity | Output |
|---|---|---|
| Day 8 | Backend #5 (Validations) + #6 (Architecture Patterns) | Understand layered architecture, validators |
| Day 9 | Backend #7 (API Design) + #14 (Config Management) | REST API design, config loading patterns |
| Day 10 | Backend #8 (Database with Backend) — PostgreSQL + SQLite | DB schemas, queries, dual-database patterns |
| Day 11 | SQLite FTS5 deep-dive + Reciprocal Rank Fusion study | Full-text search indexing, RRF fusion algorithm |
| Day 12 | **BUILD:** Hybrid search (Qdrant + FTS5 + RRF) | Hybrid search returning fused ranked results |
| Day 13 | **BUILD:** FastAPI REST endpoints + layered architecture | POST /ingest, GET /search, GET /documents/{id} |
| Day 14 | **BUILD:** Polish Checkpoint 2 — validation, config, error handling | Clean, well-structured REST API for ingestion + search |

**Milestone:** Full REST API with hybrid search (vector + keyword), proper architecture, validation.

---

### Week 3: MCP Server + Multi-Format Parsers
**Study:** Blocks 5 + 6 (MCP Protocol + Multi-Format Parsers)
**Build:** Checkpoints 3 + 4

| Day | Activity | Output |
|---|---|---|
| Day 15 | AI Phase 13: Tool Schema Design (05) + MCP Fundamentals (06) | Understand MCP protocol, tool schemas |
| Day 16 | AI Phase 13: Building an MCP Server (07) + Backend #24 (WebSockets) | MCP server implementation pattern, WebSocket transport |
| Day 17 | AI Phase 11: MCP (14) + AI Phase 14: Tool Use (06) | How agents consume MCP tools (Vani's consumer perspective) |
| Day 18 | **BUILD:** Checkpoint 3 — MCP server with search_documents + ingest_document tools | Working MCP server tested with client |
| Day 19 | AI Phase 06: Audio Fundamentals (01) + ASR/Whisper (03) + OCR docs | Audio preprocessing, Whisper pipeline, Tesseract/EasyOCR |
| Day 20 | GeoPandas + Shapely tutorials + Backend #12 (Error Handling) | Geospatial parsing, graceful parser failure handling |
| Day 21 | **BUILD:** Checkpoint 4 — audio, image, DOCX, GeoJSON parsers + orchestrator | Parser orchestrator routes by file type, handles failures |

**Milestone:** MCP server working + Vani can ingest PDFs, audio, images, DOCX, and geospatial data.

---

### Week 4: Audit Trail, NL-to-Action, Air-Gap & Ship
**Study:** Blocks 7 + 8 (Audit/Security + Air-Gap/Production)
**Build:** Checkpoints 5 + 6 + Final

| Day | Activity | Output |
|---|---|---|
| Day 22 | Backend #4 (Auth) + Backend #15 (Observability) | API keys, tenant isolation, structured logging |
| Day 23 | AI Phase 18: Moderation (29) + AI Phase 11: Guardrails (12) + hash chain study | PII detection, query safety, tamper-evident log design |
| Day 24 | **BUILD:** Checkpoint 5 — audit trail (hash chain) + NL-to-Action + auth | Append-only audit log, POST /query, API key auth |
| Day 25 | Backend #21 (Docker multi-stage) + Backend #20 (Concurrency) | Model baking, asyncio + ProcessPool for parallel ingestion |
| Day 26 | Backend #16 (Graceful Shutdown) + Backend #22 (Testing) + AI Phase 05: Multilingual (18) | Clean shutdown, pytest suite, Hindi document support |
| Day 27 | **BUILD:** Checkpoint 6 — Docker (air-gap ready) + concurrent ingestion + tests | Single Docker container, zero external calls, test suite |
| Day 28 | **FINAL:** README, Quick Start, architecture diagram, demo recording, Design Decisions | Ship-ready GitHub repo |

**Milestone:** Production-ready, air-gap-ready Vani on GitHub.

---

## Key Deliverables Checklist

When you're done, your repo should have:

- [ ] **README.md** with architecture diagram (ASCII or Mermaid)
- [ ] **Quick Start** section — ingest a sample PDF + query via MCP in < 5 minutes
- [ ] **MCP Server** that LLM agents can connect to and use for document search
- [ ] **Hybrid search** demo showing vector + keyword fusion outperforming either alone
- [ ] **At least 3 parser demos:**
  - PDF ingestion (with table extraction)
  - Audio transcription (Whisper, Hindi audio)
  - Image OCR (scanned Hindi document)
- [ ] **Air-gap proof:**
  - Single Docker container, zero external API calls
  - All models baked into the image
  - No telemetry, no phone-home
  - Deterministic, reproducible builds
- [ ] **Audit trail** showing full provenance chain for every operation
- [ ] **NL-to-Action** demo — natural language query → structured result
- [ ] **Design Decisions** doc explaining WHY you chose:
  - Qdrant embedded mode over a separate vector DB server
  - SQLite FTS5 over Elasticsearch for full-text search
  - Reciprocal Rank Fusion over other hybrid strategies
  - Append-only hash chain over a regular database for audit
  - Multi-stage Docker build for air-gap model baking
- [ ] **Test suite** — parser tests, search quality tests, MCP protocol tests
- [ ] **Docker support** — `docker build` → `docker run` → working Vani

---

## Total Time Estimate

| Activity | Time |
|---|---|
| Study (all blocks) | ~22-30 days |
| Build (all checkpoints) | ~18-24 days |
| **Overlap (study + build interleaved)** | **~25-30 days (3.5-4.5 weeks)** |

> [!TIP]
> The study blocks and build checkpoints are **interleaved** in the week-by-week plan above. You study a block, then immediately build the corresponding component. This is faster than "learn everything first, then build."

> [!IMPORTANT]
> **The single most critical phase for Vani is AI Phase 11 (LLM Engineering) — specifically lessons 04 (Embeddings), 06 (RAG), and 07 (Advanced RAG).** These three lessons map directly to Vani's core pipeline: ingest → chunk → embed → search. If you're short on time, nail these first, then build the basic pipeline, and add MCP + parsers incrementally.

> [!NOTE]
> **Vani complements Karta perfectly.** Karta defines and runs agents. Vani provides the knowledge base those agents query via MCP. When you demo them together — a Karta agent calling Vani's MCP server to search ingested documents — you tell the story: *"I can build the agent AND the data pipeline it runs on."*
