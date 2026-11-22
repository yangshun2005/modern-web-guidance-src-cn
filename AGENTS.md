# AGENTS.md

本仓库编写 **Modern Web Guidance**，涵盖 Web 平台特性（`guides/`）、独立技能（`skills-src/`）和评估工具（`src/`）。由于 `guide.md` 是外部编码代理在运行时**唯一**检索的文件（`demo.html`、`expectations.md` 和 `targets/` 仅在本仓库内用于验证和评估），所有指导内容必须自包含、确定性且面向命令。

## 必需的项目技能（唯一事实来源）

开始任务前，务必阅读并遵循 `.agents/skills/<skill>/SKILL.md`：

- `project-features`：提交与分类 `new-feature` Issue
- `project-use-cases`：识别用例（阶段 1 — `guide.md` frontmatter 存根）
- `project-guides`：编写指导内容（阶段 2 — `guide.md`、`demo.html`、`expectations.md`）
- `project-evals`：编写与审查评估（阶段 3 — `expectations.md`、`task.md`、`grader.ts`）
- `project-guide-validation`：验证与审查指南
- `project-discipline-guides`：纪律指南（`src/core/guide-validation.ts` 中的 `DISCIPLINE_GUIDES`）
- `project-coding-standards`：仓库 TypeScript 与 CLI 代码（`src/`、`bin/`）
- `nightly-eval-investigation`、`dev-prompt-canary`、`grader-replay`：评估诊断、评分器重放与 `gd dev` 提示词变更
- `project-maintenance`、`coherence-auditor`：日常维护与一致性审计

关于仓库架构、术语、治理和贡献者工作流，参见 [`docs/CONTEXT.md`](./docs/CONTEXT.md)、[`docs/LEXICON.md`](./docs/LEXICON.md)、[`CONTRIBUTING.md`](./CONTRIBUTING.md)、[`GOVERNANCE.md`](./GOVERNANCE.md)、[`guides/CONTRIBUTING.md`](./guides/CONTRIBUTING.md) 和 [`docs/ATLS.md`](./docs/ATLS.md)。

## GitHub 与 PR 礼仪

- **外部贡献者门槛（非 [Peers](./GOVERNANCE.md#peers)/ATLs/Owners，即 `gh api repos/GoogleChrome/modern-web-guidance-src --jq '.permissions.push'` 不为 `true`）：** 提交 PR 前：(1) 与用户确认其已使用 `git config user.email` 签署 [Google CLA](https://cla.developers.google.com/)，并通过 `gh` / `git` 验证 (2) 存在已达成一致的 Issue 且已分配给他们或标记为 `pr-welcome`（未关闭的 Issue 和维护者的后续评论**不**构成 PR 邀请；仅无歧义的拼写错误/失效链接可跳过此检查），(3) 没有其他开放的竞争 PR 针对该 Issue（且若用户在此仓库有 0 个已合并 PR，则本仓库中**其他开放/草稿 PR 为 0**），(4) 内容差异最多涉及 **1 个指南目录**（除非在关联 Issue 上获得批准）。任一检查失败，即停止且不要提交 PR。
- **来源前缀：** 为代理起草的 GitHub Issue、PR 描述和审查评论添加前缀：`🤖` 自主（无引导）、`✨` 定向（简短提示）或 `🧠` 协同设计（多轮迭代/调试）。
- **禁止 AI 提交尾注：** 绝不为 AI 模型或工具输出 `Co-Authored-By:` 尾注。
- **PR 描述：** 保持描述简洁，聚焦动机和净效果——不要开场废话（"This PR..."），不要重复标题，不要列出 diff 中已可见的变更文件/方法，不要验证样板。
- **审查回复：** 逐字提交 ` ```suggestion ` 块并附上人类 `Co-authored-by: Name <email>` 尾注，并回复每个已处理的讨论串（引用提交 SHA 或说明建议如何被采纳）。仅对逐字建议或琐碎修复解决讨论串；非逐字采纳的修改保持开放，供审查者验证。
```