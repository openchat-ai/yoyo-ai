# 明天回来先看这个

## 当前状态（2026-06-28 下班）

### ✅ 已完成
- Git 仓库初始化（commit `d88bc27`）
- GitHub issue 模板、CI workflow
- 目录结构（data/ docs/ models/ tests/ tools/）
- llama.cpp 下载到 `tools/llama.cpp/`
- 备份目录 `F:\yoyo-ide-backup-2026-06-28\`
- README.md 写好

### ❌ 未完成
- **Qwen 模型下载**：huggingface.co 在国内不通，我已经杀掉卡住的 Python 进程
- **Baseline benchmark**：等模型下完
- **Task 0.3 备份**：✅ 已完成（上面那个备份目录）

### ⚠️ 需要修复的问题
1. **huggingface.co 不可达**（国内网络问题）
2. **下载卡在 0 字节 17 分钟**（已终止）

## 明天回来要做的 3 件事

### 1. 升级 Cursor Pro（强烈建议）
- 现在 22% 用量
- Week 1 开始后消耗会很大
- $20/月，链接 https://cursor.com

### 2. 在 Cursor 聊天里发这条指令

```
之前的 Qwen 模型下载因为 huggingface.co 在国内不通卡住了。
请改用 HF 镜像 hf-mirror.com：
1. 设置环境变量: set HF_ENDPOINT=https://hf-mirror.com
2. 重新下载: huggingface-cli download Qwen/Qwen2.5-Coder-3B-Instruct-GGUF --include "qwen2.5-coder-3b-instruct-q4_k_m.gguf" --local-dir F:\yoyo-ide\yoyo-ai\models
3. 下载完成后跑 baseline benchmark:
   F:\yoyo-ide\yoyo-ai\tools\llama.cpp\llama-bench.exe -m F:\yoyo-ide\yoyo-ai\models\qwen2.5-coder-3b-instruct-q4_k_m.gguf -p 256 -n 30
4. 把结果写到 docs/baseline-report.md
5. 完成 Week 0 报告
```

### 3. 如果 Cursor Agent 还是不行

回我，我手动帮你跑下载。

## 备选方案

如果 hf-mirror.com 也不行，可以试：
- 国内魔搭社区：https://www.modelscope.cn/
- 直接问我要完整的 PowerShell 下载脚本

## 文件位置速查

```
F:\yoyo-ide\
├── yoyo.exe              当前 YOYO 编译器
├── yoyo.js, yoyo-gen.js  YOYO 源码
├── projects\yoyo.ty      YOYO 编译器源码
└── yoyo-ai\              ← 新项目（Cursor 在这里工作）
    ├── PLAN.md
    ├── CURSOR_PROMPT.md
    ├── README.md
    ├── models\           ← 模型下载目录
    └── tools\llama.cpp\  ← llama.cpp 已就绪

F:\yoyo-ide-backup-2026-06-28\   ← 完整备份
```

## 下班前的工作

- ✅ git commit（保存当前进度）
- ✅ 创建本状态文件
- ✅ 终止卡住的下载进程
- ⏸️ Cursor Agent 暂停（等你明天回来）