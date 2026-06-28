# YOYO AI

Zero-dependency AI coding assistant toolchain built on the [YOYO](https://github.com/openchat-ai/yoyo-ide) self-hosting x86_64 PE compiler.

## Goal

Ship a fully self-hosted AI code assistant in ~500 KB of binaries, running Qwen2.5-Coder-3B-Instruct (ternary-quantized) on commodity CPU hardware at 5–10 tok/s.

| Component | Target size |
|---|---|
| `yoyo-ai-tokenizer.exe` | ~40 KB |
| `yoyo-ai-quantize.exe` | ~60 KB |
| `yoyo-ai-inference.exe` | ~150 KB |
| `yoyo-ai-repl.exe` | ~50 KB |
| Model weights (`qwen2.5-coder-3b-yoyo.safetensors`) | ~600 MB |

## Status

**Week 0 — environment setup** ✅ complete (2026-06-28)

See [PLAN.md](./PLAN.md) for the full 12-week roadmap.

## Repository layout

```
yoyo-ai/
├── models/          # safetensors / GGUF weights
├── data/            # training / eval datasets
├── tests/           # test suites
├── docs/            # documentation & benchmark reports
├── PLAN.md          # 12-week execution plan
└── CURSOR_PROMPT.md # agent onboarding prompt
```

## Prerequisites

- [YOYO IDE](https://github.com/openchat-ai/yoyo-ide) compiler (`yoyo.exe`)
- Windows x86_64, 16+ CPU threads, 16+ GB RAM
- Python 3.10+ (dev tooling only — not a runtime dependency)

## Quick links

- [Baseline benchmark report](./docs/baseline-report.md)
- [12-week plan](./PLAN.md)

## License

Same as [yoyo-ide](https://github.com/openchat-ai/yoyo-ide).
