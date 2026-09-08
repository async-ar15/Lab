# The Ultimate AI Engineering Setup Theory Guide

Welcome to the **00 Setup & Tooling** super-guide! Before you write your first neural network or train an LLM, you must prepare your environment. AI code fails differently, relies on massive data, and demands specialized hardware. This single document synthesizes everything you need to know from the 12 core setup modules. 

Let's dive into the theory of how professional AI engineers set up their tools.

---

## 1. Dev Environment Architecture
AI environments are built bottom-up across four layers:
1. **System Foundation**: Your OS, terminal, and base GPU drivers.
2. **Package Managers**: Tools like `uv` (for Python) or `pnpm` (for Node.js) that install the runtimes.
3. **Language Runtimes**: Python 3.11+ (for ML models), Node.js (for agents/web), Rust (for performance), and Julia (for math).
4. **AI Libraries**: PyTorch, Transformers, JAX.

**Core Tool**: We use `uv` instead of standard `pip`. It is 10-100x faster and effortlessly handles both Python installations and virtual environments.

## 2. Git & Collaboration
Version control in AI is not just for code; it's for experiments. 
- You use the standard `git add`, `git commit`, `git push` workflow to save progress.
- You branch (`git checkout -b`) when testing new model architectures without breaking the main codebase.
- **Rule of Thumb**: **Never commit models or huge datasets to Git.** Your `.gitignore` must exclude `.pt`, `.safetensors`, and large `.csv` files to prevent the repository from bloating.

## 3. GPU Setup & Cloud Compute
AI models require serious compute. While you can learn on a CPU, training real models requires a GPU.
- **Local GPUs**: If you have an NVIDIA GPU, you install PyTorch with CUDA (`torch.cuda`). Mac users use MPS (`torch.backends.mps`).
- **Cloud/Colab**: If you don't have a GPU, Google Colab provides free T4 GPUs. For serious training runs, you SSH into cloud GPU instances (like Lambda Labs or Runpod).
- **Memory Rule**: A quick way to estimate if a model fits in VRAM (GPU memory) using `fp16` (half precision) is allocating ~2 bytes per parameter. A 7-billion parameter model needs roughly 14 GB of VRAM.

## 4. API Keys & LLM Integration
When building AI applications, you will often query external models (OpenAI, Anthropic, etc.).
- **Security**: Never hardcode an API key (`sk-...`). Always inject them using Environment Variables (`export ANTHROPIC_API_KEY="..."`) or a local `.env` file that is git-ignored.
- **Under the Hood**: While SDKs (like the Anthropic Python SDK) make calling models easy, they are just wrappers over standard HTTP REST requests. Understanding the raw HTTP requests is crucial for debugging rate limits or unauthorized errors.

## 5. Jupyter Notebooks
Notebooks are the lab bench of AI engineering. They allow you to run code in blocks (cells) and visualize data inline.
- **The Workflow**: The golden rule is to **explore in notebooks, ship in scripts**. Use notebooks to analyze data and prototype models, but port finished code to standard `.py` files.
- **Magic Commands**: Use `%timeit` for micro-benchmarks or `%matplotlib inline` to render charts seamlessly.
- **Traps**: Beware of "out-of-order execution" and "hidden state" memory leaks where deleted cells still hold variables in the kernel's memory.

## 6. Python Environments
Dependency hell is real in AI (e.g., PyTorch CUDA mismatches).
- **Virtual Environments (`venv`)**: Every single project gets its own isolated virtual environment. Never install `torch` globally. 
- **pyproject.toml & Lockfiles**: Use `pyproject.toml` to define dependencies. Use lockfiles to pin exact package versions so anyone cloning your repository gets identical setups.
- **Conda**: Use `conda` only when you need non-Python dependencies (like specific CUDA toolkits or C libraries) that `pip` cannot provide. 

## 7. Docker for AI
Docker solves the "it works on my machine" problem by bundling the OS, CUDA drivers, Python, and code into an isolated Container.
- **GPU Passthrough**: Linux requires the NVIDIA Container Toolkit so the Docker container can access the host's GPU.
- **Volumes**: You must mount host directories into the container for large assets (`-v ~/models:/models`). Otherwise, downloading a 15GB model inside a container will vanish the moment the container restarts.
- **Compose**: Use Docker Compose to spin up multi-service AI apps (like a Python inference server + a Qdrant vector database).

## 8. Editor Setup
Your editor should act as an active assistant. We use VS Code (or derivatives like Cursor/Windsurf).
- **Core Extensions**: Python, Pylance (for fast type-checking), Jupyter, and Ruff/Black (for formatting/linting).
- **Remote SSH**: This is the most vital extension. It allows you to edit files and run code directly on a remote cloud GPU box as if it were your local laptop.

## 9. Data Management
Data pipelines feed models. We use Hugging Face's `datasets` library for this.
- **Streaming**: For massive datasets that don't fit on disk, use `streaming=True` to process data row-by-row on the fly.
- **Formats**: While CSV is human-readable, **Parquet** (which uses Apache Arrow under the hood) is heavily compressed, vastly faster, and the standard for AI data storage.
- **Splits**: Always create reproducible splits (Train, Validation, Test) using a fixed `seed`.

## 10. Terminal & Shell Mastery
AI engineers live in the terminal.
- **Piping & Grep**: Chain tools using pipes (`|`) to analyze training outputs. e.g., `tail -f train.log | grep loss`.
- **tmux**: The ultimate terminal multiplexer. Use `tmux` to start a training run, detach the session, close your laptop, and reattach the next day without killing the training script. 
- **Monitoring**: Use `htop` (for CPU/Memory) and `nvidia-smi` or `nvtop` (for GPU usage) to constantly monitor system resources.

## 11. Linux for AI
Because you will inevitably SSH into remote Ubuntu servers, you must know basic Linux.
- **Navigation**: `cd`, `ls`, `grep`, `cat`, and `tail` are your bread and butter. 
- **Permissions**: Know how to fix "Permission denied" using `chmod +x` (make executable) or `sudo`.
- **Space Management**: GPUs run out of disk space rapidly. Learn to use `df -h` and `du -sh` to locate and delete bloated checkpoints or caches.

## 12. Debugging and Profiling
AI bugs are often silent—they don't crash, they just cause models to predict garbage.
- **Silent Failures**: The most common issues are Shape Mismatches, NaN (Not a Number) Loss, and tensors being on the wrong device (CPU instead of GPU).
- **Debugging**: Drop `breakpoint()` in your training loop to interactively inspect tensor shapes, data types (`dtypes`), and gradients during execution. Alternatively, craft precise `debug_print()` statements.
- **Profiling**: Use `cProfile`, `tracemalloc`, or custom timers to find bottlenecks. Often, slow training is caused by slow data loading, not a slow GPU.
- **TensorBoard**: Use TensorBoard to visualize loss curves and verify gradients aren't vanishing or exploding.

---
---

# DEEP DIVE: The Skills That Actually Matter

> Most of the 12 modules above are "install once, forget forever." The four topics below are different. These are **skills** — you will use them every single day of your AI engineering career. We teach them here from **first principles**: not just *how*, but *why* things work the way they do.

---
---

# Deep Dive 1: Debugging & Profiling (From First Principles)

> *"The worst AI bugs don't crash. They train silently on garbage and report a beautiful loss curve."*

## Why AI Debugging Is Fundamentally Different

Let's start from the very root. In traditional software engineering:
- You write code → it crashes → you read the stack trace → you fix the line → done.

In AI engineering:
- You write code → it runs for 8 hours → it finishes successfully → your model predicts the average of every input → **there was never an error message.**

This happens because AI code is fundamentally **numerical**, not **logical**. A web server either returns a response or it doesn't. A neural network returns *some* number no matter what — the question is whether that number is *meaningful*. This is why debugging AI requires a completely different mindset.

## First Principle: Everything Is a Tensor

Before you can debug AI code, you must internalize one fact: **every piece of data in a neural network is a tensor** (a multi-dimensional array of numbers). Every bug manifests as something wrong with a tensor. There are exactly five properties of a tensor you must check when debugging:

| Property | What it is | What goes wrong |
|----------|-----------|-----------------|
| **Shape** | The dimensions of the tensor, e.g. `[32, 3, 224, 224]` | A shape mismatch crashes immediately, but a *subtle* wrong shape (e.g., `[32, 1]` vs `[32]`) silently broadcasts and produces wrong results |
| **Dtype** | The data type: `float32`, `float16`, `int64`, etc. | Mixing `float32` and `float16` causes precision loss; passing `int` labels as `float` to a loss function silently computes garbage |
| **Device** | Where the tensor lives: `cpu` or `cuda:0` | If your model is on GPU but your data is on CPU, sometimes PyTorch auto-transfers (slowly), sometimes it crashes |
| **Values** | The actual numbers inside | NaN (Not a Number) or Inf (infinity) values poison every computation they touch |
| **Gradient** | Whether the tensor tracks gradients (`requires_grad`) | If you accidentally `.detach()` a tensor or use `.item()` inside the computation graph, gradients stop flowing and the model stops learning — silently |

### The `debug_print` Function — Your Swiss Army Knife

This single function checks all five properties at once:

```python
def debug_print(name, tensor):
    """Print everything you need to know about a tensor in one line."""
    print(
        f"{name}: "
        f"shape={tensor.shape}, "
        f"dtype={tensor.dtype}, "
        f"device={tensor.device}, "
        f"min={tensor.min().item():.4f}, "
        f"max={tensor.max().item():.4f}, "
        f"mean={tensor.mean().item():.4f}, "
        f"has_nan={tensor.isnan().any().item()}, "
        f"requires_grad={tensor.requires_grad}"
    )
```

**When to use it**: After every suspicious operation. Before and after a layer. At the input and output of your model. Once the bug is found, delete the prints.

## The Four Deadly AI Bugs (From First Principles)

### Bug 1: Shape Mismatch — *The Most Common Bug in All of AI*

**First Principle**: Every layer in a neural network expects a tensor of a specific shape. If the shape is wrong by even one dimension, either:
- The code crashes (lucky case — easy to fix), or
- PyTorch **broadcasts** the tensor silently (unlucky case — the model trains on garbage).

**What is broadcasting?** When two tensors have different shapes, PyTorch tries to automatically expand the smaller one to match the larger one. This is incredibly useful for math, but deadly when it hides bugs.

```python
# Example of silent broadcasting bug:
predictions = model(x)        # shape: [32, 10]  (32 samples, 10 classes)
labels = get_labels(batch)     # shape: [32, 1]   (WRONG — should be [32])

# PyTorch doesn't crash. It broadcasts [32, 1] to [32, 10].
# The loss computes, backprop runs, the model trains — on complete nonsense.
loss = (predictions - labels) ** 2  # This "works" but is totally wrong
```

**How to catch it**: Use `debug_print` on every tensor entering and leaving your model. Also use this hook-based shape tracer that maps every transformation:

```python
def check_shapes(model, sample_input):
    """Run one forward pass and print the shape at every layer."""
    print(f"Input: {sample_input.shape}")

    def make_hook(name):
        def hook(module, inp, out):
            in_shape = inp[0].shape if isinstance(inp, tuple) else inp.shape
            out_shape = out.shape if hasattr(out, "shape") else type(out)
            print(f"  {name}: {in_shape} -> {out_shape}")
        return hook

    hooks = []
    for name, module in model.named_modules():
        hooks.append(module.register_forward_hook(make_hook(name)))

    with torch.no_grad():
        model(sample_input)

    for h in hooks:
        h.remove()
```

Run this **once** at the start of training with a real batch. It maps the entire data flow through your model.

### Bug 2: NaN Loss — *Something Exploded*

**First Principle**: NaN stands for "Not a Number." It is the result of mathematically undefined operations like `0/0` or `∞ - ∞`. Once a single NaN enters a computation, **every subsequent result becomes NaN**. It's a virus.

**Common causes (from first principles)**:
1. **Learning rate too high**: Gradients become enormous → weights update by huge amounts → activations overflow to infinity → loss becomes NaN. *The fix: reduce learning rate by 10x.*
2. **Log of zero**: `log(0) = -∞`. If your model predicts a probability of exactly 0.0 for the correct class, `log(0)` produces `-inf`, and arithmetic with `-inf` produces NaN. *The fix: add a tiny epsilon, e.g., `torch.log(probs + 1e-8)`.*
3. **Division by zero**: Custom loss functions or normalization layers that divide by a value that can be zero. *The fix: add epsilon to the denominator.*
4. **Exploding gradients in RNNs**: Recurrent networks multiply the same weight matrix at every time step. If eigenvalues > 1, the product explodes exponentially. *The fix: gradient clipping with `torch.nn.utils.clip_grad_norm_`.*

**How to catch it**:

```python
def detect_nan(model, loss, step):
    """Call this after every training step."""
    if torch.isnan(loss):
        print(f"❌ NaN loss at step {step}")
        for name, param in model.named_parameters():
            if param.grad is not None:
                if torch.isnan(param.grad).any():
                    print(f"  NaN gradient in: {name}")
                if torch.isinf(param.grad).any():
                    print(f"  Inf gradient in: {name}")
        return True
    return False
```

### Bug 3: Data Leakage — *The 99% Accuracy Trap*

**First Principle**: A model must be evaluated on data it has **never seen during training**. If even a single training example leaks into the test set, the model memorizes it and reports inflated accuracy.

**Types of data leakage**:
1. **Direct leakage**: The same sample appears in both train and test sets. *Fix: check for overlapping IDs.*
2. **Temporal leakage**: Using future data to predict the past. If you're predicting stock prices, and your training set contains data from 2025 while your test set contains 2024, the model has "seen the future." *Fix: always split by time, not randomly.*
3. **Feature leakage**: A feature in your training data directly encodes the label. Example: predicting whether a patient has cancer, but the dataset includes a column "chemotherapy_prescribed" which is only `True` for cancer patients. *Fix: carefully audit every feature.*

**How to catch direct leakage**:

```python
def check_data_leakage(train_set, test_set, id_column="id"):
    train_ids = set(train_set[id_column].tolist())
    test_ids = set(test_set[id_column].tolist())
    overlap = train_ids & test_ids
    if overlap:
        print(f"⚠️ DATA LEAKAGE: {len(overlap)} samples in both train and test!")
        return True
    print("✅ No data leakage detected.")
    return False
```

### Bug 4: Wrong Device — *The Silent Performance Killer*

**First Principle**: A GPU is a separate computer inside your computer. It has its own memory (VRAM). Data must be explicitly copied between CPU RAM and GPU VRAM. If your model is on the GPU but one tensor stays on the CPU, two things can happen:
1. PyTorch crashes with `RuntimeError: Expected all tensors to be on the same device`.
2. PyTorch silently copies the tensor to the correct device every single step — your training "works" but runs 10x slower because of constant CPU↔GPU transfers.

```python
def check_devices(model, *tensors):
    """Verify everything is on the same device."""
    model_device = next(model.parameters()).device
    print(f"Model device: {model_device}")
    for i, t in enumerate(tensors):
        if t.device != model_device:
            print(f"  ⚠️ Tensor {i} on {t.device}, model on {model_device}")
```

## Profiling: Finding Where Time Goes

**First Principle**: Training speed is limited by the **slowest component** in the loop. Most beginners assume the GPU is the bottleneck. It rarely is. The actual bottleneck is usually **data loading** — reading files from disk, decoding images, applying transformations on the CPU.

### The Training Loop Has Three Phases

```
┌─────────────────────┐
│  1. Data Loading     │  ← Usually 40-70% of total time (CPU-bound)
│     Read from disk   │
│     Decode/transform │
├─────────────────────┤
│  2. Forward Pass     │  ← GPU computes predictions
│     model(inputs)    │
├─────────────────────┤
│  3. Backward Pass    │  ← GPU computes gradients
│     loss.backward()  │
│     optimizer.step() │
└─────────────────────┘
```

**How to measure each phase**:

```python
import time

class Timer:
    def __init__(self, name=""):
        self.name = name

    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, *args):
        elapsed = time.perf_counter() - self.start
        print(f"[{self.name}] {elapsed:.4f}s")

# Use it in your training loop:
with Timer("data loading"):
    batch = next(dataloader_iter)

with Timer("forward"):
    outputs = model(batch)

with Timer("backward"):
    loss.backward()
    optimizer.step()
```

**If data loading is the bottleneck** (which it usually is):
- Set `num_workers > 0` in your DataLoader (this spawns background processes to load data in parallel)
- Use `pin_memory=True` for faster CPU→GPU transfer
- Pre-process your data into Parquet/Arrow format instead of reading raw CSVs

### Memory Profiling: GPU OOM (Out Of Memory)

When you see `RuntimeError: CUDA out of memory`, the fix is almost always one of these (in order):

1. **Reduce batch size** — the single most effective fix
2. **Use mixed precision** (`torch.cuda.amp`) — halves memory usage
3. **Use gradient checkpointing** — trades compute for memory
4. **Delete intermediate tensors** — `del tensor; torch.cuda.empty_cache()`

Check GPU memory usage programmatically:

```python
if torch.cuda.is_available():
    allocated = torch.cuda.memory_allocated() / 1e9
    reserved = torch.cuda.memory_reserved() / 1e9
    print(f"GPU Memory — Allocated: {allocated:.2f} GB, Reserved: {reserved:.2f} GB")
```

## TensorBoard: Reading the Training Story

**First Principle**: A loss curve is the heartbeat of your training run. If you can read it, you can diagnose almost any problem.

```python
from torch.utils.tensorboard import SummaryWriter

writer = SummaryWriter("runs/experiment_1")

for step in range(num_steps):
    loss = train_step(model, batch)
    writer.add_scalar("loss/train", loss.item(), step)

    # Log weight and gradient distributions every 100 steps
    if step % 100 == 0:
        for name, param in model.named_parameters():
            writer.add_histogram(f"weights/{name}", param, step)
            if param.grad is not None:
                writer.add_histogram(f"grads/{name}", param.grad, step)

writer.close()
```

**How to read TensorBoard graphs**:

| What You See | What It Means | What To Do |
|-------------|---------------|-----------|
| Loss decreases smoothly | Training is healthy | Keep going |
| Loss flat from the start | Learning rate too low, or model can't learn | Increase learning rate by 10x |
| Loss oscillates wildly | Learning rate too high | Decrease learning rate by 10x |
| Loss goes to NaN | Numerical explosion | See NaN section above |
| Train loss ↓ but val loss ↑ | **Overfitting** — model memorized training data | Add regularization, get more data, or stop early |
| Weight histograms collapse to 0 | **Vanishing gradients** — deep layers stop learning | Use skip connections (ResNet), or batch normalization |
| Gradient histograms explode | **Exploding gradients** | Apply gradient clipping |

## The Complete Debugging Workflow

Use this checklist in order. It catches 95% of all AI bugs:

1. **Before training**: Run `check_shapes()` with one real batch. Verify every layer's input/output shape.
2. **Step 0-10**: Use `debug_print()` on loss, predictions, and labels. Confirm no NaN, values in sane ranges.
3. **Step 0-10**: Use `check_devices()` to confirm model and data are on the same device.
4. **During training**: Log loss and learning rate to TensorBoard. Watch the curve.
5. **When loss is suspiciously good**: Run `check_data_leakage()`. Verify your splits are clean.
6. **When loss goes NaN**: Use `detect_nan()` to find which layer's gradient exploded.
7. **When training is slow**: Use the `Timer` class to measure data loading vs forward vs backward. Profile memory if near OOM.

---
---

# Deep Dive 2: Data Management (From First Principles)

> *"Data is the fuel. How you manage it determines how fast you go."*

## Why Data Management Matters More Than Model Architecture

**First Principle**: A mediocre model trained on excellent data will outperform a brilliant model trained on bad data. **Every time.** This is one of the most counter-intuitive truths in AI. Beginners obsess over model architectures. Experts obsess over data quality.

Data management is the skill of getting the right data, in the right format, split the right way, flowing into your model at the right speed.

## How Data Flows in AI (The Mental Model)

```
Source                  Library              Local Storage        Your Model
(Hugging Face Hub) -->  (datasets) -->       (~/.cache/) -->      (DataLoader)
(Your CSV files)   -->  (pandas)   -->       (Parquet/Arrow) -->  (Training Loop)
(APIs)             -->  (requests) -->       (JSON) -->           
```

Every AI project follows this pipeline. The `datasets` library by Hugging Face standardizes steps 1-3 so you don't reinvent the wheel every time.

## File Formats: Why Parquet Exists (From First Principles)

**First Principle**: How you store data on disk determines how fast you can read it. Different formats make different trade-offs.

### CSV — The Format Everyone Knows (And Should Stop Using for AI)

A CSV file is plain text. Each row is a line, each column is separated by a comma.

```
name,age,score
Alice,30,0.95
Bob,25,0.87
```

**Why it's slow for AI**:
- To find column "score", the computer must read through "name" and "age" first — for every single row. This is called **row-oriented storage**.
- Everything is stored as text. The number `0.95` is stored as the characters `0`, `.`, `9`, `5` — 4 bytes of text instead of 4 bytes as a float32. Numbers must be parsed every time they're read.
- No compression. A 10GB dataset on disk is 10GB in memory.

### Parquet — The Industry Standard for AI Data

**First Principle**: Parquet uses **columnar storage**. Instead of storing data row-by-row, it stores data column-by-column.

```
Row-oriented (CSV):           Column-oriented (Parquet):
[Alice, 30, 0.95]             names:  [Alice, Bob, Charlie, ...]
[Bob, 25, 0.87]               ages:   [30, 25, 35, ...]
[Charlie, 35, 0.91]           scores: [0.95, 0.87, 0.91, ...]
```

**Why this is faster for AI**:
1. **Column selection**: If your model only needs "score", Parquet reads only the "score" column and skips the rest entirely. CSV must read every column of every row.
2. **Compression**: Within a column, all values are the same type (all integers, all floats). Similar values compress extremely well. A 10GB CSV often becomes a 2GB Parquet file.
3. **Native types**: Numbers are stored as actual binary numbers, not text. No parsing needed.
4. **Chunked reading**: Parquet files are split into "row groups." You can read 10,000 rows at a time without loading the entire file.

### Apache Arrow — The In-Memory Format

**First Principle**: Arrow is what Parquet becomes when loaded into RAM. The Hugging Face `datasets` library uses Arrow internally. When you call `load_dataset()`, the data is stored on disk in Arrow format in `~/.cache/huggingface/`. When you access it, Arrow maps it directly into memory without copying — this is called **zero-copy reads** and it's why `datasets` is fast even with huge files.

### Format Comparison Table

| Property | CSV | JSON | Parquet | Arrow |
|----------|-----|------|---------|-------|
| Storage | Row-oriented | Row-oriented | Column-oriented | Column-oriented |
| File size | Large | Large | Small (compressed) | Small |
| Read speed | Slow | Slow | Fast | Fastest |
| Human readable | Yes | Yes | No (binary) | No (binary) |
| Best use | Sharing with non-engineers | APIs, nested data | Storage & analytics | In-memory processing |

**The rule**: Store on disk as Parquet. Work in memory as Arrow. Export to CSV/JSON only when a human or an API needs to read it.

## Streaming: How to Handle Data Larger Than Your Disk

**First Principle**: A naive `load_dataset()` downloads the entire dataset to disk and then loads it into memory. For a 500GB dataset, you need 500GB of free disk space AND enough RAM to hold it. Most machines don't have this.

**Streaming solves this**. Instead of downloading everything, streaming fetches one row at a time from the server, processes it, and discards it:

```python
# Without streaming: downloads entire dataset to disk first
dataset = load_dataset("wikimedia/wikipedia", "20220301.en", split="train")
# ^ This downloads ~20GB before you can use any of it

# With streaming: processes row by row, constant memory usage
dataset = load_dataset("wikimedia/wikipedia", "20220301.en", split="train", streaming=True)
for i, example in enumerate(dataset):
    process(example)   # Each example is fetched on demand
    if i >= 1000:
        break          # You can stop whenever you want
```

**How it works under the hood**:
- The Hugging Face Hub stores datasets as Parquet files.
- When `streaming=True`, the library makes HTTP range requests — it asks for bytes 0-1000 of the file, processes those rows, then asks for bytes 1001-2000, and so on.
- Your RAM usage stays flat regardless of dataset size.

**When to use streaming**:
- Dataset is larger than your disk
- You only need a subset (first N examples)
- You want to preview a dataset before committing to a full download

**When NOT to use streaming**:
- You need random access (jumping to row 50,000)
- You'll iterate over the dataset multiple times (each iteration re-downloads)
- You need shuffling (streaming can only approximately shuffle)

## Data Splits: The Science of Not Lying to Yourself

**First Principle**: The entire purpose of machine learning evaluation is to estimate how well a model performs on **data it has never seen**. If the model has seen the test data during training, your accuracy metric is a lie.

### The Three Splits

| Split | Purpose | Typical Size | When To Use |
|-------|---------|-------------|-------------|
| **Train** | The model learns from this | 70-80% | Every training step |
| **Validation** | Tune hyperparameters (learning rate, model size) | 10-15% | After each epoch, to decide when to stop training |
| **Test** | Final, one-time evaluation | 10-15% | Only once, at the very end, to report your final number |

**Critical rule**: You **never** touch the test set during development. If you use the test set to make decisions ("this model got 89% on test, let me try a higher learning rate"), you are indirectly training on the test set. Your reported accuracy becomes meaningless.

### Creating Splits Correctly

```python
from datasets import load_dataset

dataset = load_dataset("stanfordnlp/imdb", split="train")

# Step 1: Split off the test set first
split = dataset.train_test_split(test_size=0.2, seed=42)

# Step 2: Split the remaining 80% into train and validation
train_val = split["train"].train_test_split(test_size=0.125, seed=42)
# 0.125 of 80% = 10% of total → validation set

train_ds = train_val["train"]    # 70% of total
val_ds = train_val["test"]       # 10% of total
test_ds = split["test"]          # 20% of total

print(f"Train: {len(train_ds)}, Val: {len(val_ds)}, Test: {len(test_ds)}")
```

**Why `seed=42`?** The seed controls the random number generator. Using the same seed guarantees the same split every time you run the code. This is **reproducibility** — if someone else runs your code, they get the exact same train/test split and can verify your results.

## Managing Large Files: Models and Datasets

**First Principle**: Git was designed for text files (code). A 14GB model checkpoint is not text. Pushing it to GitHub will fail, slow down every `git clone` forever, and bloat your repository.

### Three Strategies (In Order of Complexity)

**Strategy 1: `.gitignore` (Simplest — Use This First)**
Just tell Git to ignore large files. You can re-download them.

```
# .gitignore
*.pt
*.safetensors
*.bin
*.onnx
data/*.parquet
models/
```

**Strategy 2: Git LFS (When Teams Need Shared Models)**
Git LFS (Large File Storage) replaces large files with small pointer files in your repo. The actual data lives on a separate server.

```bash
git lfs install
git lfs track "*.safetensors"
git add .gitattributes
git commit -m "Track model weights with LFS"
```

**Strategy 3: DVC (When You Need Reproducible Experiments)**
DVC (Data Version Control) is like Git, but for data. It stores a `.dvc` pointer file in your repo and the actual data in cloud storage (S3, GCS).

```bash
dvc init
dvc add data/training_set.parquet
git add data/training_set.parquet.dvc
git commit -m "Track training data with DVC"
```

**For this course**: `.gitignore` is enough. Use DVC when you need to reproduce exact experiments across machines or share massive datasets with a team.

## The Hugging Face Cache System

Every time you call `load_dataset()` or download a model, it caches to `~/.cache/huggingface/`. After the first download, subsequent calls load from cache instantly.

```
~/.cache/huggingface/
├── datasets/          ← Cached datasets (Arrow format)
├── hub/               ← Cached model weights
└── modules/           ← Cached Python modules
```

**Warning**: This cache grows silently. A few models and datasets can consume 50+ GB. Monitor it:

```bash
du -sh ~/.cache/huggingface/*
```

---
---

# Deep Dive 3: Terminal & Shell (From First Principles)

> *"The terminal is where AI engineers live. Get comfortable here."*

## Why the Terminal Matters for AI

**First Principle**: When you train models, you will not be running code by clicking "Run" in VS Code. You will SSH into a remote Linux server with no GUI, no mouse, no file explorer. The terminal is your only interface. If you can't navigate, monitor, and manage processes from the command line, you are paying for idle GPU hours while googling "how to unzip a file in Linux."

## Piping: The Unix Philosophy (First Principles)

**First Principle**: Unix was built on one idea — **small tools that do one thing well, connected together via pipes**. Each tool reads text from `stdin` (standard input) and writes text to `stdout` (standard output). The pipe operator `|` connects the output of one tool to the input of the next.

```
command1 | command2 | command3
   ↓          ↓          ↓
 output → input/output → input → final result
```

This is like a factory assembly line. Each station does one job and passes the result to the next station.

### The Core Pipeline Tools for AI

**`cat`** — Dump a file's contents:
```bash
cat train.log
```

**`grep`** — Filter lines that match a pattern:
```bash
grep "loss" train.log              # Show only lines containing "loss"
grep -i "error" train.log          # Case-insensitive search
grep -c "epoch" train.log          # Count how many lines match
```

**`tail`** — Show the last N lines (or follow a live file):
```bash
tail -20 train.log                 # Last 20 lines
tail -f train.log                  # Follow live updates (Ctrl+C to stop)
```

**`awk`** — Extract specific columns:
```bash
# If your log has "epoch 5 loss: 0.342", extract just the loss value:
grep "loss:" train.log | awk '{print $NF}'
# $NF means "last field" → prints 0.342
```

**`wc`** — Count lines, words, or characters:
```bash
cat train.log | grep "loss" | wc -l    # How many lines contain "loss"?
```

**`sort`** — Sort lines:
```bash
grep "accuracy" results/*.log | sort -t= -k2 -n -r
# Sort by the 2nd field after "=", numerically, in reverse (highest first)
```

### Real AI Pipeline Examples

```bash
# Watch training loss update in real-time, filtering for just the loss:
tail -f train.log | grep --line-buffered "loss"

# Extract loss values and save them for plotting later:
grep "loss:" train.log | awk '{print $NF}' > losses.txt

# Compare two experiment results side-by-side:
diff <(grep "accuracy" exp1.log) <(grep "accuracy" exp2.log)

# Find the 20 largest model checkpoint files on your machine:
find . -name "*.pt" -o -name "*.safetensors" | xargs du -h | sort -rh | head -20
```

## Redirects: Controlling Where Output Goes

**First Principle**: Every program has three data streams:
- `stdin` (0) — where it reads input from (default: keyboard)
- `stdout` (1) — where it writes normal output (default: terminal screen)
- `stderr` (2) — where it writes error messages (default: terminal screen)

Redirects let you send these streams to files instead of the screen:

| Redirect | What It Does | Example |
|----------|-------------|---------|
| `>` | Write stdout to file (overwrites) | `python train.py > output.log` |
| `>>` | Append stdout to file | `echo "done" >> output.log` |
| `2>` | Write stderr to file | `python train.py 2> errors.log` |
| `2>&1` | Send stderr to same destination as stdout | `python train.py > all.log 2>&1` |
| `\|` | Send stdout of one command as stdin to next | `cat log.txt \| grep "loss"` |

**The most important AI pattern**:
```bash
python train.py > train.log 2>&1 &
# Runs training in background, captures ALL output to one file
```

## Background Processes: Training Doesn't Wait for You

**First Principle**: When you type a command, your shell waits for it to finish before giving you the prompt back. A training run takes hours. You need ways to run commands in the background.

### Three Levels of Background Execution

| Method | How | Survives Terminal Close? | Can Reattach? |
|--------|-----|-------------------------|---------------|
| `&` | `python train.py &` | ❌ No | ❌ No |
| `nohup` | `nohup python train.py > log.txt 2>&1 &` | ✅ Yes | ❌ No (check log file) |
| `tmux` | Start a tmux session, run inside it | ✅ Yes | ✅ Yes |

**Rule**: For anything that takes more than a few minutes, always use `tmux`.

## tmux: The Single Most Important Tool for AI Training

**First Principle**: `tmux` (terminal multiplexer) creates a **persistent session** on the server. You can:
1. Start a training run inside a tmux session
2. **Detach** from the session (the training keeps running)
3. Close your laptop, go home, sleep
4. SSH back into the server the next day
5. **Reattach** to the same session — your training is exactly where you left it

Without tmux, closing your SSH connection kills your training. With tmux, your training is immortal.

### tmux Workflow

```bash
# Create a named session
tmux new -s training

# You're now inside tmux. Run your training:
python train.py --epochs 100

# Detach: press Ctrl+B, then D
# (Ctrl+B is the "prefix key" — it tells tmux "the next key is a command")

# You're back to your normal terminal. The training keeps running.

# List all sessions:
tmux ls

# Reattach to your session:
tmux attach -t training

# Kill a session when you're done:
tmux kill-session -t training
```

### tmux Panes: Multiple Views at Once

This is where tmux becomes incredibly powerful. You can split your terminal into multiple panes and monitor everything at once:

```bash
# Start a session
tmux new -s work

# Split horizontally (top/bottom):
# Ctrl+B, then "

# Split vertically (left/right):
# Ctrl+B, then %

# Move between panes:
# Ctrl+B, then arrow keys

# Close a pane:
# Type 'exit' or press Ctrl+D
```

**The Ideal AI Training Layout**:
```
┌──────────────────────────┬──────────────────────┐
│                          │                      │
│  Pane 1: Training Run    │  Pane 2: GPU Monitor │
│  python train.py         │  watch -n1 nvidia-smi│
│  Epoch 12/100 ...        │  GPU: 85% | 14/24G   │
│                          │                      │
├──────────────────────────┴──────────────────────┤
│                                                  │
│  Pane 3: Live Log Monitoring                     │
│  tail -f logs/train.log | grep --line-buffered   │
│  "loss"                                          │
│                                                  │
└──────────────────────────────────────────────────┘
```

Three things running simultaneously. One terminal. You detach, go get coffee, come back. Everything is still running.

### tmux Quick Reference

| Action | Keystroke |
|--------|-----------|
| Create session | `tmux new -s name` |
| Detach | `Ctrl+B`, then `D` |
| Reattach | `tmux attach -t name` |
| List sessions | `tmux ls` |
| Split horizontal | `Ctrl+B`, then `"` |
| Split vertical | `Ctrl+B`, then `%` |
| Navigate panes | `Ctrl+B`, then arrow keys |
| Kill session | `tmux kill-session -t name` |
| Scroll up | `Ctrl+B`, then `[`, then arrow keys (press `q` to exit) |

## System Monitoring: Knowing What Your Machine Is Doing

**First Principle**: When training is slow, you need to determine whether the bottleneck is CPU, GPU, RAM, or disk I/O. Monitoring tools tell you this instantly.

### GPU Monitoring with `nvidia-smi`

```bash
nvidia-smi                     # Snapshot of GPU status
watch -n1 nvidia-smi           # Refresh every 1 second (live dashboard)
```

**How to read `nvidia-smi` output**:

| Field | What It Means | What To Watch For |
|-------|--------------|-------------------|
| GPU-Util | How busy the GPU cores are (%) | Low (< 30%) means the GPU is starving for data → data loading is the bottleneck |
| Memory-Usage | How much VRAM is used | Near max → reduce batch size or use mixed precision |
| Temperature | GPU temperature in °C | Above 85°C → thermal throttling (GPU slows itself down) |
| Processes | Which programs are using the GPU | Check that only YOUR training is using it |

### CPU/RAM Monitoring with `htop`

```bash
htop                           # Interactive process viewer
```

**`htop` keybindings**:
- `F6` or `>` — Sort by column (sort by memory to find leaks)
- `F5` — Toggle tree view (see parent/child processes)
- `F9` — Kill a process
- `/` — Search for a process by name
- `q` — Quit

## SSH: Working on Remote GPU Machines

**First Principle**: SSH (Secure Shell) creates an encrypted tunnel between your laptop and a remote server. Everything you type is sent to the server; everything the server outputs is sent back to your screen. It's as if you teleported your keyboard and monitor to the server room.

### Essential SSH Commands

```bash
# Connect to a remote machine:
ssh user@203.0.113.50

# Copy a file TO the server:
scp model.pt user@server:~/models/

# Copy a file FROM the server:
scp user@server:~/results/metrics.json ./

# Sync an entire directory (much faster than scp for many files):
rsync -avz --progress ./data/ user@server:~/data/

# Port forwarding (access remote Jupyter locally):
ssh -L 8888:localhost:8888 user@server
# Now open http://localhost:8888 in your browser
```

### SSH Config for Convenience

Instead of typing `ssh -i ~/.ssh/gpu_key ubuntu@203.0.113.50` every time, create `~/.ssh/config`:

```
Host gpu-box
    HostName 203.0.113.50
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes
```

Now you just type `ssh gpu-box`. That's it.

### The Complete Remote Training Workflow

```bash
# 1. SSH into your GPU box
ssh gpu-box

# 2. Start a tmux session
tmux new -s training

# 3. Activate your environment
source .venv/bin/activate

# 4. Start training
python train.py --epochs 100 --lr 1e-4 > train.log 2>&1

# 5. Split pane, monitor GPU
# Ctrl+B, " → watch -n1 nvidia-smi

# 6. Detach
# Ctrl+B, D

# 7. Close your laptop. Go home. Sleep.

# 8. Next day: SSH back in, reattach
ssh gpu-box
tmux attach -t training
# Your training is still running exactly where you left it.
```

---
---

# Deep Dive 4 (Honorable Mention): Docker for AI (From First Principles)

> *"Containers make 'works on my machine' a thing of the past."*

## Why Docker Is Especially Critical for AI

**First Principle**: AI projects have the most fragile dependency chains of any software. A typical AI stack includes:
- A specific Python version (3.11 vs 3.12)
- A specific PyTorch version (2.3 vs 2.6)
- A specific CUDA version (11.8 vs 12.4)
- A specific cuDNN version
- System-level C libraries compiled for a specific architecture
- Specialized packages like `flash-attn` that require exact compiler versions

If **any one** of these mismatches, your code either crashes or (worse) silently produces different results. Docker bundles ALL of these into a single image that runs identically on every machine.

## How Docker Works (Mental Model)

Think of Docker like shipping containers in the real world:
- Before containers: every port had different loading equipment, different truck sizes, different regulations. Shipping anything was a logistics nightmare.
- After containers: everything goes in a standard metal box. The box fits on any ship, any truck, any train. Nobody cares what's inside.

Docker does the same for software:
- **Image** = the recipe (your Dockerfile). A read-only blueprint.
- **Container** = a running instance of that recipe. Your actual kitchen.
- **Volume** = a shared folder between your computer and the container. This is how large files (models, datasets) persist across container restarts.

```
┌──────────────────────────────────────────────┐
│              Docker Container                 │
│                                              │
│  Ubuntu 22.04 + CUDA 12.4 + Python 3.12     │
│  + PyTorch 2.6 + transformers + your code    │
│                                              │
│  /workspace ←──── Volume ────→ Your local    │
│  /models   ←──── Volume ────→ ~/models/      │
└──────────────────────────────────────────────┘
               ↕ GPU passthrough
        Host GPU Driver (NVIDIA)
```

## The Dockerfile: A Layered Recipe

**First Principle**: A Dockerfile is a sequence of instructions. Each instruction creates a **layer**. Docker caches layers — if nothing changed in an instruction, Docker reuses the cached result. This means:
- Put things that change rarely (OS, CUDA) at the **top**.
- Put things that change often (your code, pip packages) at the **bottom**.
- This way, rebuilds are fast because Docker reuses the top layers.

```dockerfile
# Layer 1: Base OS + CUDA (rarely changes, ~4GB download, cached after first build)
FROM nvidia/cuda:12.4.1-devel-ubuntu22.04

# Layer 2: System packages (rarely changes)
RUN apt-get update && apt-get install -y python3.12 git curl

# Layer 3: PyTorch (changes when you upgrade versions)
RUN pip install torch==2.6.0+cu124 --index-url https://download.pytorch.org/whl/cu124

# Layer 4: Your dependencies (changes when you add new packages)
RUN pip install numpy pandas transformers datasets

# Layer 5: Your code (changes most frequently)
COPY . /workspace
WORKDIR /workspace
```

## Volumes: Why You Must Mount, Not Copy

**First Principle**: When a container stops, everything inside it **disappears** (unless you explicitly save it). A 14GB model you downloaded inside the container? Gone. Your training results? Gone.

Volumes solve this by creating a bridge between a folder on your real computer and a folder inside the container:

```bash
docker run --gpus all \
    -v $(pwd):/workspace \       # Your code directory → /workspace inside container
    -v ~/models:/models \         # Your models directory → /models inside container
    -v ~/data:/data \             # Your data directory → /data inside container
    ai-dev python train.py
```

Now your model downloads to `/models` inside the container, which is actually `~/models` on your real computer. Restart the container 100 times — the models are still there.

## Docker Compose: Multi-Service AI Applications

**First Principle**: A production AI application is never just a Python script. It's:
- An **inference server** running your model
- A **vector database** (like Qdrant) for RAG (Retrieval-Augmented Generation)
- Maybe a **web frontend**
- Maybe a **job queue** for batch processing

Docker Compose lets you define all of these in one YAML file and start them with one command:

```yaml
services:
  ai-dev:
    build: .
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    volumes:
      - ./:/workspace
      - ~/models:/models
    ports:
      - "8888:8888"

  qdrant:
    image: qdrant/qdrant:v1.12.5
    ports:
      - "6333:6333"
```

```bash
docker compose up -d      # Start everything
docker compose down        # Stop everything
```

The AI container can reach the vector database at `http://qdrant:6333` by name. Docker Compose creates a shared network automatically.

## Choosing the Right Base Image

| Base Image | Size | Use Case |
|-----------|------|----------|
| `nvidia/cuda:12.4.1-devel` | ~4 GB | Building packages from source (flash-attn, bitsandbytes) |
| `nvidia/cuda:12.4.1-runtime` | ~1.5 GB | Running pre-built code (inference, no compilation needed) |
| `pytorch/pytorch:2.6.0-cuda12.4-cudnn9-runtime` | ~6 GB | Skip PyTorch install entirely |
| `python:3.12-slim` | ~150 MB | CPU-only work, lightweight API servers |

**Rule of thumb**: Use `devel` during development (you'll need to compile things). Switch to `runtime` for production (smaller, faster startup).

---
---

### You Are Now Ready (For Real This Time)
You have now absorbed the 12 pillars of AI engineering setup AND deeply understood the four skills that you will use every single day. The setup modules (1-8, 11) are one-time tasks. But **Debugging, Data Management, Terminal mastery, and Docker** are lifelong skills that will compound with every project you build. Go forth and build.
