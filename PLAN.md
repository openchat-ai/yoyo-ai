# YOYO AI 工具链：12 周执行计划

## 总览

| 维度 | 值 |
|---|---|
| **目标** | 全球第一个完全自托管、零依赖、~500KB 大小的 AI 代码助手工具链 |
| **核心模型** | Qwen2.5-Coder 3B-Instruct（ternary 量化后 ~600MB）|
| **运行平台** | E5-2680 v3（16 核 32 线程，16-32GB RAM）|
| **目标速度** | 5-10 tok/s（对话级）|
| **代码量** | ~5500 行 yoyo + ~500 行 JS |
| **最终产物** | 6 个 .exe 文件，总计 ~500KB |

## 角色分工

| 角色 | 做什么 | 时间投入 |
|---|---|---|
| **Cursor Agent** | 写代码、跑测试、debug、commit、写文档 | 自动化 |
| **你（决策者）** | 给指令、看 demo、做关键决策、批准发布 | 每天 15-30 分钟 |
| **GPU 算力** | 一次性 LoRA 微调 | ¥10-50 一次性 |

## 成本预估

| 项目 | 月成本 | 12 周总成本 |
|---|---|---|
| Cursor Pro（可能需要 Business $40）| $20-40 | $60-120 |
| AutoDL GPU 微调 | 一次性 | ¥10-50 |
| DeepSeek API（辅助决策）| ¥50-100 | ¥150-300 |
| **总计** | — | **¥600-1500** |

---

## Week 0：环境准备

### 任务
- Task 0.1：创建 GitHub repo，初始化 README、issue 模板、CI
- Task 0.2：跑 baseline benchmark（llama.cpp + Qwen2.5-Coder-3B-Instruct-Q4_K_M.gguf 在 E5 上）
- Task 0.3：备份当前 yoyo 仓库

### 验收
- baseline-report.md 存在
- 仓库有完整 README

---

## Week 1-2：YOYO 内功（Phase 2 关键 opcode）

### Week 1：补 5 个最关键 opcode
- Task 1.1：opcode 0x84 (memcpy, 8B)
- Task 1.2：opcode 0x85 (memcpy state, 23B)
- Task 1.3：opcode 0x62 (sub imm, 10B)
- Task 1.4：opcode 0x68 (add reg, 9B)
- Task 1.5：opcode 0x55 (store u32, 11B)
- 每个 opcode 后跑 bootstrap-check.sh --strict --lock

### Week 2：补剩余 opcode + I/O 强化
- Task 2.1-2.5：opcode 0x61, 0x69, 0x66, 0x67, 0x60
- Task 2.6：强化文件 I/O emit

### 验收
- 17 个新 opcode 全部实现
- yoyo.ty 体积增长 ~30%
- bootstrap-check 全绿

---

## Week 3-4：YOYO Tokenizer（模型无关）

### Week 3：BPE encoder
- Task 3.1：写 Python BPE 参考实现（~50 行）
- Task 3.2：逐行翻译成 yoyo（~300 行 yoyo）
- Task 3.3：测试用例
- Task 3.4：验证 YOYO 输出 vs Python bit-exact

### Week 4：加载真实 Qwen vocab
- Task 4.1：vocab.json 解析器
- Task 4.2：处理特殊 token（ChatML 模板）
- Task 4.3：实现 decode（token IDs → 文本）
- Task 4.4-4.5：集成测试，对比 HuggingFace tokenizer

### 验收
- YOYO tokenizer 能处理 1000 个测试 prompt
- 输出 bit-exact HuggingFace tokenizer

### Week 4 demo
- YOYO tokenizer 处理"写一个 Python hello world" → 输出 token IDs → decode 回来一致

---

## Week 5-6：推理 v1（标量，能跑）

### 关键决策：模型尺寸
**推荐 Qwen2.5-Coder 3B**（ternary 后 ~600MB，~5-10 tok/s，~80% HumanEval）

### Week 5
- Task 5.1：写 Python PTQ 脚本（GPTQ-for-BitNet）
- Task 5.2：跑 PTQ：Qwen2.5-Coder-3B FP16 → ternary
- Task 5.3：用 llama.cpp + ternary 跑 5 个测试 prompt 验证
- Task 5.4：实现 safetensors 解析器
- Task 5.5：实现 packed trit 解码（2 bits → trit）
- Task 5.6：写 matmul handler 的 yoyo emit（标量）

### Week 6
- Task 6.1-6.4：实现 RMSNorm、RoPE、attention、SwiGLU FFN
- Task 6.5：KV cache 管理
- Task 6.6：sampling（top-k、temperature）
- Task 6.7：跑通完整前向传播

### 验收
- 推理 v1 能跑通
- 速度 ~0.3 tok/s

### Week 6 demo ⭐
- E5 上输入 prompt
- YOYO 输出 token（即使是垃圾）
- **第一次端到端 demo**

---

## Week 7-8：加速（SIMD + 多线程）

### Week 7
- Task 7.1：packed trit encoding（2 bits per trit）
- Task 7.2：SIMD matmul（vpternlogd / vpmaddubsw / vpdpwssd）
- Task 7.3-7.4：整合 + 性能对比
- Task 7.5：如果 SIMD 加速 < 2x，回退标量

### Week 8
- Task 8.1：CreateThread / WaitForSingleObject emit
- Task 8.2：线程间同步（barrier）
- Task 8.3-8.4：多线程 matmul + 测试
- Task 8.5：如果多线程 < 1.5x，回退单线程

### 验收
- 推理速度 ≥ 5 tok/s

### Week 8 demo ⭐⭐
- E5 上跑 prompt
- **速度 ≥ 5 tok/s**
- **第一个对话级 demo**

---

## Week 9：Quantizer + REPL

### Week 9a
- Task 9.1-9.3：跑 GPTQ-for-BitNet，生成 ternary 权重，验证质量

### Week 9b
- Task 9.4：Python PTQ 翻译成 yoyo
- Task 9.5：YOYO safetensors 写出器
- Task 9.6-9.7：bit-exact 验证 + 跑推理

### Week 9c
- Task 9.8-9.11：REPL（readline + 流式输出 + 代码高亮 + ChatML 模板）

### 验收
- YOYO quantizer 独立完成 FP16 → ternary
- REPL 能跑多轮对话
- 速度 ≥ 5 tok/s

### Week 9 demo ⭐⭐⭐
- 完整 AI 编程助手可用

---

## Week 10-12：整合 + 发布

### Week 10
- Task 10.1-10.2：测试套件 + benchmark
- Task 10.3-10.4：错误处理 + 100 个 prompt 质量评估
- Task 10.5：修 bug

### Week 11
- Task 11.1-11.5：README、QUICKSTART、ARCHITECTURE、BENCHMARK、DEMO_VIDEO

### Week 12
- Task 12.1-12.5：录视频 + GitHub Release + 知乎/掘金博客 + B 站视频

### Week 12 demo ⭐⭐⭐⭐
- **全球第一个完全自托管 AI 代码助手工具链发布**

---

## 决策点汇总

| Week | 决策点 | 你的动作 |
|---|---|---|
| 0 | baseline 是否符合预期 | 看 benchmark 数字 |
| 2 | Phase 2 是否继续 | 批准 Week 3 |
| 4 | Tokenizer 输出是否一致 | 看测试报告 |
| 4 | 模型尺寸选择（1.5B / 3B / 7B）| 选一个 |
| 5 | ternary 权重是否生成成功 | 批准继续 |
| 6 | 推理 v1 是否跑通 | 看 demo |
| 7 | SIMD 加速是否足够（≥ 2x）| 决定继续优化 |
| 8 | 速度是否达到 5 tok/s | 决定进 Week 9 |
| 9 | YOYO quantizer 输出是否一致 | 批准继续 |
| 10 | 100 个 prompt 质量如何 | 决定发布 |
| 12 | 是否发布 v1.0 | 点 Release |

---

## 风险预案

| 风险 | 触发 | 应对 |
|---|---|---|
| Phase 2 延期 | Week 2 末未通过 bootstrap-check | 暂停 opcode，专注 tokenizer |
| SIMD 没效果 | Week 7 加速 < 2x | 跳过 SIMD，接受 1-2 tok/s |
| 多线程更慢 | Week 8 加速 < 1.5x | 跳过多线程 |
| 质量不达标 | Week 10 HumanEval < 50% | 换 Qwen 1.5B |
| 整体延期 | 12 周只完成 70% | 接受 MVP，v1.1 补完 |

---

## 完整产出清单

### 二进制（~500KB）
```
yoyo.exe                      ~80KB
yoyo-ai-tokenizer.exe         ~40KB
yoyo-ai-quantize.exe          ~60KB
yoyo-ai-inference.exe         ~150KB
yoyo-ai-repl.exe              ~50KB
```

### 模型权重
```
qwen2.5-coder-3b-yoyo.safetensors  ~600MB
```

### 代码
```
yoyo-gen.js                  ~2000 行
yoyo.js                       ~450 行
encode-x64.js                 ~250 行
projects/yoyo.ty              ~1500 行
projects/yoyo-ai/*.ty         ~3500 行
```

### 文档
```
README.md / QUICKSTART.md / ARCHITECTURE.md / BENCHMARK.md / DEMO_VIDEO.md
```

---

## 你的实际投入

| 时间 | 投入 |
|---|---|
| 每天 | 15-30 分钟（看 demo + 决策）|
| 每周 | 2-3 小时 |
| 12 周 | 24-36 小时 |

---

## 立即开始

1. 订阅 Cursor Pro（$20）→ https://cursor.com
2. 把 `CURSOR_PROMPT.md` 整个粘贴给 Cursor Agent
3. Cursor Agent 会读 `PLAN.md`，开始 Week 0