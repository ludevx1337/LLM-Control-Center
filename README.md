# LLM Control Center

**A Windows-first control plane for local and distributed LLM inference.**

LLM Control Center brings **llama.cpp, Colibri, LM Studio, Ollama and multi-PC GPU inference** into one desktop application.

Instead of manually figuring out which model, quantization or runtime will work on your hardware, LLM Control Center analyzes your machine and recommends a practical configuration based on:

* GPU / VRAM
* available RAM
* model architecture
* quantization
* context length
* Dense vs MoE architecture
* local and remote compute nodes
* supported inference runtimes

> Find the right model, choose the right runtime, distribute the workload, and benchmark the result — from one interface.

---

## Why LLM Control Center?

Running local LLMs is easy.

Running the **right model efficiently** is much harder.

A typical setup can involve:

* Hugging Face
* GGUF files
* llama.cpp
* LM Studio
* Ollama
* CUDA
* multiple GPUs
* multiple computers
* RPC workers
* MoE expert offloading
* RAM / VRAM limitations
* long-context KV cache
* different quantizations

LLM Control Center turns all of this into a single control plane.

```text
                 LLM CONTROL CENTER

                         │
          ┌──────────────┼──────────────┐
          │              │              │
     Model Catalog    Hardware       Runtime
          │           Detection       Manager
          │              │              │
          ▼              ▼              ▼
   Hugging Face      CPU / RAM      llama.cpp
   LM Studio         GPU / VRAM     Colibri
   Ollama            Remote GPUs    LM Studio
                                    Ollama
                         │
                         ▼
                    SMART FIT
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
      Recommended model       Recommended runtime
      Recommended quant       Memory strategy
      Context estimate        Distributed strategy
```

---

# Unified Model Catalog

Search models from a single interface.

LLM Control Center currently integrates:

### Hugging Face

Used as the primary remote model catalog.

The application can inspect:

* repository metadata
* GGUF variants
* safetensors
* quantizations
* model architecture
* parameter count
* MoE expert configuration
* maximum context
* base model information
* available shards

### LM Studio

Detect locally installed models and use LM Studio as an inference provider.

Models already installed locally can appear directly in Smart Search results.

Hugging Face variants can also be handed directly to LM Studio for download.

### Ollama

Detect models already available in the local Ollama library and expose Ollama as a runtime/provider.

---

# Smart Fit

Smart Fit estimates whether a model makes sense for the current hardware.

Example:

```text
Goal
Coding Agent

Context
40,960 tokens

Hardware

RTX 3060       12 GB VRAM
GTX 1070        8 GB VRAM
System RAM      64 GB
```

A result can look like:

```text
Qwen MoE 35B / 3B active

Architecture     MoE
Context          40K
Format           GGUF + Safetensors

Best Runtime     Colibri
Fit              91%

Memory Mode
VRAM + RAM + NVMe

Alternative
llama.cpp distributed
```

For another model:

```text
40B Dense / Q4

Best Runtime     llama.cpp
Mode             GPU + RAM
Distributed      Available

Fit              76%
```

Smart Fit currently considers:

* model parameters
* active MoE parameters
* quantization
* requested context
* local free VRAM
* configured remote VRAM
* available system RAM
* runtime compatibility
* GGUF availability
* Colibri model family support

The score represents **hardware/runtime fit**, not an arbitrary model quality ranking.

---

# llama.cpp Integration

LLM Control Center can manage llama.cpp directly.

Features include:

* GGUF model discovery
* metadata parsing
* GPU layer estimation
* context memory estimation
* CUDA builds
* Vulkan builds
* CPU builds
* llama-server management
* multiple simultaneous inference services
* RPC workers
* tensor split configuration
* remote node probing
* remote llama.cpp synchronization
* distributed inference

---

# Multi-PC Inference

Use GPUs located in different computers.

Example:

```text
MAIN PC

Intel Core i7
RTX 3060 12 GB
32 GB RAM

       │
       │ Ethernet / LAN
       ▼

WORKER PC

Intel Core i7
GTX 1070 8 GB
32 GB RAM
```

With llama.cpp RPC:

```text
RTX 3060
     │
     ├── model layers
     │
     └──────── LAN ──────── GTX 1070
                              │
                              └── additional layers
```

LLM Control Center can manage:

* RPC nodes
* worker status
* latency
* VRAM availability
* tensor split
* revision matching
* remote build/sync
* network-aware distribution

---

# Colibri MoE Runtime

LLM Control Center also integrates **Colibri** as a specialized Mixture-of-Experts runtime.

Colibri uses a memory hierarchy:

```text
        GPU VRAM
           │
      Hot Experts
           │
           ▼
          RAM
           │
      Warm Experts
           │
           ▼
          NVMe
           │
      Cold Experts
```

This makes it possible to experiment with models that would normally exceed available GPU or system memory.

The integration supports:

* Colibri source detection
* clone/update workflows
* Windows MSYS2 preparation
* CPU builds
* CUDA builds
* Colibri model container discovery
* compatible family detection
* Doctor validation
* RAM budget
* GPU expert cache budget
* GPU selection
* heat files
* OpenAI-compatible API
* expert workers
* cluster coordinator

---

# MoE Expert Cluster

Colibri workers are managed separately from llama.cpp RPC nodes.

Instead of splitting the entire model pipeline, remote machines can specialize in routed expert computation.

```text
                 Coordinator

             Routing / Dense State
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       Local GPU Experts   Remote Expert Worker

         RTX 3060              GTX 1070
            12 GB                 8 GB
```

---

# Playground

All supported providers can be accessed through a common interface.

Supported provider types:

* llama.cpp
* Colibri
* LM Studio
* Ollama
* generic OpenAI-compatible APIs

The Playground provides:

* model selection
* provider health checks
* chat
* tokens/sec benchmark
* latency information

---

# Local Model Library

LLM Control Center automatically indexes local models.

For GGUF models it can display:

* architecture
* quantization
* weight size
* estimated working memory
* context size
* recommended GPU layers

For Colibri containers:

* architecture
* layer count
* expert count
* model family
* container size
* compatibility hints

---

# Hardware Monitoring

Live Windows hardware information includes:

* CPU usage
* system memory
* GPU discovery
* VRAM
* NVIDIA utilization
* NVIDIA temperature
* available GPU memory

GPU discovery uses DXGI with additional NVIDIA information obtained through `nvidia-smi` when available.

---

# Desktop Architecture

LLM Control Center is designed as a lightweight native Windows application.

```text
Rust Backend
     │
     │ Tauri IPC
     ▼
Tauri 2
     │
     ▼
Microsoft WebView2
     │
     ▼
Vanilla HTML / CSS / JavaScript
```

There is:

* no Electron
* no bundled Chromium
* no React runtime
* no Vite runtime
* no remote frontend server
* no CDN dependency

Frontend assets are embedded directly into the executable.

---

# Interface

The application is organized into eight workspaces:

```text
Dashboard
Discover
Models
Runtime
Playground
Cluster
Logs
Settings
```

### Dashboard

Hardware telemetry and active inference services.

### Discover

Unified model catalog and Smart Fit.

### Models

Local GGUF and Colibri model library.

### Runtime

Launch and configure inference services.

### Playground

Unified chat and benchmarking.

### Cluster

Manage llama.cpp RPC and Colibri expert workers.

### Logs

Structured runtime and process logs.

### Settings

Runtime paths, providers, toolchains and Windows integration.

---

# Portable Mode

LLM Control Center can run as a portable Windows application.

```text
LLM-Control-Center.exe
portable.flag

data/
├── config.json
├── catalog-cache.json
└── models/
```

Configuration and models can remain beside the executable instead of being stored in the Windows user profile.

---

# Current Status

Current version:

**v0.5.0**

Implemented:

* ✅ Windows Tauri 2 application
* ✅ WebView2 premium UI
* ✅ Hugging Face model catalog
* ✅ Unified Smart Fit search
* ✅ LM Studio integration
* ✅ Ollama integration
* ✅ llama.cpp runtime
* ✅ llama.cpp RPC clustering
* ✅ GGUF metadata parser
* ✅ Colibri integration
* ✅ MoE model detection
* ✅ Colibri expert workers
* ✅ Hardware monitoring
* ✅ OpenAI-compatible Playground
* ✅ Runtime benchmarking
* ✅ Portable Windows packaging
* ✅ Local model catalog cache

---

# Project Vision

The goal of LLM Control Center is not to become another model launcher.

The goal is to answer a more useful question:

> **What is the best practical way to run this model on the hardware I actually own?**

The application is being built around automatic decisions such as:

```text
Model
   ↓
Architecture Analysis
   ↓
Hardware Analysis
   ↓
Runtime Compatibility
   ↓
Memory Planning
   ↓
Distributed Compute Planning
   ↓
Recommended Configuration
```

Eventually, selecting a model should be as simple as:

```text
Goal: Coding Agent
Context: 40K
Priority: Balanced
```

and letting the Control Center determine:

* which model variant to download
* which quantization to use
* which runtime to use
* how much VRAM/RAM is required
* whether distributed inference is useful
* which GPU should host which workload
* the expected performance based on previous local benchmarks

---

# Development Status

LLM Control Center is currently under active development.

Some advanced features — especially heterogeneous multi-PC inference, MoE expert distribution and performance prediction — should still be considered experimental.

Benchmark results depend heavily on:

* GPU architecture
* memory bandwidth
* network bandwidth
* model architecture
* quantization
* context length
* runtime version

---

# License

License information will be added before the first public release.

---

## Built With

**Rust · Tauri 2 · WebView2 · llama.cpp · Colibri · Hugging Face · LM Studio · Ollama**


THIRD_PARTY_NOTICES

llama.cpp
MIT License
https://github.com/ggml-org/llama.cpp

Colibri
Apache License 2.0
https://github.com/JustVugg/colibri

Tauri
MIT OR Apache-2.0
https://github.com/tauri-apps/tauri

LM Studio
External optional integration.
LM Studio is not distributed with LLM Control Center.

Ollama
External optional runtime.

Models
Model weights are not part of the LLM Control Center license.
Each model remains subject to its publisher's license.

LICENSE
→ Apache License 2.0
→ Copyright 2026 LudevX
