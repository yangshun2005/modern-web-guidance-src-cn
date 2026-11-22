# 评估框架与 Agent 配置

本文档提供在受支持的 AI 编码 Agent 上运行评估框架的完整说明与配置细节。

## 概述

评估框架衡量 AI 编码 Agent 采用现代 Web API 的效果。它在多种 Agent 框架上运行自动化基准测试，并通过基于结果的 Playwright 检查验证其实现。

受支持的 Agent 定义在 [`src/harness/config.ts`](../src/harness/config.ts) 中的 `Agents` 对象里。

---

## Agents

### Claude Code

Claude Code Agent（`claude_code`）基于 [Claude Code on Vertex AI](https://code.claude.com/docs/en/google-vertex-ai) 实现。

**配置：**
1. 使用 `gcloud` 登录并设置活动项目 ID：
   ```bash
   gcloud config set project <YOUR-GCP-PROJECT-ID>
   ```
2. 确保你的 GCP 项目已启用 Vertex AI API，并在 Model Garden 中启用了所需模型。
3. 在 `.env` 文件或会话中设置以下环境变量：
   ```bash
   CLAUDE_CODE_USE_VERTEX=1
   CLOUD_ML_REGION=global
   ANTHROPIC_VERTEX_PROJECT_ID=<YOUR-GCP-PROJECT-ID>
   ANTHROPIC_MODEL=<enabled-model-in-vertex>
   # Optional: reasoning effort (`--effort <level>`); defaults to Claude Code's built-in effort when unset (captured as `observed` from `api-request` or `assistant` logs when present).
   # Cannot be combined with a model ID that already ends in a level suffix (e.g., `opus-5.5-high`).
   # CLAUDE_EFFORT=high
   ```

---

### Codex CLI

使用 Codex CLI Agent 需要申请例外，尝试使用时会弹出该申请：

```bash
node_modules/.bin/codex
```

1. 该申请应提交一个类似 [`b/492300931`](https://b.corp.google.com/issues/492300931) 的 bug。
2. 审批通过后，在本地重启 CLI 并登录你的账号。
3. 在环境中设置首选模型（以及可选的推理强度）：
   ```bash
   CODEX_MODEL='gpt-5.5'
   # Optional: reasoning effort (`-c model_reasoning_effort=<level>`); defaults to `~/.codex/config.toml` / CLI default when unset.
   # Cannot be combined with a model ID ending in a level suffix.
   # CODEX_EFFORT=high
   ```

---

### Jetski CLI

Jetski CLI（`jetski_cli`）可通过 `GD_DEV_USE_JETSKI` 标志用于指南开发工作流（`gd dev`）。

**配置：**
为 Jetski CLI Agent 运行配置首选模型：
```bash
# Model selection for Jetski CLI agent runs
JETSKI_MODEL='Gemini 3.8 Flash (Medium)'
# Optional: reasoning effort (`--effort <level>`) when using a base model ID without a level suffix; defaults to Jetski CLI's built-in effort.
# JETSKI_EFFORT=high
GD_DEV_USE_JETSKI=1  # Required to use Jetski CLI for 'gd dev'
```

---

### Antigravity CLI

Antigravity CLI（`antigravity_cli`）是指南开发工作流（`gd dev`）使用的默认 Agent。

**安装：**
必须先安装 Antigravity CLI（`agy`）：
```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```
然后交互式运行一次 `agy` 完成登录。

**配置：**
所有设置均为可选。若使用个人 Antigravity 账号登录，则无需 GCP 项目。若通过 GCP 项目使用 agy，请在 `~/.gemini/antigravity-cli/settings.json`（`gcp.project`）中设置，或在 `.env` 文件中覆盖：
```bash
# Optional: GCP project (only for GCP-project auth)
ANTIGRAVITY_GCP_PROJECT=<YOUR-GCP-PROJECT-ID>
# Optional: path to the agy binary if it is not on your PATH (e.g. under cron)
ANTIGRAVITY_CLI_BIN=/home/<you>/.local/bin/agy
# Optional: model override for Antigravity CLI agent runs
ANTIGRAVITY_MODEL=gemini-3.8-flash-medium
# Optional: reasoning effort (`--effort <level>`); defaults to agy's built-in effort when unset.
# Cannot be combined with a tiered model ID that already ends in a level suffix (like `gemini-3.8-flash-medium`).
# ANTIGRAVITY_EFFORT=high
```

---

### 推理强度记录与冲突规则

每次试验（`trajectory_summary.json`）和任务（`evals.json`）都会在 `model` 旁记录 `reasoningEffort: { level?: string, source: 'observed' | 'configured' | 'model-id' | 'agent-default' }`（优先级：`observed > configured > model-id > agent-default`）：

- **Antigravity CLI（`ANTIGRAVITY_EFFORT`）**、**Jetski CLI（`JETSKI_EFFORT`）** 和 **Claude Code（`CLAUDE_EFFORT`）** 传递 `--effort <level>`。
- **Codex CLI（`CODEX_EFFORT`）** 传递 `-c model_reasoning_effort=<level>`。
- **Pi（`PI_MODEL` / `PROMPT_MODEL`）** 通过模型 ID 上的 `:<thinking>` 后缀接受思考级别（例如 `openrouter/deepseek/deepseek-v4.1-flash:low` 或 `...:off`）。
- **启动冲突检查：** 若同时设置 `<AGENT>_EFFORT` 和已带 `-<level>` 或 `:<level>` 层级后缀的模型 ID（如 `gemini-3.8-flash-medium` 或 `opus-5.5-high`），启动时会报错，而不是传递冲突的标志。若既无 effort 环境变量也无模型 ID 层级，且日志未报告观测到的层级，则 `reasoningEffort` 记录为 `{ source: 'agent-default' }`。