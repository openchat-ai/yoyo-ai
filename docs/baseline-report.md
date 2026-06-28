# Baseline Benchmark Report

**Date:** 2026-06-28  
**Task:** Week 0 Task 0.2 — establish CPU inference baseline before YOYO AI toolchain development

## Environment

| Item | Value |
|---|---|
| **CPU** | Hygon C86-3G (OPN:3350), 16 logical threads |
| **RAM** | 40 GB total (~15 GB free at test time) |
| **OS** | Windows 10 x64 |
| **Target platform (plan)** | Intel Xeon E5-2680 v3, 16–32 GB RAM |
| **Note** | This machine is Hygon x86_64, not E5-2680 v3. Numbers are indicative; re-run on target E5 before Week 5 decisions. |

## Model

| Item | Value |
|---|---|
| **Model** | Qwen2.5-Coder-3B-Instruct |
| **Quantization** | Q4_K_M (GGUF) |
| **Source** | `bartowski/Qwen2.5-Coder-3B-Instruct-GGUF` |
| **File** | `models/Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf` |
| **File size** | 1.93 GB on disk (1.79 GiB in GGUF metadata) |
| **Parameters** | 3.09 B |

## Runtime

| Item | Value |
|---|---|
| **Engine** | llama.cpp b9829 (`llama-b9829-bin-win-cpu-x64`) |
| **Backend** | CPU (ggml-cpu-haswell) |
| **Threads** | 16 (`-t 16`) |
| **Context** | 32768 (model default; KV cache pre-allocated) |

## Results

### llama-bench (3 runs, `-p 64 -n 30 -r 3`)

| Test | Throughput |
|---|---|
| Prompt processing (64 tokens) | **39.90 ± 0.30 tok/s** |
| Text generation (30 tokens) | **9.72 ± 0.09 tok/s** |

### llama-cli end-to-end (prompt file, 30 new tokens)

| Metric | Value |
|---|---|
| **Prompt** | `Write a Python hello world function.` (36 prompt tokens) |
| **Prompt eval** | 966 ms → **37.26 tok/s** |
| **Generation (30 tokens)** | **3028 ms → 9.91 tok/s** |
| **Wall time (generation only)** | ~3.0 s for 30 tokens |

### Memory

| Component | Size |
|---|---|
| Model weights (CPU mapped) | ~1823 MiB |
| Weight repack buffer | ~1266 MiB |
| KV cache (32768 ctx) | ~1152 MiB |
| Compute buffer | ~113 MiB |
| **Total host memory (llama.cpp estimate)** | **~1833 MiB** working set for inference |

## Interpretation vs. project targets

| Target (PLAN.md) | Baseline | Status |
|---|---|---|
| 5–10 tok/s dialogue speed | **9.7–9.9 tok/s** (Q4_K_M, llama.cpp optimized CPU) | ✅ Meets target on this CPU |
| ~600 MB ternary model | 1.93 GB Q4_K_M GGUF | Expected — ternary PTQ is Week 5+ |
| E5-2680 v3 platform | Tested on Hygon C86-3G | ⚠️ Re-benchmark on E5 recommended |

**Conclusion:** Qwen2.5-Coder-3B-Instruct at Q4_K_M already hits the 5–10 tok/s goal on a 16-thread CPU with llama.cpp. The YOYO toolchain must reach similar speed with ternary weights (~600 MB) and zero external dependencies. This baseline confirms the 3B model size is viable.

## Reproduce

```powershell
# Download model (once)
curl -L -o models/Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf `
  "https://hf-mirror.com/bartowski/Qwen2.5-Coder-3B-Instruct-GGUF/resolve/main/Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf"

# Benchmark
tools/llama.cpp/llama-bench.exe -m models/Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf -t 16 -p 64 -n 30 -r 3

# Interactive generation (30 tokens)
tools/llama.cpp/llama-cli.exe -m models/Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf `
  -f docs/benchmark-prompt.txt -n 30 -t 16 --no-display-prompt
```

Raw logs: `docs/benchmark-bench.txt`, `docs/benchmark-cli.err`
