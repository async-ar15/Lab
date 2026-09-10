# 🔥 Project 3: Agni — Cross-Platform Model Optimization Toolkit
# Complete Study & Build Plan

> **Goal:** Build an open-source Python CLI toolkit that automates the full model optimization pipeline — PyTorch → ONNX export → quantization → target runtime optimization → benchmarking → deployment workbook generation — targeting the Performance Engineer (On-Device Inference) role at Sarvam AI.

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

Agni is NOT a web server and NOT an API. It's a **CLI toolkit** that takes a PyTorch model as input and produces optimized, benchmarked, documented inference artifacts for multiple target runtimes. It is the tool a Performance Engineer would use daily to bridge the gap between *"the model works on our training cluster"* and *"the model runs on a customer's phone/laptop/edge device."*

The JD states the mission bluntly: *"Take Sarvam models from research-handoff state to production-ready artifacts on at least two of our target chipsets."*

### Core Components You're Building

| Component | What It Is |
|---|---|
| **Export Engine** | PyTorch → ONNX converter handling dynamic shapes, control flow, custom ops, opset negotiation |
| **Quantization Engine** | INT8, INT4, FP16 with configurable strategies (PTQ static/dynamic, GPTQ, AWQ), per-layer sensitivity analysis, mixed-precision search |
| **Target Runtime Backends** | Optimization for ONNX Runtime (CPU/GPU), TensorRT (NVIDIA), CoreML (Apple), OpenVINO (Intel), LiteRT (ARM), WebGPU (Browser) |
| **Accuracy Validator** | Run quantized model against reference outputs, flag quality degradation beyond threshold |
| **Benchmark Harness** | Latency (p50/p95/p99), throughput (tok/sec), memory footprint, thermal behavior, power consumption |
| **Deployment Workbook Generator** | Auto-generate Markdown deployment workbooks with all results, configs, and reproduction steps |

### Tech Stack

- **CLI Framework:** Python + Click/Typer
- **Model Loading:** HuggingFace Transformers + safetensors
- **ONNX Export:** torch.onnx.export + onnx-simplifier
- **Quantization:** ONNX Runtime quantization, AutoGPTQ, AutoAWQ
- **Target Runtimes:** onnxruntime, tensorrt, coremltools, openvino, tflite
- **Benchmarking:** Custom harness + psutil + nvidia-smi/powermetrics
- **Workbook Generation:** Jinja2 templates → Markdown
- **CI/CD:** GitHub Actions matrix builds

---

## Backend Phases & Skills Required

These are the chapters from [Backend-from-first-Principle](file:///c:/Lab/Backend-from-first-Principle) you need to study.

> [!NOTE]
> **Agni is a CLI tool, not a web server.** This means many backend chapters (HTTP, Routing, API Design, Auth, WebSockets) are NOT needed. However, the *engineering discipline* chapters — error handling, config management, logging, testing, Docker — are MORE important than ever because ONNX export and quantization pipelines are notoriously fragile.

### 🔴 Critical (Must Do Before Building)

| # | Chapter | Why You Need It for Agni | Specific Skills |
|---|---|---|---|
| **3** | [Serialization and Deserialization](file:///c:/Lab/Backend-from-first-Principle/3.Serialization-and-Deserialization-or-backend-engineers) | Agni's entire job is format conversion: PyTorch tensors → ONNX protobuf → quantized weights → target runtime formats. Benchmark results serialize to JSON. Deployment workbooks are generated from structured data → Markdown. Understanding binary serialization is critical for model artifacts | Binary/protobuf serialization, JSON output, format conversion pipelines, safetensors format |
| **12** | [Error Handling and Fault-Tolerant Systems](file:///c:/Lab/Backend-from-first-Principle/12.%20Error%20Handling%20and%20Building%20Fault%20Tolerant%20Systems) | **This is THE most critical backend chapter for Agni.** ONNX export is "full of landmines" — dynamic shapes break, control flow fails, custom ops aren't registered, opsets are incompatible. Quantization can silently degrade quality. The pipeline MUST handle partial failures gracefully (export works for 3 of 4 targets, 1 fails) | Error propagation across pipeline stages, partial failure handling, retry strategies, detailed error messages with fix suggestions |
| **14** | [Production-grade Configuration Management](file:///c:/Lab/Backend-from-first-Principle/14.Production-grade%20Configuration%20Management) | Agni's CLI takes dozens of parameters: model source, target runtimes, quantization strategies, accuracy thresholds, benchmark durations, calibration dataset paths, output directories. These need config files (YAML/TOML), CLI arg overrides, and validation | Config loading (YAML/TOML), CLI argument parsing, config validation, default values, config inheritance |
| **15** | [Logging, Monitoring, and Observability](file:///c:/Lab/Backend-from-first-Principle/15.Logging,%20Monitoring%20and%20Observability) | Agni runs long pipelines (export → quantize → benchmark can take hours). Users need structured progress logging, benchmark result logging, and clear pipeline status. The auto-generated workbook depends on structured log/result data | Structured logging, progress bars, JSON result output, pipeline stage tracking |

### 🟡 Important (Need During Building)

| # | Chapter | Why You Need It for Agni | Specific Skills |
|---|---|---|---|
| **5** | [Validations and Transformations](file:///c:/Lab/Backend-from-first-Principle/5.%20Validations%20and%20transformations%20for%20backend%20engineers) | Validate model inputs (is the model a valid PyTorch checkpoint?), validate config parameters (is the accuracy threshold between 0 and 1?), validate calibration datasets, transform benchmark results into workbook format | Input validation, config schema validation, data transformation pipelines |
| **22** | [Automated Testing](file:///c:/Lab/Backend-from-first-Principle/22.Automated-Testing-Unit-Integration-and-E2E) | Testing the Agni pipeline itself: does export produce valid ONNX? does quantization preserve accuracy within threshold? does the benchmark harness produce correct metrics? Testing across multiple model architectures | pytest, fixtures, mocking (mock model for fast tests), integration tests, parameterized tests across targets |
| **21** | [Containerization — Docker, K8s, CI/CD](file:///c:/Lab/Backend-from-first-Principle/21.Containerization-and-Deployment-Docker-Kubernetes-and-CICD) | Containerized benchmark environments for reproducibility. GitHub Actions matrix builds testing across platforms. Docker images with pre-installed target runtimes (ONNX Runtime, OpenVINO, etc.) | Dockerfile, multi-stage builds, GitHub Actions matrix, CI/CD pipelines |

### 🟢 Nice to Have (Polish Phase)

| # | Chapter | Why You Need It for Agni | Specific Skills |
|---|---|---|---|
| **20** | [Concurrency & Parallelism](file:///c:/Lab/Backend-from-first-Principle/20.Concurrency%20&%20Parallelism%20-%20IO%20Bound%20vs%20CPU%20Bound) | Running benchmarks across multiple targets in parallel. Quantizing for multiple strategies concurrently. Understanding CPU-bound vs GPU-bound workloads for benchmark scheduling | multiprocessing, ProcessPoolExecutor, parallel benchmark execution, resource contention |
| **16** | [Graceful Shutdown](file:///c:/Lab/Backend-from-first-Principle/16.Graceful%20Shutdown) | Long-running benchmarks (thermal stress tests run for 5+ minutes). Users may Ctrl+C. Agni needs to save partial results, clean up temp files, and report what completed | Signal handlers, partial result saving, temp file cleanup |

---

## AI Engineering Phases & Skills Required

These are the phases from [ai-engineering-from-scratch](file:///c:/Lab/ai-engineering-from-scratch/phases) you need to study.

> [!IMPORTANT]
> **Agni is the most ML-heavy project.** Unlike Karta (agent engineering) and Vani (backend + RAG), Agni requires deep understanding of model internals — how neural networks are structured, how tensors flow through layers, what attention mechanisms look like at the implementation level. You're not *using* models, you're *dissecting and rebuilding* them for different hardware.

### 🔴 Critical — Foundations (Must Do First)

| Phase | Name | Why You Need It for Agni | Key Lessons |
|---|---|---|---|
| **Phase 00** | [Setup and Tooling](file:///c:/Lab/ai-engineering-from-scratch/phases/00-setup-and-tooling) | Dev environment, Python envs, Docker for AI, GPU setup, model downloads. You need CUDA, PyTorch, ONNX Runtime all working locally | `01-dev-environment`, `06-python-environments`, `07-docker-for-ai` |
| **Phase 01** (selected) | [Math Foundations](file:///c:/Lab/ai-engineering-from-scratch/phases/01-math-foundations) | **Yes, you need math for Agni.** Quantization is linear algebra (scaling float tensors to int ranges). Understanding tensor operations, matrix multiplication precision, and numerical stability directly impacts quantization quality | Linear algebra (tensors, matrix ops), numerical precision (float32 vs float16 vs int8 representation), statistics (for benchmark analysis — p50/p95/p99) |
| **Phase 03** | [Deep Learning Core](file:///c:/Lab/ai-engineering-from-scratch/phases/03-deep-learning-core) | **Critical.** You need to understand what's inside the models you're optimizing: layers (linear, conv, normalization), activation functions, forward pass execution, computation graphs, autograd. ONNX export traces the computation graph — if you don't understand the graph, you can't debug export failures | Neural network layers, computation graphs, forward pass tracing, PyTorch internals, model architecture |

### 🔴 Critical — Core Agni Skills (The Heart of the Toolkit)

| Phase | Name | Why You Need It for Agni | Key Lessons |
|---|---|---|---|
| **Phase 07** | [Transformers Deep Dive](file:///c:/Lab/ai-engineering-from-scratch/phases/07-transformers-deep-dive) | **THE most critical phase for Agni.** Sarvam's models are transformers (LLMs, translation, TTS). You need to understand attention mechanisms, KV-cache (critical for inference optimization), positional encodings, encoder-decoder architectures. ONNX export of attention with KV-cache is one of the hardest engineering challenges | Self-attention, multi-head attention, KV-cache mechanics, positional encoding, encoder-decoder architecture, model architecture variants |
| **Phase 10** | [LLMs from Scratch](file:///c:/Lab/ai-engineering-from-scratch/phases/10-llms-from-scratch) | **Understanding model internals is essential for debugging export and quantization.** Tokenizer export, embedding layers, attention implementation details, how pre-training shapes the weight distributions (affects quantization sensitivity). You don't need to train an LLM, but you need to understand its anatomy | `01-tokenizers` → tokenizer export, `02-building-a-tokenizer` → tokenizer internals, `03-attention-mechanisms` → attention export challenges, `04-pre-training-mini-gpt` → understand weight distributions |
| **Phase 17** | [Infrastructure & Production](file:///c:/Lab/ai-engineering-from-scratch/phases/17-infrastructure-and-production) | **Directly maps to Agni's target.** Model serving infrastructure, deployment optimization, GPU utilization, inference optimization techniques, model compilation. This is where export → optimize → serve lives | `01-gpu-fundamentals` → GPU architecture for benchmarking, `02-model-serving-fundamentals` → serving optimization, `03-inference-optimization` → quantization/pruning/distillation, `04-model-compilation` → ONNX/TensorRT compilation, `05-vllm-deep-dive` → serving architecture, `08-quantization-deep-dive` → **THE key lesson**, `09-onnx-export-and-runtime` → **THE other key lesson** |

### 🟡 Important — Deepening Skills

| Phase | Name | Why You Need It for Agni | Key Lessons |
|---|---|---|---|
| **Phase 02** (selected) | [ML Fundamentals](file:///c:/Lab/ai-engineering-from-scratch/phases/02-ml-fundamentals) | Understanding evaluation metrics (BLEU, WER, perplexity, accuracy) that you use to validate quantized model quality. Loss functions help understand what "accuracy degradation" means numerically | Evaluation metrics, loss functions, model evaluation methodology |
| **Phase 05** (selected) | [NLP Foundations](file:///c:/Lab/ai-engineering-from-scratch/phases/05-nlp-foundations-to-advanced) | Understanding the NLP models you're optimizing — translation models (BLEU validation), ASR models (WER validation). Embedding model internals for quantization sensitivity | `22-embedding-models-deep-dive` → embedding quantization, `18-multilingual-nlp` → Hindi model considerations |
| **Phase 06** (selected) | [Speech and Audio](file:///c:/Lab/ai-engineering-from-scratch/phases/06-speech-and-audio) | Sarvam has TTS models that need optimization. Understanding TTS model architecture helps with export and quantization decisions specific to audio models | `07-text-to-speech` → TTS model architecture, `03-automatic-speech-recognition` → ASR model architecture |
| **Phase 11** (selected) | [LLM Engineering](file:///c:/Lab/ai-engineering-from-scratch/phases/11-llm-engineering) | Understanding how optimized models are ultimately used in production. Token counting for throughput benchmarks. Model context length implications for KV-cache memory profiling | `01-prompt-engineering` → understanding usage patterns, `04-embeddings` → embedding model optimization, `13-production-app` → production model usage patterns |

### 🟢 Nice to Have — Advanced Topics

| Phase | Name | Why You Need It for Agni | Key Lessons |
|---|---|---|---|
| **Phase 04** (selected) | [Computer Vision](file:///c:/Lab/ai-engineering-from-scratch/phases/04-computer-vision) | If optimizing vision models (OCR, document understanding). Understanding CNN architectures for export | Conv layer export, vision model architectures |
| **Phase 12** (selected) | [Multimodal AI](file:///c:/Lab/ai-engineering-from-scratch/phases/12-multimodal-ai) | If optimizing multimodal models that combine text + image + audio | Cross-modal architecture export challenges |

---

## Skills You DON'T Need for This Project

These phases/chapters are **NOT required** for Agni. Don't waste time on them:

| Phase | Name | Why You Can Skip It |
|---|---|---|
| Phase 08 | Generative AI (images) | Not optimizing image generation models |
| Phase 09 | Reinforcement Learning | No RL in model optimization |
| Phase 13 | Tools and Protocols (MCP) | Agni is a CLI, not an agent tool |
| Phase 14 | Agent Engineering | Not building agents |
| Phase 15 | Autonomous Systems | Not building autonomous agents |
| Phase 16 | Multi-Agent & Swarms | Not building multi-agent systems |
| Phase 18 | Ethics, Safety, Alignment | Not relevant for model optimization |
| Phase 19 | Capstone Projects | Reference only |
| Backend #1 | HTTP and CORS | Agni is a CLI, not a web server |
| Backend #2 | Routing | No web routing needed |
| Backend #4 | Authentication & Authorization | No auth in a CLI tool |
| Backend #6 | Controllers/Services/Middlewares | No web architecture needed |
| Backend #7 | API Design — REST | No REST API |
| Backend #8 | Database with Backend | No database (file-based artifacts) |
| Backend #9 | Caching | Not needed for CLI pipeline |
| Backend #10 | Task Queues | Not needed |
| Backend #11 | Elasticsearch | No search |
| Backend #13 | gRPC | No inter-service communication |
| Backend #17 | Backend Security | No web security needed |
| Backend #18-19 | Scaling & Performance (web) | Web scaling, not model optimization |
| Backend #23 | Message Brokers / Kafka | Not needed |
| Backend #24 | WebSockets | No real-time communication |

---

## Recommended Study Order (The Flow)

> **Strategy:** Learn → Build incrementally. Agni's learning curve is steeper than Karta/Vani because you need to understand model internals before you can export and optimize them. The study blocks are front-loaded with ML theory.

```
📚 STUDY BLOCK 1: "Math & ML Foundations"                ⏱️ ~4-5 days
├── AI Phase 01: Math Foundations (selected — linear 
│   algebra, numerical precision, statistics)
├── AI Phase 02: ML Fundamentals (selected — evaluation
│   metrics: BLEU, WER, perplexity, accuracy)
├── AI Phase 00: Setup and Tooling (dev env, GPU, Docker)
└── Backend #3: Serialization/Deserialization
         │
         ▼
📚 STUDY BLOCK 2: "Deep Learning Internals"             ⏱️ ~5-7 days
├── AI Phase 03: Deep Learning Core (full)
│   — layers, activations, computation graphs,
│     forward pass tracing, PyTorch internals
├── AI Phase 07: Transformers Deep Dive (full)
│   — attention, KV-cache, positional encoding,
│     encoder-decoder, model architectures
└── AI Phase 10: LLMs from Scratch (01-04)
    — tokenizers, attention implementation,
      weight distributions, pre-training concepts
         │
         ▼
🔨 BUILD CHECKPOINT 1: "Hello Export"                    ⏱️ ~2-3 days
│   Build: CLI that takes a HuggingFace model name,
│   loads it with Transformers, and exports to ONNX
│   using torch.onnx.export. Handle basic dynamic
│   shapes. Validate exported ONNX matches PyTorch
│   outputs within epsilon. Just export — no quant,
│   no benchmark yet.
│
         ▼
📚 STUDY BLOCK 3: "Quantization Deep Dive"              ⏱️ ~5-7 days
├── AI Phase 17: Infrastructure & Production
│   — 08-quantization-deep-dive (THE key lesson)
│   — 09-onnx-export-and-runtime
│   — 03-inference-optimization
│   — 04-model-compilation
├── Quantization theory: PTQ vs QAT, symmetric vs
│   asymmetric, per-tensor vs per-channel, calibration
├── AutoGPTQ documentation + tutorials
├── AutoAWQ documentation + tutorials
└── Backend #12: Error Handling (export failure recovery)
         │
         ▼
📚 STUDY BLOCK 4: "Configuration & CLI Design"          ⏱️ ~2-3 days
├── Backend #14: Configuration Management
├── Backend #5: Validations (config + input validation)
├── Click/Typer CLI framework documentation
└── Backend #15: Logging (structured pipeline logging)
         │
         ▼
🔨 BUILD CHECKPOINT 2: "Export + Quantize"               ⏱️ ~3-4 days
│   Build: Add quantization engine to the CLI.
│   Support INT8 (dynamic + static PTQ), INT4 (GPTQ),
│   and FP16. Calibration dataset loading. Per-layer
│   sensitivity analysis. Accuracy validation against
│   reference outputs. Proper CLI with Click/Typer,
│   config files, and structured logging.
│
         ▼
📚 STUDY BLOCK 5: "Target Runtimes"                     ⏱️ ~4-5 days
├── AI Phase 17: Infrastructure & Production
│   — 01-gpu-fundamentals
│   — 02-model-serving-fundamentals
│   — 05-vllm-deep-dive
├── ONNX Runtime optimization docs
├── CoreML tools documentation
├── OpenVINO toolkit documentation
├── TensorRT documentation (concepts)
└── LiteRT / TFLite documentation
         │
         ▼
🔨 BUILD CHECKPOINT 3: "Multi-Target Optimization"      ⏱️ ~3-4 days
│   Build: Add target runtime backends. At minimum:
│   ONNX Runtime (CPU/GPU) + one of CoreML/OpenVINO.
│   Each backend: convert quantized ONNX → target
│   format, validate outputs, report compatibility.
│   Plugin architecture for adding new backends.
│
         ▼
📚 STUDY BLOCK 6: "Benchmarking & Profiling"            ⏱️ ~3-4 days
├── AI Phase 17: Production (GPU profiling, serving perf)
├── psutil documentation — CPU/memory profiling
├── nvidia-smi / NVML — GPU profiling
├── Thermal profiling techniques (throughput over time)
├── Statistics: p50/p95/p99 calculation, warmup handling,
│   confidence intervals
└── Backend #22: Automated Testing
         │
         ▼
🔨 BUILD CHECKPOINT 4: "Benchmark Harness"              ⏱️ ~3-4 days
│   Build: Full benchmark harness measuring latency
│   (p50/p95/p99, warmup-aware), throughput (tok/sec,
│   req/sec), memory (peak RSS, VRAM), and thermal
│   behavior (throughput degradation over sustained
│   load). Comparison tables across all targets +
│   quantization levels. JSON result output.
│
         ▼
📚 STUDY BLOCK 7: "Documentation & Polish"              ⏱️ ~2-3 days
├── Jinja2 templating documentation
├── AI Phase 05: Embedding Models (22) — if optimizing
│   embedding models
├── AI Phase 06: TTS (07) + ASR (03) — if optimizing
│   speech models
├── Backend #21: Docker/CI/CD
└── Backend #20: Concurrency (parallel benchmarks)
         │
         ▼
🔨 BUILD CHECKPOINT 5: "Workbook Generator + CI"        ⏱️ ~2-3 days
│   Build: Jinja2 template → auto-generated deployment
│   workbook (Markdown) with model card, quantization
│   config, accuracy validation, benchmark comparison
│   tables, reproduction commands, known issues.
│   GitHub Actions CI with matrix builds.
│
         ▼
📚 STUDY BLOCK 8: "Advanced Optimization" (Optional)    ⏱️ ~2-3 days
├── Mixed-precision search (critical layers stay FP16)
├── KV-cache optimization for LLM export
├── TensorRT advanced (engine building, plugins)
├── Backend #16: Graceful Shutdown (interrupt long benchmarks)
└── AI Phase 17: Shadow/canary deployment (20)
         │
         ▼
🔨 FINAL: "Ship It"                                     ⏱️ ~2-3 days
│   Optimize a real Sarvam-compatible model end-to-end.
│   Generate a polished deployment workbook. README
│   with architecture diagram, Quick Start guide,
│   sample workbook, Design Decisions doc.
```

---

## Week-by-Week Execution Plan

### Week 1: ML Foundations & Deep Learning Internals
**Study:** Blocks 1 + 2 (Math/ML Foundations + Deep Learning Internals)

| Day | Activity | Output |
|---|---|---|
| Day 1 | AI Phase 00 (Setup) + GPU environment setup (CUDA, PyTorch, ONNX Runtime) | Working dev environment with GPU support |
| Day 2 | AI Phase 01: Linear Algebra (tensors, matrix ops) + Numerical Precision (float32/16/int8) | Understand tensor operations, quantization math |
| Day 3 | AI Phase 02: ML Fundamentals (evaluation metrics — BLEU, WER, perplexity) + Backend #3 (Serialization) | Understand validation metrics, binary serialization |
| Day 4 | AI Phase 03: Deep Learning Core — layers, activations, computation graphs | Understand neural network architecture internals |
| Day 5 | AI Phase 03: Deep Learning Core — forward pass tracing, PyTorch internals, autograd | Understand how PyTorch traces models (basis for ONNX export) |
| Day 6 | AI Phase 07: Transformers — self-attention, multi-head attention, KV-cache | Understand transformer architecture at implementation level |
| Day 7 | AI Phase 07: Transformers — encoder-decoder, positional encoding + AI Phase 10: LLMs (01-04) | Understand LLM anatomy: tokenizers, attention, weight distributions |

**Milestone:** Deep understanding of what's inside the models you'll be optimizing. Can explain attention, KV-cache, and computation graphs.

---

### Week 2: First Export + Quantization Engine
**Study:** Blocks 3 + 4 (Quantization + Configuration/CLI)
**Build:** Checkpoints 1 + 2

| Day | Activity | Output |
|---|---|---|
| Day 8 | AI Phase 17: ONNX Export & Runtime (09) + Inference Optimization (03) | Understand ONNX format, export mechanics, optimization techniques |
| Day 9 | **BUILD:** Checkpoint 1 — CLI that exports HuggingFace model → ONNX | Working ONNX export with dynamic shape handling |
| Day 10 | AI Phase 17: Quantization Deep Dive (08) — PTQ, QAT, calibration, sensitivity | Understand quantization theory and techniques |
| Day 11 | AutoGPTQ + AutoAWQ documentation + AI Phase 17: Model Compilation (04) | Understand practical quantization tools |
| Day 12 | Backend #12 (Error Handling) + Backend #14 (Config Management) + Backend #5 (Validations) | Robust error handling for export failures, CLI config |
| Day 13 | Backend #15 (Logging) + Click/Typer CLI framework study | Structured logging, CLI argument design |
| Day 14 | **BUILD:** Checkpoint 2 — quantization engine (INT8/INT4/FP16) + CLI + config + logging | Export + quantize pipeline with proper CLI interface |

**Milestone:** CLI that exports a model to ONNX and quantizes it with multiple strategies, validates accuracy.

---

### Week 3: Target Runtimes + Benchmark Harness
**Study:** Blocks 5 + 6 (Target Runtimes + Benchmarking)
**Build:** Checkpoints 3 + 4

| Day | Activity | Output |
|---|---|---|
| Day 15 | AI Phase 17: GPU Fundamentals (01) + Model Serving (02) + vLLM (05) | GPU architecture, serving patterns, inference optimization |
| Day 16 | ONNX Runtime optimization docs + CoreML tools documentation | Target runtime conversion patterns |
| Day 17 | OpenVINO toolkit docs + TensorRT concepts | Intel + NVIDIA optimization backends |
| Day 18 | **BUILD:** Checkpoint 3 — multi-target optimization (ONNX Runtime + CoreML or OpenVINO) | Quantized ONNX → target format conversion + validation |
| Day 19 | Benchmark profiling: psutil, nvidia-smi, statistics (p50/p95/p99, warmup) | Profiling tools, statistical analysis for benchmarks |
| Day 20 | Thermal profiling techniques + Backend #22 (Automated Testing) | Throughput-over-time measurement, test suite design |
| Day 21 | **BUILD:** Checkpoint 4 — benchmark harness (latency, throughput, memory, thermal) | Full benchmark harness with comparison tables |

**Milestone:** Can export, quantize, optimize for multiple targets, and benchmark with proper statistical analysis.

---

### Week 4: Workbook Generation, Advanced Optimization & Ship
**Study:** Blocks 7 + 8 (Documentation + Advanced)
**Build:** Checkpoints 5 + Final

| Day | Activity | Output |
|---|---|---|
| Day 22 | Jinja2 templating + AI Phase 05: Embedding Models (22) | Template system, embedding model optimization |
| Day 23 | AI Phase 06: TTS (07) + ASR (03) — speech model optimization considerations | Understanding speech model architectures for export |
| Day 24 | **BUILD:** Checkpoint 5 — deployment workbook generator + GitHub Actions CI | Auto-generated workbook with all results, CI matrix |
| Day 25 | Backend #21 (Docker/CI/CD) + Backend #20 (Concurrency — parallel benchmarks) | Containerized benchmarks, parallel target processing |
| Day 26 | Advanced: mixed-precision search, KV-cache optimization, Backend #16 (Graceful Shutdown) | Critical layers stay FP16, interrupt-safe benchmarks |
| Day 27 | **BUILD:** End-to-end optimization of a real Sarvam-compatible model | Full pipeline demo with polished workbook |
| Day 28 | **FINAL:** README, Quick Start, sample workbook, architecture diagram, Design Decisions | Ship-ready GitHub repo |

**Milestone:** Production-ready Agni toolkit on GitHub with a sample deployment workbook.

---

## Key Deliverables Checklist

When you're done, your repo should have:

- [ ] **README.md** with architecture diagram (ASCII or Mermaid)
- [ ] **Quick Start** section — optimize a small model in < 5 minutes
- [ ] **CLI interface** with clean `agni optimize` command:
  ```bash
  agni optimize --model bert-base-uncased --targets onnxrt,coreml --quant int8,int4 --benchmark --workbook
  ```
- [ ] **At least 2 target runtimes working:**
  - ONNX Runtime (CPU/GPU)
  - One of: CoreML (Apple), OpenVINO (Intel), or TensorRT (NVIDIA)
- [ ] **At least 3 quantization strategies:**
  - INT8 dynamic PTQ
  - INT8 static PTQ (with calibration)
  - INT4 GPTQ or AWQ
- [ ] **Accuracy validation** showing pass/fail against reference outputs with configurable threshold
- [ ] **Benchmark results** showing:
  - Latency (p50/p95/p99)
  - Throughput (tokens/sec or requests/sec)
  - Memory footprint (peak RSS, VRAM)
  - Thermal stress test (throughput degradation over 5 minutes)
- [ ] **Sample deployment workbook** (auto-generated Markdown) with:
  - Model card
  - Quantization config
  - Accuracy validation results
  - Benchmark comparison table across targets
  - Reproduction commands
  - Known issues per target
- [ ] **Per-layer sensitivity analysis** showing which layers can tolerate INT4 and which need FP16
- [ ] **Design Decisions** doc explaining WHY you chose:
  - ONNX as the intermediate representation
  - Plugin architecture for target backends
  - Semantic versioning for deployment workbooks
  - Statistical methods for benchmark analysis
  - Thermal profiling methodology
- [ ] **Test suite** — export tests, quantization accuracy tests, benchmark correctness tests
- [ ] **CI/CD** — GitHub Actions matrix testing across Python versions and target runtimes
- [ ] **Docker support** — containerized benchmark environment for reproducibility

---

## Total Time Estimate

| Activity | Time |
|---|---|
| Study (all blocks) | ~27-37 days |
| Build (all checkpoints) | ~16-22 days |
| **Overlap (study + build interleaved)** | **~28-35 days (4-5 weeks)** |

> [!TIP]
> The study blocks and build checkpoints are **interleaved** in the week-by-week plan above. You study a block, then immediately build the corresponding component. This is faster than "learn everything first, then build."

> [!WARNING]
> **Agni has the steepest learning curve of all 5 projects.** Unlike Karta/Vani where you can start building quickly with APIs, Agni requires deep ML theory before you can write meaningful code. The first 7 days are pure study with no build checkpoint. This is intentional — you cannot debug ONNX export failures if you don't understand computation graphs, and you cannot tune quantization if you don't understand numerical precision.

> [!IMPORTANT]
> **The two most critical lessons for Agni are in AI Phase 17 (Infrastructure & Production): lesson 08 (Quantization Deep Dive) and lesson 09 (ONNX Export & Runtime).** If you're short on time, prioritize Phase 03 (Deep Learning Core) + Phase 07 (Transformers) + Phase 17 (Infrastructure) above everything else. The backend chapters can be learned as you code.

> [!NOTE]
> **Agni leverages your VoidChat experience.** Your work on VoidChat (browser-based LLM with WebGPU, dynamic 4-bit/8-bit quantization, thermal profiling) directly applies here. The thermal stress test metric and throughput-based thermal inference technique from VoidChat is something most benchmark tools completely ignore — this is your unique advantage.
