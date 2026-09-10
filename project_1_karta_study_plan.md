# 🔥 Project 1: Karta — Agent-as-Code Framework
# Complete Study & Build Plan

> **Goal:** Build an open-source Python framework for defining, testing, versioning, and deploying conversational AI agents as code — targeting the Agent Engineer role at Sarvam AI.

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

Karta is NOT a chatbot. It's the **engineering layer** between raw LLM APIs and production deployment. Think of it as "Kubernetes for agents" — you define agents as YAML specs, test them with eval suites, version them in git, and deploy them with circuit breakers and fallback strategies.

### Core Components You're Building

| Component | What It Is |
|---|---|
| **Agent Definition Language** | YAML + Python DSL for declarative agent specs |
| **Scenario Coverage Matrix** | Declarative test case definitions (happy path, edge, adversarial) |
| **Built-in Eval Pipeline** | Accuracy, hallucination, latency, cost, regression detection |
| **Tool Integration Framework** | Agents calling external APIs, databases, webhooks |
| **Failure Mode Engine** | Circuit breakers, fallbacks, graceful degradation, token budgets |
| **Version Control & Diffing** | Git-diffable agent specs |
| **Indian Language Support** | Sarvam APIs for Hindi + English |
| **FSM Router** | Guided finite state machine for conversational flow |
| **Memory Manager** | Short-term (sliding window), long-term (vector), episodic (summary) |
| **Observability** | OpenTelemetry + Langfuse integration |

### Tech Stack

- **Runtime:** Python + LangGraph
- **Specs:** YAML + Pydantic
- **API:** FastAPI + asyncio
- **Memory:** Redis (session) + ChromaDB (long-term)
- **Observability:** OpenTelemetry + Langfuse
- **Indian Languages:** Sarvam APIs

---

## Backend Phases & Skills Required

These are the chapters from [Backend-from-first-Principle](file:///c:/Lab/Backend-from-first-Principle) you need to study.

### 🔴 Critical (Must Do Before Building)

| # | Chapter | Why You Need It for Karta | Specific Skills |
|---|---|---|---|
| **1** | [HTTP and CORS](file:///c:/Lab/Backend-from-first-Principle/1.HTTP-AND-CORS) | Karta's FastAPI server communicates via HTTP. You need to understand request/response cycles, headers, status codes, CORS for when agents are accessed from web UIs | HTTP methods, status codes, headers, CORS policy |
| **2** | [Routing in Backend](file:///c:/Lab/Backend-from-first-Principle/2.Routing-in-backend) | Karta exposes REST endpoints (`/agents/{id}/chat`, `/agents/{id}/eval`, etc.). Understanding route parameters, query params, path-based routing | Route handlers, path params, query params, route organization |
| **3** | [Serialization and Deserialization](file:///c:/Lab/Backend-from-first-Principle/3.Serialization-and-Deserialization-or-backend-engineers) | Agent specs are YAML → Pydantic models → JSON responses. Tool calls return JSON. Understanding how data crosses boundaries | JSON parsing, schema validation, Pydantic models, YAML loading |
| **5** | [Validations and Transformations](file:///c:/Lab/Backend-from-first-Principle/5.%20Validations%20and%20transformations%20for%20backend%20engineers) | Agent specs need validation (is the YAML valid? Are tool configs correct? Are guardrail thresholds reasonable?). Input sanitization for user messages | Pydantic validators, custom validation rules, data sanitization |
| **6** | [Controllers, Services, Repositories, Middlewares](file:///c:/Lab/Backend-from-first-Principle/6.controllers-services-repositories-middlewares-and-request-context) | Karta's codebase needs clean architecture. Separate API controllers from agent runtime logic from data access. Middleware for logging, auth, request context | Layered architecture, dependency injection, middleware chains, request context |
| **7** | [API Design — REST API](file:///c:/Lab/Backend-from-first-Principle/7.API-DESIGN-RestAPI) | Designing the Karta API surface: endpoints for agent CRUD, conversation management, eval triggers, version management | RESTful conventions, resource naming, versioned APIs, error responses |

### 🟡 Important (Need During Building)

| # | Chapter | Why You Need It for Karta | Specific Skills |
|---|---|---|---|
| **4** | [Authentication and Authorization](file:///c:/Lab/Backend-from-first-Principle/4.Authentication%20and%20authorization%20for%20backend%20engineers) | Multi-tenant agent deployments need auth. API key validation, tenant isolation | API keys, JWT basics, RBAC concepts |
| **8** | [Database with Backend](file:///c:/Lab/Backend-from-first-Principle/8.Database-with-backend) | Agent specs, eval results, conversation histories need persistent storage. Understanding SQL/NoSQL basics for ChromaDB and Redis integration | DB connections, queries, ORM basics, migrations |
| **9** | [Caching](file:///c:/Lab/Backend-from-first-Principle/9.Caching,%20the%20secret%20behind%20it%20all) | Redis for session state (short-term memory), caching LLM responses for eval determinism, caching embeddings | Redis basics, cache strategies (TTL, LRU), session storage |
| **12** | [Error Handling and Fault-Tolerant Systems](file:///c:/Lab/Backend-from-first-Principle/12.%20Error%20Handling%20and%20Building%20Fault%20Tolerant%20Systems) | The **Failure Mode Engine** is a core Karta feature. Circuit breakers, retry strategies, fallback logic, graceful degradation — this chapter is DIRECTLY applicable | Circuit breaker pattern, retry with backoff, fallback chains, error propagation |
| **14** | [Production-grade Configuration Management](file:///c:/Lab/Backend-from-first-Principle/14.Production-grade%20Configuration%20Management) | Karta loads agent configs from YAML files, environment variables, secrets for API keys (OpenAI, Sarvam, etc.) | Config loading, env vars, secrets management, config validation |
| **15** | [Logging, Monitoring, and Observability](file:///c:/Lab/Backend-from-first-Principle/15.Logging,%20Monitoring%20and%20Observability) | Karta integrates OpenTelemetry + Langfuse. Understanding structured logging, traces, spans, metrics is essential | Structured logging, OpenTelemetry concepts, trace propagation |

### 🟢 Nice to Have (Polish Phase)

| # | Chapter | Why You Need It for Karta | Specific Skills |
|---|---|---|---|
| **16** | [Graceful Shutdown](file:///c:/Lab/Backend-from-first-Principle/16.Graceful%20Shutdown) | When shutting down the Karta server, active agent conversations need to complete cleanly | Signal handlers, drain connections, cleanup |
| **20** | [Concurrency & Parallelism](file:///c:/Lab/Backend-from-first-Principle/20.Concurrency%20&%20Parallelism%20-%20IO%20Bound%20vs%20CPU%20Bound) | Agents make concurrent API calls (LLM + tools). Understanding asyncio deeply matters for FastAPI + agent runtime | asyncio, event loop, concurrent futures, IO-bound vs CPU-bound |
| **21** | [Containerization — Docker, K8s, CI/CD](file:///c:/Lab/Backend-from-first-Principle/21.Containerization-and-Deployment-Docker-Kubernetes-and-CICD) | Packaging Karta as a Docker container for deployment | Dockerfile, docker-compose, basic CI/CD |
| **22** | [Automated Testing](file:///c:/Lab/Backend-from-first-Principle/22.Automated-Testing-Unit-Integration-and-E2E) | Testing the Karta framework itself (unit tests for validators, integration tests for agent runtime) | pytest, fixtures, mocking, integration tests |
| **24** | [WebSockets and Real-Time Communication](file:///c:/Lab/Backend-from-first-Principle/24.Web-Sockets-And-Real-time-Communication-with-WebSockets) | Streaming agent responses to clients in real-time | WebSocket protocol, server-sent events, streaming |

---

## AI Engineering Phases & Skills Required

These are the phases from [ai-engineering-from-scratch](file:///c:/Lab/ai-engineering-from-scratch/phases) you need to study.

### 🔴 Critical — Foundations (Must Do First)

| Phase | Name | Why You Need It for Karta | Key Lessons |
|---|---|---|---|
| **Phase 00** | [Setup and Tooling](file:///c:/Lab/ai-engineering-from-scratch/phases/00-setup-and-tooling) | Dev environment, Python envs, Docker for AI, API key management | `01-dev-environment`, `04-apis-and-keys`, `06-python-environments`, `07-docker-for-ai` |
| **Phase 10** (partial) | [LLMs from Scratch](file:///c:/Lab/ai-engineering-from-scratch/phases/10-llms-from-scratch) | Understanding tokenizers is essential for token-budget guardrails. Understanding how LLMs work helps you design better agent specs | `01-tokenizers`, `02-building-a-tokenizer` (conceptual understanding of `04-pre-training-mini-gpt` helps) |

### 🔴 Critical — Core Agent Skills (The Heart of Karta)

| Phase | Name | Why You Need It for Karta | Key Lessons |
|---|---|---|---|
| **Phase 11** | [LLM Engineering](file:///c:/Lab/ai-engineering-from-scratch/phases/11-llm-engineering) | **This is THE most critical phase.** Prompt engineering, structured outputs, embeddings, RAG, function calling, guardrails, evaluation — every single one maps to a Karta feature | `01-prompt-engineering` → persona/system prompts, `02-few-shot-cot` → agent reasoning, `03-structured-outputs` → tool call parsing, `04-embeddings` → long-term memory, `06-rag` + `07-advanced-rag` → memory retrieval, `09-function-calling` → tool integration, `10-evaluation` → eval pipeline, `12-guardrails` → guardrail engine, `13-production-app` → production patterns, `14-model-context-protocol` → MCP awareness, `16-langgraph-state-machines` → FSM routing |
| **Phase 13** | [Tools and Protocols](file:///c:/Lab/ai-engineering-from-scratch/phases/13-tools-and-protocols) | Karta's Tool Integration Framework. Understanding function calling, tool schemas, MCP protocol (Vani will use this, Karta needs to integrate with it) | `01-the-tool-interface` → tool design, `02-function-calling-deep-dive` → tool call mechanics, `03-parallel-and-streaming-tool-calls` → concurrent tools, `04-structured-output` → parsing, `05-tool-schema-design` → schema engineering, `06-mcp-fundamentals` through `08-building-an-mcp-client` → MCP integration |
| **Phase 14** | [Agent Engineering](file:///c:/Lab/ai-engineering-from-scratch/phases/14-agent-engineering) | **This is the other most critical phase.** The agent loop, planning patterns, memory architectures, framework comparisons, failure modes, production runtimes, eval-driven development | `01-the-agent-loop` → core runtime, `02-rewoo-plan-and-execute` → planning, `03-reflexion-verbal-rl` → self-correction, `06-tool-use-and-function-calling` → tool execution, `07-memory-virtual-context-memgpt` → memory system, `08-memory-blocks-sleep-time-compute` → memory strategies, `09-hybrid-memory-mem0` → memory implementation, `12-anthropic-workflow-patterns` → design patterns, `13-langgraph-stateful-graphs` → LangGraph (your runtime!), `26-failure-modes-agentic` → failure mode engine, `27-prompt-injection-defense` → guardrails, `28-orchestration-patterns` → architecture, `29-production-runtimes` → production design, `30-eval-driven-agent-development` → eval pipeline |

### 🟡 Important — Deepening Skills

| Phase | Name | Why You Need It for Karta | Key Lessons |
|---|---|---|---|
| **Phase 05** (selected) | [NLP Foundations](file:///c:/Lab/ai-engineering-from-scratch/phases/05-nlp-foundations-to-advanced) | Understanding embeddings, semantic similarity (needed for eval matching), multilingual NLP (Hindi support) | `03-word-embeddings-word2vec` → embedding intuition, `22-embedding-models-deep-dive` → choosing embedding models, `18-multilingual-nlp` → Hindi support, `29-dialogue-state-tracking` → FSM routing |
| **Phase 18** (selected) | [Ethics, Safety, Alignment](file:///c:/Lab/ai-engineering-from-scratch/phases/18-ethics-safety-alignment) | Karta's guardrail engine: PII detection, hallucination checks, banned topics, prompt injection defense | `12-red-teaming-pair-automated-attacks` → adversarial testing, `15-indirect-prompt-injection` → prompt injection defense, `29-moderation-systems-openai-perspective-llamaguard` → content moderation |

### 🟢 Nice to Have — Advanced Topics

| Phase | Name | Why You Need It for Karta | Key Lessons |
|---|---|---|---|
| **Phase 06** (selected) | [Speech and Audio](file:///c:/Lab/ai-engineering-from-scratch/phases/06-speech-and-audio) | Only if implementing voice agent support using Sarvam's TTS | `07-text-to-speech` → TTS integration, `12-voice-assistant-pipeline` → voice pipeline |
| **Phase 17** (selected) | [Infrastructure & Production](file:///c:/Lab/ai-engineering-from-scratch/phases/17-infrastructure-and-production) | Production deployment of Karta agents | `13-llm-observability` → Langfuse setup, `20-shadow-canary-progressive` → deployment strategies |

---

## Skills You DON'T Need for This Project

These phases are **NOT required** for Karta. Don't waste time on them:

| Phase | Name | Why You Can Skip It |
|---|---|---|
| Phase 01 | Math Foundations | Not building ML models from scratch |
| Phase 02 | ML Fundamentals | Using pre-trained LLMs, not training |
| Phase 03 | Deep Learning Core | Same — using, not building |
| Phase 04 | Computer Vision | No vision in Karta |
| Phase 08 | Generative AI (images) | No image generation |
| Phase 09 | Reinforcement Learning | Not training agents with RL |
| Phase 10 (most of it) | LLMs from Scratch (advanced) | Don't need to build an LLM, just use one |
| Phase 12 | Multimodal AI | Text-only agents for now |
| Phase 15 | Autonomous Systems | Overkill for Karta v1 |
| Phase 16 | Multi-Agent & Swarms | Single agent framework |
| Backend #10 | Task Queues | Not needed for core Karta |
| Backend #11 | Elasticsearch | No full-text search in Karta |
| Backend #13 | gRPC | REST is sufficient for Karta v1 |
| Backend #17 | Backend Security (deep) | Basic auth is enough |
| Backend #18-19 | Scaling & Performance | Premature for v1 |
| Backend #23 | Message Brokers / Kafka | Not needed |

---

## Recommended Study Order (The Flow)

> **Strategy:** Learn → Build incrementally. Don't study everything first. Study a block, then build the corresponding Karta component.

```
📚 STUDY BLOCK 1: "Foundations"                          ⏱️ ~3-4 days
├── Backend #1: HTTP and CORS
├── Backend #2: Routing
├── Backend #3: Serialization/Deserialization
├── AI Phase 00: Setup and Tooling (selected lessons)
└── AI Phase 10: Tokenizers (01-02 only)
         │
         ▼
📚 STUDY BLOCK 2: "LLM Engineering Core"                ⏱️ ~5-7 days
├── AI Phase 11: Prompt Engineering (01)
├── AI Phase 11: Few-Shot, CoT (02)
├── AI Phase 11: Structured Outputs (03)
├── AI Phase 11: Embeddings (04)
├── AI Phase 11: Function Calling (09)
├── AI Phase 11: Guardrails (12)
└── AI Phase 11: LangGraph State Machines (16)
         │
         ▼
🔨 BUILD CHECKPOINT 1: "Hello Agent"                    ⏱️ ~2-3 days
│   Build: Basic agent that takes a YAML spec, loads it,
│   and responds to messages using an LLM. No tools, no
│   memory, no eval yet. Just YAML → Pydantic → LLM call.
│
         ▼
📚 STUDY BLOCK 3: "Backend Architecture"                ⏱️ ~3-4 days
├── Backend #5: Validations
├── Backend #6: Controllers/Services/Middlewares
├── Backend #7: API Design (REST)
├── Backend #14: Configuration Management
└── Backend #12: Error Handling & Fault Tolerance
         │
         ▼
📚 STUDY BLOCK 4: "Tools & Agent Engineering"           ⏱️ ~5-7 days
├── AI Phase 13: Tool Interface (01)
├── AI Phase 13: Function Calling Deep Dive (02)
├── AI Phase 13: Tool Schema Design (05)
├── AI Phase 14: The Agent Loop (01)
├── AI Phase 14: Tool Use & Function Calling (06)
├── AI Phase 14: Anthropic Workflow Patterns (12)
└── AI Phase 14: LangGraph Stateful Graphs (13)
         │
         ▼
🔨 BUILD CHECKPOINT 2: "Agent with Tools & FSM"         ⏱️ ~3-4 days
│   Build: Add FSM routing (state machine), tool
│   integration (mock APIs), and the failure mode engine
│   (circuit breakers, fallbacks, token budget).
│
         ▼
📚 STUDY BLOCK 5: "Memory & RAG"                        ⏱️ ~3-4 days
├── AI Phase 11: RAG (06)
├── AI Phase 11: Advanced RAG (07)
├── AI Phase 14: Memory — MemGPT (07)
├── AI Phase 14: Memory Blocks (08)
├── AI Phase 14: Hybrid Memory — Mem0 (09)
└── Backend #9: Caching (Redis)
         │
         ▼
📚 STUDY BLOCK 6: "Data & Storage"                      ⏱️ ~2-3 days
├── Backend #8: Database with Backend
└── AI Phase 05: Embedding Models Deep Dive (22)
         │
         ▼
🔨 BUILD CHECKPOINT 3: "Agent with Memory"              ⏱️ ~3-4 days
│   Build: Add memory manager — short-term (sliding
│   window, Redis), long-term (ChromaDB vector store),
│   episodic (session summary). Wire up embeddings.
│
         ▼
📚 STUDY BLOCK 7: "Evaluation & Testing"                ⏱️ ~3-4 days
├── AI Phase 11: Evaluation (10)
├── AI Phase 14: Eval-Driven Agent Development (30)
├── AI Phase 14: Failure Modes Agentic (26)
├── Backend #22: Automated Testing
└── AI Phase 18: Red-teaming (12), Prompt Injection (15)
         │
         ▼
🔨 BUILD CHECKPOINT 4: "Eval Pipeline"                  ⏱️ ~3-4 days
│   Build: YAML eval suite definition, scenario runner,
│   semantic assertion matching (embedding similarity
│   instead of exact match), statistical pass criteria,
│   regression detection.
│
         ▼
📚 STUDY BLOCK 8: "Production Polish"                   ⏱️ ~2-3 days
├── Backend #15: Logging, Monitoring, Observability
├── Backend #4: Authentication & Authorization
├── Backend #20: Concurrency (asyncio deep dive)
├── AI Phase 14: Production Runtimes (29)
└── AI Phase 05: Multilingual NLP (18) — Hindi support
         │
         ▼
🔨 BUILD CHECKPOINT 5: "Production-Ready Karta"         ⏱️ ~3-4 days
│   Build: Add observability (OpenTelemetry + Langfuse),
│   Sarvam API integration (translate, TTS), FastAPI
│   endpoints, version diffing, documentation,
│   README with architecture diagram.
│
         ▼
📚 STUDY BLOCK 9: "Deployment" (Optional)               ⏱️ ~1-2 days
├── Backend #21: Docker, CI/CD
├── Backend #16: Graceful Shutdown
└── AI Phase 17: LLM Observability (13)
         │
         ▼
🔨 FINAL: "Ship It"                                     ⏱️ ~2-3 days
│   Docker packaging, GitHub README, Quick Start guide,
│   demo recording, Design Decisions doc.
```

---

## Week-by-Week Execution Plan

### Week 1: Foundations & First LLM Call
**Study:** Blocks 1 + 2 (Backend HTTP/Routing/Serialization + LLM Engineering Core)

| Day | Activity | Output |
|---|---|---|
| Day 1 | Backend #1 (HTTP/CORS) + Backend #2 (Routing) | Notes: HTTP lifecycle, route patterns |
| Day 2 | Backend #3 (Serialization) + AI Phase 00 (Setup) | Working dev environment, API keys configured |
| Day 3 | AI Phase 10: Tokenizers (01-02) | Understand tokenization, token counting |
| Day 4 | AI Phase 11: Prompt Engineering (01) + Few-Shot (02) | First LLM API calls, prompt templates |
| Day 5 | AI Phase 11: Structured Outputs (03) + Embeddings (04) | JSON mode outputs, embedding vectors |
| Day 6 | AI Phase 11: Function Calling (09) + Guardrails (12) | Tool call parsing, safety checks |
| Day 7 | AI Phase 11: LangGraph State Machines (16) | Basic state machine with LangGraph |

**Milestone:** Can make LLM API calls, parse structured outputs, understand state machines.

---

### Week 2: First Working Agent + Tools
**Study:** Block 3 (Backend Architecture) + Block 4 (Tools & Agent Engineering)
**Build:** Checkpoints 1 + 2

| Day | Activity | Output |
|---|---|---|
| Day 8 | Backend #5 (Validations) + #6 (Architecture Patterns) | Understand layered architecture |
| Day 9 | Backend #7 (API Design) + #12 (Error Handling) | REST API design, circuit breaker pattern |
| Day 10 | Backend #14 (Config Management) | YAML config loading, env vars |
| Day 11 | AI Phase 13: Tool Interface (01) + Function Calling Deep Dive (02) | Tool schema design |
| Day 12 | AI Phase 14: Agent Loop (01) + Tool Use (06) | Core agent loop implementation |
| Day 13 | AI Phase 14: Anthropic Patterns (12) + LangGraph (13) | Production agent patterns |
| Day 14 | **BUILD:** Checkpoint 1 + 2 — basic agent with YAML spec, tools, FSM | Working agent that routes through states |

**Milestone:** Agent that loads from YAML, routes through FSM, calls tools, has circuit breakers.

---

### Week 3: Memory System + Eval Pipeline
**Study:** Blocks 5 + 6 + 7 (Memory, Data, Evaluation)
**Build:** Checkpoints 3 + 4

| Day | Activity | Output |
|---|---|---|
| Day 15 | AI Phase 11: RAG (06) + Advanced RAG (07) | Vector search, chunking, retrieval |
| Day 16 | AI Phase 14: Memory systems (07, 08, 09) | MemGPT, memory blocks, Mem0 patterns |
| Day 17 | Backend #8 (Database) + Backend #9 (Caching/Redis) | Redis session store, DB basics |
| Day 18 | AI Phase 05: Embedding Models (22) | Choosing embeddings, similarity metrics |
| Day 19 | **BUILD:** Checkpoint 3 — memory manager | Sliding window + ChromaDB + session summary |
| Day 20 | AI Phase 11: Evaluation (10) + Phase 14: Eval-Driven Dev (30) | Eval framework design |
| Day 21 | AI Phase 14: Failure Modes (26) + AI Phase 18: Red-teaming (12) + Backend #22 (Testing) | Adversarial test cases, pytest |

**Milestone:** Agent with full memory + eval pipeline that detects regressions.

---

### Week 4: Production Polish & Ship
**Study:** Blocks 8 + 9 (Production, Deployment)
**Build:** Checkpoints 4 + 5 + Final

| Day | Activity | Output |
|---|---|---|
| Day 22 | **BUILD:** Checkpoint 4 — eval suite runner | YAML eval definitions, semantic matching |
| Day 23 | Backend #15 (Observability) + AI Phase 14: Production (29) | Structured logging, traces |
| Day 24 | Backend #4 (Auth) + Backend #20 (Concurrency/asyncio) | API keys, async agent runtime |
| Day 25 | AI Phase 05: Multilingual NLP (18) + Sarvam API integration | Hindi agent support |
| Day 26 | **BUILD:** Checkpoint 5 — Langfuse + FastAPI endpoints + versioning | Observable, API-accessible agents |
| Day 27 | Backend #21 (Docker) + Backend #16 (Graceful Shutdown) | Containerized deployment |
| Day 28 | **FINAL:** README, Quick Start, architecture diagram, demo recording | Ship-ready GitHub repo |

**Milestone:** Production-ready Karta framework on GitHub.

---

## Key Deliverables Checklist

When you're done, your repo should have:

- [ ] **README.md** with architecture diagram (ASCII or Mermaid)
- [ ] **Quick Start** section — run a demo agent in < 5 minutes
- [ ] **Agent spec example** (like the `banking_kyc_agent` in the battleplan)
- [ ] **Eval suite example** with regression tests
- [ ] **At least 2 demo agents:**
  - Banking KYC agent (Hindi + English)
  - Customer support agent (with tool calls)
- [ ] **Eval results** showing accuracy, latency, cost metrics
- [ ] **Design Decisions** doc explaining WHY you chose:
  - YAML over pure Python for specs
  - Guided FSM over free-form routing
  - Semantic matching over exact matching in evals
  - LangGraph as the runtime
- [ ] **Docker support** for containerized deployment
- [ ] **CI/CD** — GitHub Actions running eval suite on every push

---

## Total Time Estimate

| Activity | Time |
|---|---|
| Study (all blocks) | ~25-35 days |
| Build (all checkpoints) | ~15-20 days |
| **Overlap (study + build interleaved)** | **~28-35 days (4-5 weeks)** |

> [!TIP]
> The study blocks and build checkpoints are **interleaved** in the week-by-week plan above. You study a block, then immediately build the corresponding component. This is faster than "learn everything first, then build."

> [!IMPORTANT]
> **The single most important phase for Karta is Phase 14 (Agent Engineering).** If you're short on time, prioritize Phase 11 (LLM Engineering) and Phase 14 (Agent Engineering) above everything else. You can learn backend patterns as you code.
