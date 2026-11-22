# 评估与基准测试词典

Modern Web Guidance 评估数据集、基准执行、评分与报告的规范领域语言（`src/harness/`、`src/grading/`、`src/authoring/`、`src/dashboard/`）。区别于 [`src/legacy-scan/LEXICON.md`](../src/legacy-scan/LEXICON.md) 中的 `legacy-scan` 领域词汇。

本词典优先约束文档、CLI 帮助和仪表盘 UI 术语；`_Reference_` 中引用的遗留内部符号、标志和构建产物路径在后续代码迁移前保持不变。

## 基准结构

### Dataset（数据集）

仓库中跨指南的评估 **Task** 完整集合（编写于指南 `targets/` 和遗留 `tasks/` 目录）。区别于 **Job**（在某一时间点执行配置选定的任务）和 `legacy-scan` 的 **Grading Deck**（`src/legacy-scan/LEXICON.md`）。

* _Reference_：`src/core/guide-validation.ts#getTaskMap`
* _AKA_：Evaluation Suite — `Agentic Benchmarks + Evals` / Anthropic 评估术语
* _Avoid_：Test Suite、Suite（指静态集合时）

### Task（任务）

单个基准问题，配对开发者提示（`task.md`）、**Base App** 和 Playwright **Grader**（`grader.ts`，由 `expectations.md` 派生）。一个指南可在不同基础应用或 Baseline 目标（`widely` / `newly` / `limited`）下拥有多个任务。

* _Reference_：`src/core/guide-validation.ts#TaskInfo`、`guides/<category>/<slug>/targets/<base_app>/task.md`
* _Avoid_：Scenario、Test Case

### Base App（基础应用）

起始 Web 应用（`daily-grind`、`devtools-times`、`cards-app`、`empty-app`），存放于 `src/harness/base-apps/`，作为 **Task** 或临时 `gd run` 的干净起始工作区被复制。针对该起始应用的按指南任务输入（`task.md`、`grader.ts`、补丁）因历史原因位于 `guides/<category>/<slug>/targets/<base_app>/`；区别于 Web Platform Baseline 特性目标（`widely` / `newly` / `limited`）。

* _Reference_：`src/core/guide-validation.ts#getSupportedBaseApps`、`src/harness/base-apps/`
* _Avoid_：Template（CLI/UI 文案中）、Target App

## 执行与条件

### Job（作业）

**Eval Harness** 针对配置选定的 **Task**、**Condition** 和重复次数，在某个 **Agent Harness** 和模型快照上的一次批量执行（例如夜间调度、基线发布快照或临时调用）。评估仪表盘落地页的每一行代表一个作业。

* _Reference_：`src/harness/run-suite.ts`、`src/dashboard/summary-extractor.js#JobSummary`、`results/suites/`
* _AKA_：Job — Harbor Framework
* _Avoid_：Suite、Suite Run、Run（未限定指批量时）、Trial（指指南趋势图中的历史作业点时）

### Trial（试验）

**Agent Harness** 在某一 **Condition**（**Guided** 或 **Unguided**）下对单个 **Task** 的一次执行，产出一条 **Trajectory** 和一组 Grader **Check** 结果。当 **Job** 重复任务（`×N/task`）时，`N` 表示**每个条件的试验次数**（每次重复包含一次 guided 试验和一次 unguided 试验）。区别于 Chrome 的 `Origin Trial`。

* _Reference_：`src/harness/lib/metrics.ts#RunResult`、`results/suites/<suiteId>/<n>/<guide>/<task>/<guided|unguided>/`
* _AKA_：Trial — Harbor Framework 和 `Agentic Benchmarks + Evals`；Attempt — Harbor Framework（`n_attempts`）
* _Avoid_：Run / Task Run（指叶子执行时）、Pass（指重复次数时）、Trial Pair

### Condition（条件）

**Job** 中 **Trial** 运行所依据的 A/B 实验处理：**Guided** 让 **Agent Harness** 访问 `modern-web-guidance` 技能和 CLI 检索，**Unguided** 则在未启用指导工具或文档的情况下运行对照组。

* _Reference_：`src/harness/run-suite.ts`、`src/harness/lib/metrics.ts#calculateMetrics`（`runType: 'guided' | 'unguided'`）

### Agent Harness（Agent 执行框架）

与被评估模型配对的编码 CLI 循环（简称 **Agent** — 如 Claude Code、Codex CLI、Jetski CLI 或 Antigravity CLI），通过仓库内 agent runner 在隔离沙箱中调用。区别于围绕其编排基准测试的 **Eval Harness**。

* _Reference_：`src/harness/config.ts#Agents`、`src/harness/agents/`
* _AKA_：Agent Harness — `Agentic Benchmarks + Evals`

### Eval Harness（评估执行框架）

仓库的基准测试基础设施（`src/harness/`），负责准备 **Base App** 工作区、调用 **Agent Harness** 试验、运行 Playwright **Grader** 并聚合 **Job** 结果。区别于被测的 **Agent Harness**。

* _Reference_：`src/harness/run-suite.ts`、`docs/EVALS.md`
* _AKA_：Evaluation Harness — `Agentic Benchmarks + Evals`

### Trajectory（轨迹）

**Trial** 的规范化结构化执行历史，从原始 agent 会话日志解析为轮次、推理、工具调用、技能搜索、指南检索和文件编辑。区别于未解析的原始 CLI 会话文件和 `modern-web.log` 附属文件。

* _Reference_：`src/harness/lib/trajectory-normalizer.ts#TrajectorySummary`
* _AKA_：Trajectory — Harbor Framework（`ATIF`）；Trace / Transcript — `Agentic Benchmarks + Evals`

## 评分与结果

### Grader（评分器）

Playwright 测试文件（`grader.ts`，由 `expectations.md` 编写），通过检查构建产物、DOM 状态、计算样式和浏览器运行时行为来验证 **Task** 工作区。区别于其中的单个 **Check** 和 `legacy-scan` 的 **Detection Eval Judge**（`src/legacy-scan/LEXICON.md`）。

* _Reference_：`src/grading/run-grader.ts`、`guides/<category>/<slug>/targets/<base_app>/grader.ts`
* _Avoid_：Spec（Playwright 内部除外）、LLM Judge（保留给 `src/legacy-scan/`）

### Check（检查项）

**Grader** 在 **Trial** 期间执行的单条确定性断言；通过率等于有效配对任务中通过的检查项数除以运行的检查项数。区别于整个 `grader.ts` **Grader** 文件。

* _Reference_：`src/harness/lib/metrics.ts#ScenarioCheck`、`src/harness/lib/metrics.ts#calculateMetrics`
* _Avoid_：Assertion（UI/指标文案中指检查项时）

### Uplift（提升）

**Guided** 与 **Unguided** 检查项通过率之间的差值，以百分点（`pp`）计量（例如 `25% -> 90%` 为 `+65 pp`），而非相对百分比变化。在紧凑 UI 徽章中也简称为 **Lift**。

* _Reference_：`src/harness/lib/metrics.ts#calculateMetrics`、`src/dashboard/index.html`
* _AKA_：Lift — 仪表盘紧凑徽章

### Infra Error（基础设施错误）

因 harness、沙箱、环境或上游 API 基础设施而非生成解决方案质量导致 **Trial** 失败的统称分类，不计入通过率分母，并在图表中渲染为不确定区间。涵盖：

- **Generation Error**：agent 崩溃、完成前非零退出、初始化失败、上游限流耗尽或 **Agent Timeout**。
- **Grading Infra Error**：grader 文件缺失、Playwright runner 崩溃、结果报告缺失，或评分期间非解决方案导致的服务器/浏览器启动失败。

不要与解决方案导致的 **`Page-Load / Build Error`**（`pageLoadFailed`）、解决方案应用文件缺失（`Target app missing`）、**`Unguided Leak`** 或编写期 **`Calibration Failure`** 混淆。

* _Reference_：`src/harness/lib/metrics.ts#isEarlyFailureRun`、`src/harness/lib/collection.ts#evaluateScenarioResults`、`src/authoring/dev-report.ts#DevReportFlag`（`INFRASTRUCTURE_ERROR`）
* _Avoid_：Early Failure、Run failed before grading、System error（指运行时基础设施故障时）、Generation Error（用作顶层统称时）

### Page-Load / Build Error（页面加载 / 构建错误）

当 agent 修改后的应用在 `checkTargetPageLoad` 期间构建失败或初始浏览器加载时出现运行时错误，所记录的评分结果。与 **`Infra Error`** 分开跟踪，因为 agent 输出中的构建/运行时缺陷反映解决方案质量而非 harness 基础设施（由 runner/浏览器启动崩溃导致的情况除外）。

* _Reference_：`src/grading/test-fixture.ts#checkTargetPageLoad`、`src/grading/run-grader.ts#runPlaywright`、`src/dashboard/insights.js#toJobRef`（`pageLoadFailed`）

### Unguided Leak（Unguided 泄漏）

当 **Unguided** 试验访问或检索到其预期指南（或 `modern-web-guidance` 技能）时触发的对照卫生标志，表明 unguided 基线被污染，而非 **`Infra Error`** 或检查项失败。

* _Reference_：`src/harness/lib/collection.ts#collectGuideUsage`、`src/dashboard/utils.js#isUnguidedLeakTrial`

### Calibration Failure（校准失败）

编写期 `gd dev` 失败，即 **Task** 的 **Grader** 针对黄金解决方案或未改动基线（`zero-passrate.patch`）验证失败。区别于运行时 **`Infra Error`** 和基准 **Job** 中的运行时检查项失败。

* _Reference_：`src/authoring/dev-report.ts#DevReportFlag`（`CALIBRATION_FAILED`）、`src/grading/run-grader.ts`