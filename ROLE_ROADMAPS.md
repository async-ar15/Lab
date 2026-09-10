# AI Engineer Roadmap

> **Goal**: Build, deploy, and operate production AI systems. You USE models — you don't invent new architectures.
>
> **Estimated total**: ~180–220 hours (vs ~342 hours for the full curriculum)
>
> **Mindset**: Get to shipping fast. Understand enough theory to debug, not enough to publish a paper.

---

## Recommended Learning Order

Follow this sequence. Each tier builds on the previous one.

```
Tier 1 — Foundations (must do first)
  Phase 0 → Phase 1 (partial) → Phase 2 (partial) → Phase 3 (partial)

Tier 2 — Core Engineering (the meat of your job)
  Phase 5 (partial) → Phase 7 (partial) → Phase 11 → Phase 13 → Phase 14

Tier 3 — Production & Deployment (what separates you from hobbyists)
  Phase 17 → Phase 15 (partial) → Phase 18 (partial)

Tier 4 — Capstone Projects (prove you can ship)
  Phase 19 (selected projects)

Tier 5 — Specialization (pick ONE based on your niche)
  Phase 10 (partial) OR Phase 12 (partial) OR Phase 6 (partial)
```

---

## Tier 1 — Foundations (~40 hours)

### Phase 0: Setup & Tooling — ✅ ALL 12 lessons

> Every single lesson here is essential. This is your dev environment.

| # | Lesson | Priority |
|:-:|--------|:--------:|
| 01 | Dev Environment | 🔴 Must |
| 02 | Git & Collaboration | 🔴 Must |
| 03 | GPU Setup & Cloud | 🔴 Must |
| 04 | APIs & Keys | 🔴 Must |
| 05 | Jupyter Notebooks | 🟡 Useful |
| 06 | Python Environments | 🔴 Must |
| 07 | Docker for AI | 🔴 Must |
| 08 | Editor Setup | 🟡 Useful |
| 09 | Data Management | 🔴 Must |
| 10 | Terminal & Shell | 🟡 Useful |
| 11 | Linux for AI | 🟡 Useful |
| 12 | Debugging & Profiling | 🔴 Must |

---

### Phase 1: Math Foundations — ⚡ 8 of 22 lessons

> You need the intuition, not the proofs. Know enough to understand loss curves, embeddings, and why your model is diverging.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | Linear Algebra Intuition | 🔴 Must | Vectors/matrices are everywhere in AI |
| 02 | Vectors, Matrices & Operations | 🔴 Must | You'll read model code daily |
| 04 | Calculus for ML: Derivatives & Gradients | 🔴 Must | Understand backprop & training |
| 06 | Probability & Distributions | 🔴 Must | Core to every ML model |
| 08 | Optimization: Gradient Descent Family | 🔴 Must | Why Adam? Why learning rate matters? |
| 09 | Information Theory: Entropy, KL Divergence | 🟡 Useful | Helps understand loss functions |
| 10 | Dimensionality Reduction: PCA, t-SNE, UMAP | 🟡 Useful | Debugging embeddings |
| 14 | Norms & Distances | 🟡 Useful | Vector search relies on this |

**Skip for now**: Lessons 03, 05, 07, 11-13, 15-22 (eigenvalues, autodiff internals, SVD, tensor ops, numerical stability, sampling methods, linear systems, convex optimization, complex numbers, Fourier, graph theory, stochastic processes — these are researcher territory).

---

### Phase 2: ML Fundamentals — ⚡ 10 of 18 lessons

> Classical ML still powers most production systems. Know the fundamentals, skip the from-scratch math.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | What Is Machine Learning | 🔴 Must | Mental model |
| 02 | Linear Regression from Scratch | 🔴 Must | Foundation |
| 03 | Logistic Regression & Classification | 🔴 Must | Foundation |
| 04 | Decision Trees & Random Forests | 🟡 Useful | Still used in production |
| 07 | Unsupervised Learning: K-Means, DBSCAN | 🟡 Useful | Clustering use cases |
| 08 | Feature Engineering & Selection | 🔴 Must | Critical production skill |
| 09 | Model Evaluation: Metrics, Cross-Validation | 🔴 Must | You'll do this constantly |
| 10 | Bias, Variance & the Learning Curve | 🔴 Must | Debugging models |
| 12 | Hyperparameter Tuning | 🟡 Useful | Production tuning |
| 13 | ML Pipelines & Experiment Tracking | 🔴 Must | MLOps is your bread and butter |

**Skip for now**: Lessons 05-06, 11, 14-18 (SVMs, KNN, ensemble deep-dives, Naive Bayes, time series, anomaly detection, imbalanced data — learn when you encounter them on the job).

---

### Phase 3: Deep Learning Core — ⚡ 7 of 13 lessons

> Understand the building blocks. You don't need to write backprop from raw numpy for your job, but you need to understand what PyTorch is doing.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | The Perceptron | 🟡 Useful | Historical context |
| 02 | Multi-Layer Networks & Forward Pass | 🔴 Must | Core concept |
| 03 | Backpropagation from Scratch | 🟡 Useful | Understand, don't memorize |
| 04 | Activation Functions | 🔴 Must | You'll configure these |
| 05 | Loss Functions | 🔴 Must | You'll choose these |
| 06 | Optimizers: SGD, Momentum, Adam, AdamW | 🔴 Must | AdamW is your default |
| 11 | Introduction to PyTorch | 🔴 Must | Your primary framework |
| 13 | Debugging Neural Networks | 🔴 Must | Production debugging |

**Skip for now**: Lessons 07-10, 12 (regularization deep-dives, weight init math, LR schedules from scratch, building your own framework, JAX — researcher territory).

---

## Tier 2 — Core Engineering (~80 hours)

> This is where AI Engineer diverges completely from AI Researcher. This is YOUR domain.

### Phase 5: NLP Foundations — ⚡ 10 of 29 lessons

> Understand text processing and embeddings. Skip the from-scratch NLP models — you'll use pretrained ones.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | Text Processing: Tokenization, Stemming, Lemmatization | 🔴 Must | Daily work |
| 02 | Bag of Words, TF-IDF | 🟡 Useful | Baseline methods |
| 03 | Word Embeddings: Word2Vec from Scratch | 🟡 Useful | Understand embeddings |
| 05 | Sentiment Analysis | 🟡 Useful | Common use case |
| 10 | Attention Mechanism — The Breakthrough | 🔴 Must | Core to everything modern |
| 19 | Subword Tokenization: BPE, WordPiece | 🔴 Must | How LLMs process text |
| 20 | Structured Outputs & Constrained Decoding | 🔴 Must | Production APIs |
| 22 | Embedding Models Deep Dive | 🔴 Must | RAG depends on this |
| 23 | Chunking Strategies for RAG | 🔴 Must | Core RAG skill |
| 27 | LLM Evaluation: RAGAS, DeepEval, G-Eval | 🔴 Must | You must evaluate your systems |

**Skip**: Lessons 04, 06-09, 11-18, 21, 24-26, 28-29 (GloVe, NER, POS tagging, CNNs for text, seq2seq, translation, summarization, QA, topic modeling, chatbots, multilingual NLP, NLI, coreference, entity linking, long-context eval, dialogue — researcher/specialist territory).

---

### Phase 7: Transformers Deep Dive — ⚡ 6 of 16 lessons

> Understand the architecture you'll use every day. Don't build it from scratch.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | Why Transformers: Problems with RNNs | 🔴 Must | Context |
| 02 | Self-Attention from Scratch | 🔴 Must | Core understanding |
| 03 | Multi-Head Attention | 🔴 Must | What's inside every LLM |
| 04 | Positional Encoding: Sinusoidal, RoPE, ALiBi | 🟡 Useful | Helps with context window understanding |
| 07 | GPT — Causal Language Modeling | 🔴 Must | How GPT-style models work |
| 12 | KV Cache, Flash Attention & Inference Optimization | 🔴 Must | Production performance |

**Skip**: Lessons 05-06, 08-11, 13-16 (full transformer build, BERT, T5/BART, ViT, audio transformers, MoE, scaling laws, building from scratch, attention variants, speculative decoding — researcher territory).

---

### Phase 11: LLM Engineering — ✅ ALL 17 lessons 🔥

> **THIS IS YOUR CORE PHASE.** Every single lesson here is essential for an AI Engineer. This is the phase that defines your role.

| # | Lesson | Priority |
|:-:|--------|:--------:|
| 01 | Prompt Engineering: Techniques & Patterns | 🔴 Must |
| 02 | Few-Shot, CoT, Tree-of-Thought | 🔴 Must |
| 03 | Structured Outputs | 🔴 Must |
| 04 | Embeddings & Vector Representations | 🔴 Must |
| 05 | Context Engineering | 🔴 Must |
| 06 | RAG: Retrieval-Augmented Generation | 🔴 Must |
| 07 | Advanced RAG: Chunking, Reranking | 🔴 Must |
| 08 | Fine-Tuning with LoRA & QLoRA | 🔴 Must |
| 09 | Function Calling & Tool Use | 🔴 Must |
| 10 | Evaluation & Testing | 🔴 Must |
| 11 | Caching, Rate Limiting & Cost | 🔴 Must |
| 12 | Guardrails & Safety | 🔴 Must |
| 13 | Building a Production LLM App | 🔴 Must |
| 14 | Model Context Protocol (MCP) | 🔴 Must |
| 15 | Prompt Caching & Context Caching | 🔴 Must |
| 16 | Agent State Machines | 🔴 Must |
| 17 | Agent Framework Tradeoffs | 🔴 Must |

---

### Phase 13: Tools & Protocols — ⚡ 20 of 31 lessons 🔥

> Another core phase. MCP, function calling, tool design — this is what AI Engineers build.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | The Tool Interface | 🔴 Must | Foundation |
| 02 | Function Calling Deep Dive | 🔴 Must | Core skill |
| 03 | Parallel and Streaming Tool Calls | 🔴 Must | Production patterns |
| 04 | Structured Output | 🔴 Must | API design |
| 05 | Tool Schema Design | 🔴 Must | Engineering craft |
| 06 | MCP Fundamentals | 🔴 Must | Industry standard |
| 07 | Building an MCP Server | 🔴 Must | You'll build these |
| 08 | Building an MCP Client | 🔴 Must | You'll build these |
| 09 | MCP Transports | 🔴 Must | stdio, HTTP |
| 10 | MCP Resources and Prompts | 🟡 Useful | Advanced MCP |
| 14 | MCP Apps | 🟡 Useful | Advanced MCP |
| 15 | MCP Security | 🔴 Must | Production security |
| 16 | MCP Authorization | 🔴 Must | OAuth, auth flows |
| 18 | MCP Auth in Production | 🔴 Must | Real-world auth |
| 19 | A2A Protocol | 🔴 Must | Agent-to-Agent |
| 20 | OpenTelemetry GenAI | 🔴 Must | Observability |
| 21 | LLM Routing Layer | 🔴 Must | Cost optimization |
| 22 | Agent Skills: Portable Contract | 🟡 Useful | Advanced |
| 23 | Capstone: Tool Ecosystem | 🟡 Useful | Practice |
| 28 | MCP Tool Contracts and Content | 🟡 Useful | Advanced |

**Skip**: Lessons 11-13, 17, 24-27, 29-31 (MCP sampling, elicitation, async tasks, gateways, skill deep-dives, reliability internals, registry supply chain, conformance engineering — deep specialist territory).

---

### Phase 14: Agent Engineering — ⚡ 30 of 54 lessons 🔥

> Your third core phase. Agents are the product. Focus on building and shipping, not academic patterns.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | The Agent Loop | 🔴 Must | Foundation of everything |
| 02 | ReWOO and Plan-and-Execute | 🔴 Must | Key pattern |
| 05 | Self-Refine and CRITIC | 🟡 Useful | Quality loops |
| 06 | Tool Use and Function Calling | 🔴 Must | Core skill |
| 07 | Agent Memory — Virtual Context | 🔴 Must | Stateful agents |
| 08 | Memory Blocks and Sleep-Time Compute | 🟡 Useful | Advanced memory |
| 09 | Hybrid Memory — Vector + Graph + KV | 🔴 Must | Production memory |
| 12 | Anthropic's Workflow Patterns | 🔴 Must | Industry-standard patterns |
| 13 | Stateful Graph Orchestration | 🔴 Must | LangGraph |
| 14 | The Actor Model for Agents | 🟡 Useful | AutoGen pattern |
| 15 | Role-Based Agent Teams | 🟡 Useful | CrewAI pattern |
| 16 | OpenAI Agents SDK | 🔴 Must | Major SDK |
| 17 | The Harness as a Library | 🔴 Must | Claude SDK |
| 18 | Production Agent Runtimes | 🔴 Must | Agno, Mastra |
| 21 | Computer Use Agents | 🟡 Useful | Growing area |
| 22 | Voice Agents — Pipecat and LiveKit | 🟡 Useful | Growing area |
| 23 | OpenTelemetry GenAI Conventions | 🔴 Must | Observability |
| 24 | Agent Observability Platforms | 🔴 Must | Langfuse, Phoenix |
| 26 | Failure Modes — Why Agents Break | 🔴 Must | Debugging |
| 27 | Prompt Injection and PVE Defense | 🔴 Must | Security |
| 28 | Orchestration Patterns | 🔴 Must | Architecture |
| 29 | Production Runtimes | 🔴 Must | Queue, Event, Cron |
| 30 | Eval-Driven Agent Development | 🔴 Must | Testing |
| 31 | Agent Workbench: Why Models Fail | 🔴 Must | Debugging |
| 32 | The Minimal Agent Workbench | 🔴 Must | Tooling |
| 36 | Scope Contracts and Task Boundaries | 🟡 Useful | Design |
| 37 | Runtime Feedback Loops | 🟡 Useful | Design |
| 38 | Verification Gates | 🔴 Must | Quality control |
| 43 | Frame the Task Before Code | 🟡 Useful | Methodology |
| 44 | Build an Evidence-Backed Plan | 🟡 Useful | Methodology |

**Skip**: Lessons 03-04, 10-11, 19-20, 25, 33-35, 39-42, 45-54 (Reflexion, Tree of Thoughts, skill libraries, HTN planning, benchmarks, multi-agent debate, instructions-as-constraints, init scripts, repo memory, reviewer agent, multi-session handoff, workbench capstone, product judgment lessons — researcher/advanced territory).

---

## Tier 3 — Production & Deployment (~40 hours)

### Phase 17: Infrastructure & Production — ✅ ALL 28 lessons 🔥

> **Your fourth core phase.** If Phase 11 is about building LLM apps, Phase 17 is about shipping and operating them. This is what makes you a *senior* AI Engineer.

Every single lesson here is relevant. Prioritize:

| Priority | Lessons |
|:--------:|---------|
| 🔴 Must (do first) | 01 (Managed Platforms), 02 (Inference Economics), 04 (Serving Internals), 08 (Inference Metrics), 09 (Quantization), 13 (Observability), 14 (Prompt Caching), 16 (Model Routing), 19 (AI Gateways), 20 (Shadow/Canary Deploy), 22 (Load Testing), 25 (Security), 26 (Compliance), 27 (FinOps) |
| 🟡 Do second | 03 (GPU K8s), 05-07 (Speculative Decoding, RadixAttention, TensorRT), 10-12 (Cold Start, Multi-Region, Edge), 15 (Batch APIs), 17-18 (Disaggregated Serving, KV Offloading), 21 (A/B Testing), 23-24 (SRE, Chaos Engineering), 28 (Self-Hosted Selection) |

---

### Phase 15: Autonomous Systems — ⚡ 8 of 22 lessons

> Focus on the safety and operational side, not the research papers.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 01 | From Chatbots to Long-Horizon Agents | 🔴 Must | Context |
| 09 | Autonomous Coding Agent Landscape | 🔴 Must | Industry knowledge |
| 10 | Permission Modes for Autonomous Agents | 🔴 Must | Safety design |
| 11 | Browser Agents and Indirect Prompt Injection | 🔴 Must | Security |
| 12 | Durable Execution for Long-Running Agents | 🔴 Must | Production |
| 13 | Action Budgets, Iteration Caps, Cost Governors | 🔴 Must | Cost control |
| 14 | Kill Switches, Circuit Breakers, Canary Tokens | 🔴 Must | Safety |
| 15 | HITL: Propose-Then-Commit | 🔴 Must | Human oversight |

**Skip**: Lessons 02-08, 16-22 (STaR, AlphaEvolve, Darwin Gödel Machine, AI Scientist, Automated Alignment, Recursive Self-Improvement, Bounded Self-Improvement, Checkpoints, Constitutional AI theory, Llama Guard, RSP, Preparedness Framework, METR, CAIS — pure researcher territory).

---

### Phase 18: Ethics, Safety & Alignment — ⚡ 8 of 30 lessons

> Know enough to build responsibly. Leave the alignment research to researchers.

| # | Lesson | Priority | Why |
|:-:|--------|:--------:|-----|
| 12 | Red-Teaming: PAIR & Automated Attacks | 🔴 Must | Test your systems |
| 15 | Indirect Prompt Injection | 🔴 Must | #1 production threat |
| 16 | Red-Team Tooling: Garak, Llama Guard, PyRIT | 🔴 Must | Your toolkit |
| 20 | Bias & Representational Harm | 🔴 Must | Responsible AI |
| 23 | Watermarking: SynthID, Stable Signature | 🟡 Useful | Emerging requirement |
| 24 | Regulatory Frameworks: EU, US, UK, Korea | 🔴 Must | Compliance |
| 26 | Model, System & Dataset Cards | 🔴 Must | Documentation standard |
| 29 | Moderation Systems | 🔴 Must | Production safety |

---

## Tier 4 — Capstone Projects (pick 3-5)

### Phase 19: Capstone Projects — ⚡ Selected projects

**Must-do for AI Engineers:**

| # | Project | Why |
|:-:|---------|-----|
| 02 | RAG over Codebase | Core RAG skills |
| 06 | DevOps Troubleshooting Agent | Real-world agent |
| 08 | Production RAG Chatbot (Regulated Vertical) | Production-grade |
| 11 | LLM Observability & Eval Dashboard | Operational excellence |
| 13 | Stateless MCP Server with Registry | MCP mastery |
| 16 | GitHub Issue-to-PR Autonomous Agent | End-to-end agent |

**Deep-build tracks for Engineers:**

| Track | Projects | Why |
|-------|----------|-----|
| A. Agent harness | 20-29 | Build your own agent framework |
| F. Advanced RAG | 64-69 | Master RAG end-to-end |
| G. Eval framework | 70-75 | Build evaluation systems |
| I. Safety harness | 82-87 | Build safety gates |

---

## Tier 5 — Specialization (pick ONE based on your niche)

### Option A: If you work with LLM internals — Phase 10 (partial)

| # | Lesson | Why |
|:-:|--------|-----|
| 01 | Tokenizers: BPE, WordPiece | Understand tokenization |
| 06 | Instruction Tuning — SFT | Fine-tuning |
| 08 | DPO | Preference optimization |
| 11 | Quantization: INT8, GPTQ, AWQ, GGUF | Model compression |
| 12 | Inference Optimization | Speed |
| 14 | Open Models: Architecture Walkthroughs | Know what you're deploying |

### Option B: If you work with multimodal — Phase 12 (partial)

| # | Lesson | Why |
|:-:|--------|-----|
| 02 | CLIP and Contrastive Pretraining | Vision-language |
| 05 | LLaVA and Visual Instruction Tuning | VLMs |
| 22 | Document and Diagram Understanding | Enterprise use case |
| 23 | ColPali Vision-Native Document RAG | Cutting edge |
| 24 | Multimodal RAG | Production multimodal |
| 25 | Multimodal Agents and Computer-Use | Capstone |

### Option C: If you work with voice — Phase 6 (partial)

| # | Lesson | Why |
|:-:|--------|-----|
| 04 | Speech Recognition (ASR) | Input pipeline |
| 05 | Whisper | Industry standard |
| 07 | Text-to-Speech (TTS) | Output pipeline |
| 11 | Real-Time Audio Processing | Production |
| 12 | Build a Voice Assistant Pipeline | End-to-end |

---

## ⚠️ Gaps — What This Curriculum LACKS for AI Engineers

The following are critical for a professional AI Engineer role but are NOT covered or barely touched in this curriculum. You need to learn these from other sources.

| Gap | Why It Matters | Where to Learn |
|-----|----------------|----------------|
| **Data Engineering** (Spark, Airflow, dbt, data lakes) | You'll build data pipelines before any model runs | DataTalksClub DE Zoomcamp (free) |
| **Vector Databases deep-dive** (Pinecone, Weaviate, Qdrant, pgvector architecture) | RAG systems need production vector DBs | Official docs + Pinecone learning center |
| **Cloud Architecture** (AWS/GCP/Azure AI services, Terraform, IaC) | Real deployments live in the cloud | Cloud provider certifications |
| **CI/CD for ML** (GitHub Actions for models, model registries, MLflow) | Shipping models needs automation | Made With ML by Goku Mohandas |
| **API Design & Backend** (FastAPI, authentication, rate limiting at scale) | Your AI lives behind an API | FastAPI docs + real projects |
| **Cost Engineering** (token budgets, spend tracking, multi-tenant billing) | Covered lightly in Phase 17, needs more depth | Learn on the job |
| **Data Labeling & Annotation** (Label Studio, RLHF data collection) | Fine-tuning requires labeled data | Practical experience |
| **Frontend for AI** (Streaming UIs, chat interfaces, Vercel AI SDK) | Users interact through UIs | Vercel AI SDK docs |
| **Database Knowledge** (PostgreSQL, Redis, message queues) | State management for agents | Standard backend resources |
| **System Design for AI** (latency budgets, caching layers, fallback chains) | Interviews & production architecture | AI system design resources |

---

## Summary: AI Engineer at a Glance

```
ESSENTIAL (do all lessons):
  Phase 0  — Setup & Tooling           (12 lessons, ~18 hrs)
  Phase 11 — LLM Engineering           (17 lessons, ~25 hrs)  ← YOUR CORE
  Phase 17 — Infrastructure & Prod     (28 lessons, ~42 hrs)  ← YOUR CORE

ESSENTIAL (do selected lessons):
  Phase 1  — Math Foundations           (8 of 22,   ~12 hrs)
  Phase 2  — ML Fundamentals           (10 of 18,  ~15 hrs)
  Phase 3  — Deep Learning Core        (7 of 13,   ~10 hrs)
  Phase 5  — NLP Foundations            (10 of 29,  ~15 hrs)
  Phase 7  — Transformers              (6 of 16,   ~9 hrs)
  Phase 13 — Tools & Protocols         (20 of 31,  ~30 hrs)  ← YOUR CORE
  Phase 14 — Agent Engineering         (30 of 54,  ~45 hrs)  ← YOUR CORE
  Phase 15 — Autonomous Systems        (8 of 22,   ~12 hrs)
  Phase 18 — Ethics & Safety           (8 of 30,   ~12 hrs)

SKIP ENTIRELY:
  Phase 4  — Computer Vision            (researcher)
  Phase 6  — Speech & Audio             (specialist — see Tier 5)
  Phase 8  — Generative AI              (researcher)
  Phase 9  — Reinforcement Learning     (researcher)
  Phase 10 — LLMs from Scratch          (researcher — see Tier 5)
  Phase 12 — Multimodal AI              (researcher — see Tier 5)
  Phase 16 — Multi-Agent & Swarms       (advanced/researcher)
```

---
---

## AI Researcher Roadmap

> **Goal**: Understand, design, and advance AI at the theoretical level. You BUILD new models, architectures, training methods, and publish papers.
>
> **Estimated total**: ~300–342 hours (nearly the full curriculum)
>
> **Mindset**: Depth over breadth. Derive everything from first principles. Understand every equation.

---

## Recommended Learning Order

```
Tier 1 — Mathematical & Theoretical Foundations (non-negotiable)
  Phase 0 → Phase 1 (ALL) → Phase 2 (ALL) → Phase 3 (ALL)

Tier 2 — Domain Mastery (go deep in at least one)
  Phase 4 (Vision) AND/OR Phase 5 (NLP) AND/OR Phase 6 (Speech)

Tier 3 — Architecture & Training (the heart of research)
  Phase 7 (ALL) → Phase 8 (ALL) → Phase 9 (ALL) → Phase 10 (ALL)

Tier 4 — Frontiers
  Phase 12 (Multimodal) → Phase 15 (Autonomous) → Phase 16 (Multi-Agent)
  → Phase 18 (Ethics & Alignment)

Tier 5 — Applied Context (lighter coverage)
  Phase 11 (partial) → Phase 13 (partial) → Phase 14 (partial) → Phase 17 (partial)

Tier 6 — Capstone
  Phase 19 (selected research-oriented projects)
```

---

## Tier 1 — Mathematical & Theoretical Foundations (~80 hours)

### Phase 0: Setup & Tooling — ✅ ALL 12 lessons

Same as AI Engineer. You need a working dev environment.

---

### Phase 1: Math Foundations — ✅ ALL 22 lessons 🔥

> **This is non-negotiable for researchers.** Every single lesson matters. You cannot skip SVD, Fourier transforms, or stochastic processes — they show up in papers constantly.

| # | Lesson | Priority |
|:-:|--------|:--------:|
| 01-22 | All lessons | 🔴 Must |

Key lessons that separate researchers from engineers:
- **03**: Matrix Transformations & Eigenvalues — PCA, spectral methods
- **05**: Chain Rule & Automatic Differentiation — you'll implement custom grad functions
- **07**: Bayes' Theorem & Statistical Thinking — Bayesian ML
- **11**: Singular Value Decomposition — LoRA is literally SVD
- **13**: Numerical Stability — training at scale requires this
- **16**: Sampling Methods — diffusion models, MCMC
- **18**: Convex Optimization — loss landscape analysis
- **19**: Complex Numbers — Fourier, signal processing
- **20**: The Fourier Transform — audio, vision, positional encodings
- **21**: Graph Theory — GNNs, knowledge graphs
- **22**: Stochastic Processes — diffusion, noise schedules

---

### Phase 2: ML Fundamentals — ✅ ALL 18 lessons

> Researchers must understand every classical method. Your baselines come from here.

All 18 lessons are relevant. The lessons engineers skip (SVMs, KNN, ensemble deep-dives, Naive Bayes, time series, anomaly detection, imbalanced data, feature selection) are all important for you — you need to understand the full landscape to design novel methods.

---

### Phase 3: Deep Learning Core — ✅ ALL 13 lessons 🔥

> **You must build every component from scratch.** This is the difference between using PyTorch and understanding PyTorch.

Critical researcher-only lessons:
- **07**: Regularization — Dropout, Weight Decay, BatchNorm (understand the math)
- **08**: Weight Initialization & Training Stability (He, Xavier, why they work)
- **09**: Learning Rate Schedules & Warmup (cosine annealing, linear warmup)
- **10**: Build Your Own Mini Framework (understand autograd internals)
- **12**: Introduction to JAX (functional paradigm for research)

---

## Tier 2 — Domain Mastery (~60-80 hours)

> Pick at least ONE domain and go deep. Many researchers cover two.

### Phase 4: Computer Vision — ✅ ALL 28 lessons (if vision is your domain)

> From CNNs through ViTs to 3D Gaussian Splatting and World Models. The full arc.

### Phase 5: NLP Foundations — ✅ ALL 29 lessons (if NLP is your domain)

> From tokenization through embeddings through every NLP task. Engineers skip most of this; you can't.

### Phase 6: Speech & Audio — ✅ ALL 17 lessons (if audio is your domain)

> Waveforms through neural codecs through streaming speech-to-speech.

---

## Tier 3 — Architecture & Training (~80 hours) 🔥

> **This is the heart of AI research.** Engineers barely touch this; you live here.

### Phase 7: Transformers Deep Dive — ✅ ALL 16 lessons

> Every lesson is essential. You don't just use transformers — you design new attention mechanisms.

Key researcher-only lessons:
- **05**: The Full Transformer: Encoder + Decoder (build it completely)
- **06**: BERT — Masked Language Modeling (understand pre-training objectives)
- **08**: T5, BART — Encoder-Decoder (architecture comparison)
- **09**: Vision Transformers (cross-domain attention)
- **10**: Audio Transformers — Whisper (multimodal architecture)
- **11**: Mixture of Experts (sparse computation)
- **13**: Scaling Laws (Chinchilla, compute-optimal training)
- **14**: Build a Transformer from Scratch (capstone)
- **15**: Attention Variants — Sliding Window, Sparse, Differential
- **16**: Speculative Decoding (inference innovation)

---

### Phase 8: Generative AI — ✅ ALL 15 lessons

> VAEs, GANs, Diffusion, Flow Matching — the generative paradigm.

All lessons are research-essential. Engineers skip this entire phase.

---

### Phase 9: Reinforcement Learning — ✅ ALL 12 lessons 🔥

> **RLHF requires deep RL knowledge.** You need to understand policy gradients, PPO, and reward modeling from first principles.

All 12 lessons are essential. Engineers skip this entirely.

---

### Phase 10: LLMs from Scratch — ✅ ALL 24 lessons 🔥

> **This defines you as a researcher.** Building tokenizers, pre-training GPTs, RLHF, DPO, quantization, novel architectures — all from scratch.

Key researcher-only lessons:
- **02**: Building a Tokenizer from Scratch
- **03**: Data Pipelines for Pre-Training
- **04**: Pre-Training a Mini GPT (124M)
- **05**: Distributed Training, FSDP, DeepSpeed
- **07**: RLHF — Reward Model + PPO
- **09**: Constitutional AI & Self-Improvement
- **15-22**: Speculative Decoding, Differential Attention, Native Sparse Attention, Multi-Token Prediction, DualPipe, DeepSeek-V3 walkthrough, Jamba hybrid SSM, Async/Hogwild inference

---

## Tier 4 — Frontiers (~80 hours)

### Phase 12: Multimodal AI — ✅ ALL 25 lessons

> The frontier of AI research. CLIP, LLaVA, Flamingo, Chameleon, Transfusion, embodied VLAs — this is where papers are published.

### Phase 15: Autonomous Systems — ✅ ALL 22 lessons

> Self-improvement, evolutionary agents, alignment — the cutting edge.

Key lessons:
- **02-08**: STaR family, AlphaEvolve, Darwin Gödel Machine, AI Scientist, Automated Alignment Research, Recursive Self-Improvement, Bounded Self-Improvement
- **17-22**: Constitutional AI, Llama Guard, RSP, Preparedness Framework, METR, CAIS

### Phase 16: Multi-Agent & Swarms — ✅ ALL 25 lessons

> Coordination, emergence, collective intelligence. Active research area.

### Phase 18: Ethics, Safety & Alignment — ✅ ALL 30 lessons 🔥

> **Non-negotiable for responsible research.** Alignment, deceptive alignment, mesa-optimization, scalable oversight, red-teaming, differential privacy, watermarking, regulation.

Key researcher-only lessons:
- **01-11**: Alignment theory (instruction-following, reward hacking, DPO family, sycophancy, Constitutional AI, mesa-optimization, sleeper agents, in-context scheming, alignment faking, AI control, scalable oversight)
- **17**: WMDP & Dual-Use Capability Evaluation
- **19**: Model Welfare Research
- **28**: Alignment Research Ecosystem: MATS, Redwood, Apollo, METR

---

## Tier 5 — Applied Context (~20 hours, lighter coverage)

> Researchers should understand how their work gets deployed, but don't need production mastery.

### Phase 11: LLM Engineering — ⚡ 6 of 17 lessons

| # | Lesson | Why |
|:-:|--------|-----|
| 01 | Prompt Engineering | Understand how your models are used |
| 06 | RAG | Major application pattern |
| 08 | Fine-Tuning with LoRA & QLoRA | Applied research |
| 10 | Evaluation & Testing | Benchmark your work |
| 12 | Guardrails & Safety | Responsible deployment |
| 14 | Model Context Protocol (MCP) | Industry context |

### Phase 13: Tools & Protocols — ⚡ 5 of 31 lessons

| # | Lesson | Why |
|:-:|--------|-----|
| 01 | The Tool Interface | How agents use tools |
| 02 | Function Calling Deep Dive | Foundation |
| 06 | MCP Fundamentals | Industry standard |
| 19 | A2A Protocol | Agent-to-Agent |
| 20 | OpenTelemetry GenAI | Experiment tracking |

### Phase 14: Agent Engineering — ⚡ 10 of 54 lessons

| # | Lesson | Why |
|:-:|--------|-----|
| 01 | The Agent Loop | Foundation |
| 03 | Reflexion and Verbal RL | Research pattern |
| 04 | Tree of Thoughts and LATS | Research pattern |
| 07 | Agent Memory — Virtual Context | Research area |
| 10 | Skill Libraries and Lifelong Learning (Voyager) | Research area |
| 11 | Planning with HTN and Evolutionary Search | Research area |
| 19 | Benchmarks — SWE-bench, GAIA | Evaluate your work |
| 20 | Benchmarks — WebArena, OSWorld | Evaluate your work |
| 25 | Multi-Agent Debate and Collaboration | Research pattern |
| 30 | Eval-Driven Agent Development | Methodology |

### Phase 17: Infrastructure — ⚡ 5 of 28 lessons

| # | Lesson | Why |
|:-:|--------|-----|
| 03 | GPU Autoscaling on Kubernetes | Training at scale |
| 04 | Serving Engine Internals | Understand inference |
| 08 | Inference Metrics | Benchmark your models |
| 09 | Production Quantization | Model compression |
| 17 | Disaggregated Prefill/Decode | Architecture understanding |

---

## Tier 6 — Capstone Projects (pick 3-5)

### Phase 19: Researcher-Focused Projects

| # | Project | Why |
|:-:|---------|-----|
| 05 | Autonomous Research Agent (AI-Scientist Class) | Your dream project |
| 07 | End-to-End Fine-Tuning Pipeline | Applied research |
| 14 | Speculative-Decoding Inference Server | Architecture innovation |
| 15 | Constitutional Safety Harness + Red-Team Range | Alignment research |
| 17 | Personal AI Tutor (Adaptive, Multimodal) | Multimodal application |

**Deep-build tracks for Researchers:**

| Track | Projects | Why |
|-------|----------|-----|
| B. NLP LLM | 30-41 | Build a complete LLM from scratch |
| C. Train end-to-end | 42-49 | Full training pipeline |
| D. Auto research | 50-57 | Build an AI Scientist |
| E. Multimodal VLM | 58-63 | Build a VLM from scratch |
| H. Distributed train | 76-81 | Distributed training from scratch |

---

## Summary: AI Researcher at a Glance

```
ESSENTIAL (do all lessons):
  Phase 0  — Setup & Tooling           (12 lessons)
  Phase 1  — Math Foundations           (22 lessons)  ← YOUR CORE
  Phase 2  — ML Fundamentals           (18 lessons)
  Phase 3  — Deep Learning Core        (13 lessons)  ← YOUR CORE
  Phase 7  — Transformers Deep Dive    (16 lessons)  ← YOUR CORE
  Phase 8  — Generative AI             (15 lessons)
  Phase 9  — Reinforcement Learning    (12 lessons)  ← YOUR CORE
  Phase 10 — LLMs from Scratch         (24 lessons)  ← YOUR CORE
  Phase 15 — Autonomous Systems        (22 lessons)
  Phase 16 — Multi-Agent & Swarms      (25 lessons)
  Phase 18 — Ethics & Alignment        (30 lessons)  ← YOUR CORE

ESSENTIAL (do all lessons in your chosen domain — pick 1-2):
  Phase 4  — Computer Vision           (28 lessons)
  Phase 5  — NLP Foundations            (29 lessons)
  Phase 6  — Speech & Audio            (17 lessons)
  Phase 12 — Multimodal AI             (25 lessons)

LIGHTER COVERAGE:
  Phase 11 — LLM Engineering           (6 of 17)
  Phase 13 — Tools & Protocols         (5 of 31)
  Phase 14 — Agent Engineering         (10 of 54)
  Phase 17 — Infrastructure            (5 of 28)
```

---

## Side-by-Side Comparison

| Phase | AI Engineer | AI Researcher |
|-------|:-----------:|:-------------:|
| 0 — Setup & Tooling | ✅ All 12 | ✅ All 12 |
| 1 — Math Foundations | ⚡ 8 of 22 | ✅ All 22 |
| 2 — ML Fundamentals | ⚡ 10 of 18 | ✅ All 18 |
| 3 — Deep Learning Core | ⚡ 7 of 13 | ✅ All 13 |
| 4 — Computer Vision | ❌ Skip | ✅ Domain choice |
| 5 — NLP Foundations | ⚡ 10 of 29 | ✅ Domain choice |
| 6 — Speech & Audio | ❌ Skip | ✅ Domain choice |
| 7 — Transformers | ⚡ 6 of 16 | ✅ All 16 |
| 8 — Generative AI | ❌ Skip | ✅ All 15 |
| 9 — Reinforcement Learning | ❌ Skip | ✅ All 12 |
| 10 — LLMs from Scratch | ⚡ Optional 6 | ✅ All 24 |
| 11 — LLM Engineering | ✅ All 17 🔥 | ⚡ 6 of 17 |
| 12 — Multimodal AI | ⚡ Optional 6 | ✅ All 25 |
| 13 — Tools & Protocols | ⚡ 20 of 31 🔥 | ⚡ 5 of 31 |
| 14 — Agent Engineering | ⚡ 30 of 54 🔥 | ⚡ 10 of 54 |
| 15 — Autonomous Systems | ⚡ 8 of 22 | ✅ All 22 |
| 16 — Multi-Agent & Swarms | ❌ Skip | ✅ All 25 |
| 17 — Infrastructure | ✅ All 28 🔥 | ⚡ 5 of 28 |
| 18 — Ethics & Alignment | ⚡ 8 of 30 | ✅ All 30 |
| 19 — Capstone Projects | ⚡ Engineer picks | ⚡ Research picks |
| **Total lessons** | **~170 of 523** | **~420 of 523** |
| **Estimated hours** | **~180-220 hrs** | **~300-342 hrs** |
