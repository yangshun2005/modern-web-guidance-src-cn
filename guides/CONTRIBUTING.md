# 贡献指南

本指南介绍 **同行评审者与内容贡献者** 在 `guides/` 中编写或更新 Web 平台指南的工作流程。

关于编写权限和贡献者角色，请参阅 **[`GOVERNANCE.md`](../GOVERNANCE.md)** 和 **[`docs/ATLS.md`](../docs/ATLS.md)**。内容作者只专注于技术准确性、参考演示和可测试的预期——Stage 3 评估套件（`grader.ts`）、解决方案补丁和校准运行由下游通过 [`src/harness/README.md`](../src/harness/README.md) 处理。


## 指南范围

* **厂商无关的核心指南**：核心指南聚焦于 [`web-features`](https://github.com/web-platform-dx/web-features) 和 [Browser Compat Data (BCD)](https://github.com/mdn/browser-compat-data) 中跟踪（或待纳入）的 Web 平台特性。浏览器特定指南（例如扩展商店或专有 API）绝不属于核心指南，只能以独立的可选技能包形式发布，并遵循相同的审查规则。
* **浏览器特定实现细节**：指南不得引用浏览器特定的实现细节（例如某浏览器内置的端侧模型），除非明确说明这些细节是浏览器特定的且可能变化。
* **基线目标与回退方案**：指南以 **Baseline Widely available** 的 Web 特性为目标，对于 Limited availability 或 Newly available 的特性，必须提供回退策略或渐进增强模式。当不存在合理的替代方案时，若该特性的浏览器支持范围窄于用户的支持目标，文档化的回退方案可以是放弃使用该特性。


## 特性稳定性

某特性能否出现在默认指南中，取决于其发布进度：

| 特性成熟度 | 指南中的处理方式 |
|---|---|
| 尚未进入 Origin Trial 或任何稳定版本 | 不在范围内。 |
| Origin Trial | 在 [`features/origin-trials.json`](../features/origin-trials.json) 中跟踪，并通过 `.mwgrc` 中的 Origin Trial 选项控制启用。 |
| 仅在一种浏览器引擎的稳定版中发布 | 发布后 3 个月内视为 `volatile`，并通过 `.mwgrc` 中的 `volatile` 选项控制启用。 |
| 已在两种或更多引擎中发布，或已从 `volatile` 毕业 | 纳入默认指南。 |

* **毕业**：`volatile` 特性在 3 个月窗口结束时自动转入默认指南，除非异议成立。超出窗口仍保持 `volatile` 的特性，在相关问题解决后毕业（例如第二个引擎发布该特性且设计稳定，或 Content ATL 将指南范围限定为稳定子集）。
* **异议与请愿**：任何同行评审者、Content ATL 或 Owner 均可对特性毕业提出异议、请愿缩短或延长其窗口，或对变化超出预期的特性重新打标签。参见 [易变性异议](#易变性异议)。

### 易变性异议

要提出异议或请愿，请给相关 Issue 或 Pull Request 打上 `volatility` 标签（或新建一个带该标签的 Issue）并说明理由。根据 [`docs/ATLS.md`](../docs/ATLS.md#review--triage-workflows)，指定的 Content ATL（若该领域无 ATL，则为 Owner）在两周内进行分诊并建议结果，由 Owners 通过[治理决策流程](../GOVERNANCE.md#decision-making)决定。


## 内容编写工作流程（Stage 1 与 Stage 2）

指南从概念到评估发布需经过三个连续阶段（参见 [`docs/CONTEXT.md` 中的指南开发阶段](../docs/CONTEXT.md#guide-development-stages--pipeline-tracking)）。**内容作者专注于前两个阶段**，每个阶段由一份规范性 Agent Skill 管理：

### Stage 1：识别用例（`Needs use cases`）

将 Web 平台特性转化为 2–5 个独立、面向行动的开发者任务（聚焦于用户想要完成*什么*，而非 API *如何*工作）。

* **通过 Issue**：打开一个 [`new-use-case` Issue](../.github/ISSUE_TEMPLATE/new-use-case.yml) 提出用例，并在打开 PR 前与 [`.github/atls.json`](../.github/atls.json) 中列出的该类别 [Content Area Tech Lead (ATL)](../docs/ATLS.md) 达成一致。
* **对于已加入的[同行评审者](../GOVERNANCE.md#peers)、Content ATL 和 Owners**：可直接快速进入 Stage 2，或打开一个 Stage 1 存根 PR（`guides/<category>/<use-case-slug>/guide.md`，仅包含带 `draft: stub` 的 YAML frontmatter 存根），以便与该类别 ATL 达成一致。

* **权威规范**：**[`.agents/skills/project-use-cases/SKILL.md`](../.agents/skills/project-use-cases/SKILL.md)**（类别中心指南则参见 **[`project-discipline-guides`](../.agents/skills/project-discipline-guides/SKILL.md)**）。

### Stage 2：编写指南（`Needs guidance`）

用例对齐后，在 `guides/<category>/<use-case-slug>/` 中编写三个核心文件：

| 文件 | 用途 |
|---|---|
| `guide.md` | 自包含指南，供 AI 编码 Agent 通过 RAG 检索（无外部链接，带注释的代码片段解释*为什么*，Baseline 回退宏）。 |
| `demo.html` | 独立、无控制台警告的用例参考实现。 |
| `expectations.md` | 可观察、可测试的需求要点列表，供下游生成自动化评分器。 |

* **权威规范**：**[`.agents/skills/project-guides/SKILL.md`](../.agents/skills/project-guides/SKILL.md)**


## 自验证（打开 PR 前）

起草 `guide.md`、`demo.html` 和 `expectations.md` 后，请让编码 Agent 运行 **[`project-guide-validation`](../.agents/skills/project-guide-validation/SKILL.md)** 技能：

```text
Please run the project-guide-validation skill on my guide: guides/<category>/<use-case-slug>
```

该技能通过 DevTools MCP 在真实浏览器会话中测试 `demo.html`，对照 [`guides/accessibility/accessibility/guide.md`](./accessibility/accessibility/guide.md) 检查无障碍一致性，并验证 `guide.md` 要求的每个结果在 `expectations.md` 中都有对应预期，且 `demo.html` 逐一覆盖。


## 指南 PR 检查清单

- [ ] 已签署 [Google CLA](https://developers.google.com/open-source/cla/individual)。
- [ ] 已遵循 [`project-use-cases`](../.agents/skills/project-use-cases/SKILL.md) 和 [`project-guides`](../.agents/skills/project-guides/SKILL.md)。
- [ ] 已通过 [`project-guide-validation`](../.agents/skills/project-guide-validation/SKILL.md) 运行自验证，并确认 `pnpm test` 通过。
- [ ] 已标记 [`.github/atls.json`](../.github/atls.json) 中指定的类别 ATL 进行审查和批准。