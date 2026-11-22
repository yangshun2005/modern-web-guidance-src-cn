# 内容领域技术负责人（ATL）

内容领域技术负责人（Content ATL）是项目[同行](../GOVERNANCE.md#peers)和长期贡献者，负责 Modern Web Guidance 中领域类别（如*性能*、*CSS*、*表单*、*UI 行为*、*无障碍*）及横切功能横向组的归属与管理。

Content ATL 确保其领域内的所有指南技术准确、符合现代 Web 标准，并可供下游自动化评估使用。


## 核心职责

* **领域专业知识与覆盖策略**：作为其领域内架构或平台问题的主要技术联系人，主动识别需要新指南的高影响力开发者任务。
* **用例验证（阶段 1）**：在作者投入精力撰写完整指南之前，依据 [`project-use-cases`](../.agents/skills/project-use-cases/SKILL.md) 标准，对其类别内提出的新用例进行分诊和对齐。
* **撰写与技术审查（阶段 2）**：为新用例撰写指南，并依据 [`project-guides`](../.agents/skills/project-guides/SKILL.md) 和 [`project-guide-validation`](../.agents/skills/project-guide-validation/SKILL.md) 标准，对其领域内的 PR 提供正式技术审查和签核。每份指南必须由同行或 Content ATL 撰写（或共同撰写/赞助），并依据 [`GOVERNANCE.md`](../GOVERNANCE.md#content-area-tech-leads-content-atls) 获得批准后方可合并。
* **学科指南管理**：确保学科级指南按照 [`project-discipline-guides`](../.agents/skills/project-discipline-guides/SKILL.md) 拆分为聚焦的子指南，而非单体指南。
* **评估分诊与持续维护**：当自动化评估暴露内容或预期问题时，与工程团队协作；随 Web 标准和 Baseline 状态演进，保持 `guide.md` 与 `expectations.md` 一致；清理变得冗余的指南。


## 类别归属与查找

ATL 管理范围涵盖**领域纵向**（目录类别）和**指南横向**（如 Motion 或 WebAuthn 等横切功能组）。

官方归属映射维护在 **[`.github/atls.json`](../.github/atls.json)** 中。路由 Pull Request 时，`src/ci/atl-triage.ts` 通过严格的优先级瀑布为每份被修改的指南解析出**单一主审人**（仅当无合格审阅者匹配或审阅者为 PR 作者时，才降至下一层级）：

1. **主要指南作者（`git blame`）**：对于现有指南的更新，由撰写 `guide.md` 主要份额且具有写权限的活跃[同行](../GOVERNANCE.md#peers)/ATL/Owner 负责（同行进行首轮审查并标记类别 ATL 以完成合并签核，依据 [`GOVERNANCE.md`](../GOVERNANCE.md)；ATL/Owner 可直接合并，或在需要时标记其他审阅者）。
2. **`web_features`**：`.github/atls.json` 中的具体功能级分配（如 `prefers-reduced-motion`、`canvas-html`）。
3. **`web_features_groups`**：横切指南横向组和功能组（如 `animation`、`transitions`、`view-transitions`、`scrolling`、`webauthn`）；已作为 `default` 类别存在的宽泛纵向组（如 `css`、`security`）不会在指南 PR 上覆盖其他目录的类别 ATL。
4. **`default`**：各指南目录的领域纵向默认值（`css`、`forms`、`performance`、`ui-behaviors`、`ui-components`、`built-in-ai`、`privacy`、`security`、`webmcp` 等），当某层级列出多位负责人时，选择未完成审查请求最少的候选人。

`.github/workflows/atl-triage.yml` 工作流要求外部贡献者 PR 在请求审查前，须有已批准（`pr-welcome` 或作者指派）的 Issue 并通过 `cla/google` 检查。


## 审查与分诊工作流

* **审查阶段 1 用例**：在批准作者进入阶段 2 之前，验证提议的用例遵循 [`project-use-cases`](../.agents/skills/project-use-cases/SKILL.md)（面向任务的表述、范围不重复、真实开发者影响）。
* **审查阶段 2 指南 PR**：验证 `guide.md`、`demo.html` 和 `expectations.md` 满足 [`project-guides`](../.agents/skills/project-guides/SKILL.md) 和 [`project-guide-validation`](../.agents/skills/project-guide-validation/SKILL.md)（技术准确、指南自包含、Baseline 回退宏使用恰当、独立演示干净、预期可观测）。
* **分诊易变性异议**：在其领域内的 Issue 或 Pull Request 被标记 `volatility` 标签后两周内，评估该关切（例如指南能否限定到该功能的稳定子集），并给出建议结果供 Owner 依据[易变性异议](../guides/CONTRIBUTING.md#volatility-objections)决定。