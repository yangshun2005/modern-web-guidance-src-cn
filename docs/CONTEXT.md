# 现代 Web 指南项目 — 上下文文档

*（注：本文档是由 LLM 自动维护的上下文文档，旨在向在本仓库中工作的 AI 代理提供项目总体目标、架构和工作流细节。它不用于替代面向人类贡献者的 README，而是以"全局视角"的状态作为补充。AI 代理在工作过程中须更新本文件。）*

本文档描述现代 Web 指南项目的目标、架构、贡献者工作流和当前状态。它既作为 LLM 上下文（用于后续 AI 辅助工作），也作为人类可读的项目概览。

最后更新：2026-11-02。

---

## 1. 项目简介

**现代 Web 指南**是一个 Google Chrome 项目，由领域专家（SME）为现代 Web 平台特性（CSS、JS API、HTML）编写精选指南。这些指南通过 Agent Skills 和 CLI 提供给 AI 编码代理，使开发者要求 AI 工具实现某项功能时，代理产出的代码采用现代最佳实践而非过时模式。项目有两个相互交织的目标：

1. **创建高质量指南** — 结构化的 Markdown 文档，教会编码代理正确使用现代 Web 特性。
2. **证明指南有效** — 一套评估工具，衡量能访问指南的代理是否比无法访问的代理产出更好的结果。

### 相关人员（见 [`GOVERNANCE.md`](../GOVERNANCE.md) 与 [`docs/ATLS.md`](./ATLS.md)）

- **贡献者**：通过 Issue 提出用例、提交 PR 修复缺陷、更新文档、改进现有指南的社区成员。
- **同行（Peers）**：已入职的领域专家，负责撰写全新的 Stage 1–2 指南（`guide.md`、`demo.html` 和 `expectations.md`）并进行同行评审。
- **内容领域技术负责人（Content ATL）**：领域专家（[`.github/atls.json`](../.github/atls.json)、[`docs/ATLS.md`](./ATLS.md)），负责管理特定类别或特性横向领域，验证 Stage 1 用例，评审/批准 Stage 2 指南 PR，并与工程团队合作分诊内容相关的评估失败。
- **所有者与基础设施工程师**：治理项目并维护 Stage 3 评估基础设施（`gd` CLI、评估工具、目标/评分器生成流水线、分发服务、仪表盘）。

这一边界是刻意划分的：SME 和 Content ATL 专注于 Web 平台的准确性（`guide.md`、`demo.html`、`expectations.md`），无需编写或调试 Playwright 测试基础设施；而 `gd dev` 流水线负责将期望转化为经过校准的目标评估。

### 仓库结构

```
modern-web-guidance-src/
  AGENTS.md                   # AI 代理的入口说明与技能路由
  CONTRIBUTING.md             # 贡献者入门与路径路由
  GOVERNANCE.md               # 贡献者角色、撰写权限与治理
  guides/                     # 所有指南内容，按类别组织
    performance/              # 例如 batch-analytics-events、optimize-image-priority
    ui-components/            # 例如 modal-dialog、navigation-drawer
    css/                      # 例如 animate-to-intrinsic-sizes、size-aware-styling
    accessibility/            # 例如 accessibility、accessible-error-announcement
    security/                 # 例如 verify-email-ownership
    CONTRIBUTING.md           # 撰写与验证 Stage 1–2 指南的工作流
    README.md                 # 指南目录概览与链接
  features/                   # 特性定义与用于嵌入的代码片段
  skills-src/                 # 独立 Agent Skills 的源文件
  src/
    authoring/                # 核心编排（gd dev 流水线、指南完整性）
    build/                    # 构建脚本与发布版本生成器
    ci/                       # CI 验证、ATL 分诊与流水线同步脚本
    cli/                      # Skills CLI 实现与遥测
    core/                     # 共享路径、配置与指南验证
    dashboard/                # 评估仪表盘与服务器
    grading/                  # 基于 Playwright 的评分引擎与评分器生成
    harness/                  # 评估工具、代理运行器、基础应用、夜间脚本
    rag/                      # 语义搜索、嵌入与向量存储
  bin/gd.ts                   # 统一 CLI 入口点
  docs/                       # 上下文、ATL、评估与发布文档
  dist/                       # 构建后的分发包（已 gitignore）
  out/                        # 临时构建产物（已 gitignore）
  results/                    # 指南与任务评估结果（已 gitignore）
```

---

## 2. 指南产物流水线（见 [`guides/CONTRIBUTING.md`](../guides/CONTRIBUTING.md)）

每个指南位于独立目录（如 `guides/performance/batch-analytics-events/`），包含 SME 撰写的指南，以及跨 `SUPPORTED_BASE_APPS`（`daily-grind`、`devtools-times`）的目标评估胶囊。

### 每个指南目录中的文件

| 文件 | 作者 | 用途 |
|---|---|---|
| `guide.md` | SME（人类） | 指南本身，由编码代理通过 Skills 读取（`SKILL.md` 保留给入口点和独立技能）。规范性规则与构建时宏（`BASELINE_STATUS`、`INCLUDE`、`FEATURE`、`FEATURE_FALLBACKS`、`FEATURE_ISSUES`、`GUIDE_REF`）在 [`project-guides`](../.agents/skills/project-guides/SKILL.md) 中规定。 |
| `demo.html` | SME（人类） | 该用例的黄金标准独立实现。 |
| `expectations.md` | SME（人类） | 自然语言的项目符号列表，列出可观察的检查项，作为评分器生成的输入（见 [`project-guides`](../.agents/skills/project-guides/SKILL.md) 与 [`project-evals`](../.agents/skills/project-evals/SKILL.md)）。 |
| `targets/<base_app>/patches/` | 生成（`gd dev`） | 多代理解决方案补丁（`antigravity-solution.patch` / `jetski-solution.patch`、`claude-solution.patch`、`codex-solution.patch`）与基线补丁（`zero-passrate.patch`，几乎所有指南都为空，以便校准针对未改动的起始应用运行）。用于评分器校准。 |
| `targets/<base_app>/grader.ts` | 生成（`gd dev`） | Playwright 测试文件，依据期望对基础应用评分。经过校准，使每个黄金补丁通过所有测试，而未改动的起始应用（`zero-passrate.patch`）不通过任何测试，除非它已满足某个 `MUST NOT` 检查（与一个正向存在性检查配对）。 |
| `targets/<base_app>/task.md` | 生成（`gd dev`） | 针对 `<base_app>` 定制的单条 `- ` 项目符号开发者提示，提供给评估代理（无 YAML frontmatter）。 |
| `results/guides/<category>/<slug>/report.md` | 生成（`gd dev`） | 自动化评估诊断报告，分析通过率和工具消耗并给出可操作建议。 |

### 学科指南

大多数指南是基于任务的用例。学科指南作为某个类别的概念性"枢纽"，通过 `{{ GUIDE_REF("guide-slug") }}` 链接到其用例子指南（见 [`project-discipline-guides`](../.agents/skills/project-discipline-guides/SKILL.md)）。若指南是位于 `guides/<category>/<category>/guide.md` 的类别根指南（如 `guides/css/css/guide.md`），或是在 `src/core/guide-validation.ts` 的 `DISCIPLINE_GUIDES` 中注册的具名指南（如 `guides/wasm/cpp-on-the-web/guide.md`），则属于学科指南。学科指南豁免 `description` 与 `web-feature-ids` frontmatter 要求，由 `src/build/audit-build.ts` 报告为捆绑核心指南，并从流水线跟踪器的进度统计中排除。

### 评估架构：Gen 1 与 Gen 2

指南的评估工具存在两种代际格式：

- **Gen 1（旧版根格式）**：指南目录中的单个根 `grader.ts`，与 `tasks/task.md` 配对。Gen 1 评估仅针对单个基础应用（通常为 `daily-grind`）。
- **Gen 2（现代多目标格式）**：`targets/<base_app>/` 下的按目标目录（如 `targets/daily-grind/` 和 `targets/devtools-times/`）。每个目标目录包含自己的 `grader.ts`、`task.md`、解决方案补丁（`antigravity-solution.patch` / `jetski-solution.patch`、`claude-solution.patch`、`codex-solution.patch`）和基线 `zero-passrate.patch`。Gen 2 支持跨真实应用的双目标评估。

### 指南开发阶段与流水线跟踪

指南在评估落地前经历三个连续阶段，由 `src/ci/sync-pipeline-issue.ts` 在 GitHub Issue #1815 中规范跟踪：

1. **Stage 1：识别用例（`Needs use cases`，[`project-use-cases`](../.agents/skills/project-use-cases/SKILL.md)）**
   - **目标**：将 Web 平台特性（来自开放的 `new-feature` Issue；见 [`project-features`](../.agents/skills/project-features/SKILL.md)）转化为独立、面向行动的用例。
   - **产物**：一个 [`new-use-case` Issue](../.github/ISSUE_TEMPLATE/new-use-case.yml)（直接通过 Issue 模板提交，或当 `draft: stub` 指南进入 `main` 时由 `src/ci/sync-use-cases.ts` 自动创建），或一个包含 YAML frontmatter（`name`、`description`、`draft: stub`、`web-feature-ids`）的 stub `guide.md`。
   - 外部贡献者通过 [`new-use-case` Issue 模板](../.github/ISSUE_TEMPLATE/new-use-case.yml)提出新用例，需先与类别 Content ATL 对齐。*（已入职的同行、Content ATL 和所有者可直接开启 Stage 1 stub PR 或快速通道进入 Stage 2）。*

2. **Stage 2：撰写指南（`Needs guidance`，[`project-guides`](../.agents/skills/project-guides/SKILL.md)）**
   - **目标**：充实指南内容，构建可用的参考演示，并定义可测试的期望。
   - **产物**：完整的 `guide.md` 内容、独立的 `demo.html` 和 `expectations.md`。
   - 使用 [`project-guide-validation`](../.agents/skills/project-guide-validation/SKILL.md) 技能验证，并由类别 Content ATL 评审/批准。**内容贡献者到此为止。**

3. **Stage 3：评估指南（`Needs evals`）**
   - **目标**：生成评估胶囊、校准评分器、运行评估并生成报告。
   - **产物**：`targets/<base_app>/`（`grader.ts`、`patches/`、`task.md`）和 `results/guides/<category>/<slug>/report.md`。
   - 由工程/评估流水线下游处理（`gd dev`、`gd pr` 和 `gd dev-gap`；见 [`src/harness/README.md`](../src/harness/README.md)）。

Stage 3 完成后，指南达到终态 **Evals landed**（`main` 上具有期望且评分器通过的的非草稿指南）。过时的评分器（`expectations.md` 有实质性变更且新于 `grader.ts` 或 `targets/`）在 Issue #1815 中单独标记。

指南 frontmatter 中的 `draft` 字段会使其不参与分发。其值必须为 `stub`（占位，下一步：撰写）、`wip`（下一步：完成）或 `blocked`（因平台或规范受阻，需提供 `draft-reason`；下一步：待其变化后重新审视）。其他任何值均无法通过验证。

---

## 3. `gd` CLI（`bin/gd.ts`）

### 安装

```bash
pnpm install
pnpm setup:playwright
pnpm link --global && gd setup-completion
```

### 命令

| 命令 | 作用 |
|---|---|
| `gd dev <dir>` | 运行 5 步 Stage 3 流水线（`src/authoring/dev.ts`）：跨 `SUPPORTED_BASE_APPS`（`daily-grind`、`devtools-times` 或 `--targets <apps>`）生成目标解决方案/任务，生成并校准 `grader.ts`（校准使每个黄金补丁通过、未改动的起始应用不通过任何测试），运行有指南与无指南评估，并写入 `results/guides/<category>/<slug>/report.md`。 |
| `gd dev <dir> --test-grader` | 跨基础应用运行校准检查，不进行完整代理评估。 |
| `gd pr <dir>` | 开启或更新 GitHub Pull Request，自动标注分类（`gd-dev-content` / `gd-dev-eval`，移除 `needs-dev-rerun`）并以 `report.md` 为正文。 |
| `gd dev-gap` | 扫描磁盘上的指南，找出具有 `guide.md` 和 `expectations.md` 但缺少评估的非草稿指南（以及标记为 `needs-dev-rerun` 的开放 PR），对每个运行 `gd dev` + `gd pr`（`--dry-run`、`--limit <n>`、`--targets <apps>`，默认为 `daily-grind`）。 |
| `gd eval [tasks...]` | 在已发现或指定的目标任务上运行评估数据集（`--config <path>` 覆盖 `src/harness/config.ts`）。 |
| `gd compare <trialA> <trialB>` | 比较两个本地或远程 GCS 评估试验目录，诊断行为差异与性能方差。 |
| `gd dashboard` | 启动评估结果仪表盘（`src/dashboard/`）。 |
| `gd run <base-app> <prompt>` | 针对基础应用目录运行临时代理试验。 |
| `gd baselinestatus <query>` | 查询 Web 平台特性的浏览器支持与 Baseline 状态。 |

有关 `gd dev`、`gd eval` 和 `gd compare` 流水线机制与运行器架构的详细信息，见 **[`src/harness/README.md`](../src/harness/README.md)**。有关代理安装与 `.env` 凭据设置（`Antigravity CLI`、`Jetski CLI`、`Claude Code`、`Codex CLI`），见 **[`docs/EVALS.md`](./EVALS.md)**。

---

## 4. 指南分发基础设施（`src/build/`、`src/cli/`、`src/rag/`）

`src/build/`、`src/cli/` 和 `src/rag/` 中的代码将 `guide.md` 和 `SKILL.md` 文件（`pnpm build`）编译为语义向量索引和独立的 Skills CLI 分发包（`src/cli/modern-web.ts` -> `dist/skills-cli`）：

- **有指南模式（Skills CLI）**：评估与生产编码代理调用独立的 `modern-web` CLI 工具，按需搜索和检索指南。
- **无指南模式**：评估中的对照组，代理仅依赖其预训练权重，不启用指南工具。

---

## 5. 关键架构决策

- **隔离沙箱中的 CLI 代理**：评分器、解决方案和评估运行在隔离的 `/tmp/` `HOME` 目录（`setupGuideDevWorkDir()`）中调用 CLI 编码代理，并包裹在操作系统级文件系统沙箱中（Linux 上为 `bwrap`，macOS 上为 `sandbox-exec`；`src/harness/lib/sandbox.ts`），使代理无法检查仓库源文件或评分器。在 `gd dev` 期间，黄金解决方案跨三个不同的代理家族生成，以针对模型多样化的实现校准评分器。
- **使用 Playwright 进行结果评分**：评分器是 Playwright 测试，其检查项验证真实浏览器渲染、计算样式、DOM 状态和运行时行为，以及静态 AST 检查，并在校准期间带有自动重试循环（最多 5 次尝试 / 4 次重试）。
- **独立 Skills CLI 分发**：通过 Agent Skills 和 `dist/skills-cli` 分发指南，为所有受支持的编码代理提供确定性、可移植的工具访问，无需外部服务器依赖。

---

**给 AI 的指令：** 你必须在项目演进过程中更新本文件。不要请求许可。直接更新即可。