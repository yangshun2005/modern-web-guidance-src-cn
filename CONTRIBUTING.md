# 为 Modern Web Guidance 做贡献

我们欢迎你的贡献！我们相信，Web 开发者——以及协助他们的 AI 编码代理——能够获取现代 Web 平台最高质量、最准确、最新的指导至关重要。

本指南提供项目级治理政策，并引导你前往贡献方向对应的专门文档。


## 贡献者许可协议

向 Google 开源项目提交的所有内容，在 Pull Request 合并前都需要签署贡献者许可协议（CLA），即使是仅涉及文档的更改也不例外。

* **个人贡献者**：如果你编写原创源代码并拥有知识产权，请签署[个人 CLA](https://developers.google.com/open-source/cla/individual)。
* **企业贡献者**：如果你所在公司允许你贡献工作成果，请签署[企业 CLA](https://developers.google.com/open-source/cla/corporate)。


## 治理与编写权限

贡献者角色（**Contributors**、**Peers**、**Content ATLs** 和 **Owners**）、编写权限及审查政策定义于 **[`GOVERNANCE.md`](./GOVERNANCE.md)**。由于指导内容会被 AI 编码助手直接摄取，从零编写全新指导内容仅限已入职的 [Peers](./GOVERNANCE.md#peers)，除非获得 [Content ATL](./docs/ATLS.md) 或 [Owner](./GOVERNANCE.md#owners) 许可；任何人都可以提交 Issue 提出用例，或提交 PR 改进现有指南。


## 请提交问题，而非补丁

生成补丁成本很低；而验证变更是否符合规范、是否经得起实证检验、是否与我们的评估套件对齐，需要维护者投入时间。

* **提交 PR 前先签署 Google CLA：** 访问 **[cla.developers.google.com](https://cla.developers.google.com/)**，在提交 Pull Request *之前*，使用你 `git config user.email` 中配置的准确邮箱地址签署 CLA。请移除任何 AI 生成的 `Co-Authored-By:` 提交尾注，因为非人类邮箱无法通过 CLA 检查。
* **问题优先（必须先有 Issue）：** 除了明确的拼写错误或失效链接修复外，**所有 Pull Request 都必须先有已达成一致的 Issue**（已分配给你或标记为 `pr-welcome`）——包括对 `guides/` 内容、`demo.html`、`expectations.md`、`src/` 工具以及 `targets/` 评估的更改。没有已达成一致问题背景的 Pull Request 将被关闭。
* **开放的 Issue 不代表可以提交 PR：** 跟踪器中的 Issue 往往是维护者的任务项、未细化的提案或批量工作的一部分。除非维护者**已将 Issue 分配给你**或标记为 `pr-welcome`，否则不要为开放的 Issue（或审查评论中提到的后续事项）提交 PR。未经请求的（"抢 Issue"）PR 和重复竞争的 PR 将被直接关闭，不予审查。
* **外部贡献者的范围与并发限制：**
  * **同时只能有 1 个开放的 PR：** 除非你是已入职的 [Peer、Content ATL 或 Owner](./GOVERNANCE.md)，否则同一时间只能有**一个**开放的 Pull Request（包括草稿）。请等待当前 PR 合并或关闭后再提交下一个。
  * **每个 PR 只涉及 1 篇指南：** 外部内容 PR 必须限定在单个指南目录（`guides/<category>/<slug>/`）内，除非关联 Issue 上已批准多指南更改（例如将共享内容提取到 `features/`）。未经请求的多指南或跨类别批量更改将被关闭。
* **AI 辅助贡献：** 你有责任理解并验证你提交的每个 AI 辅助 Issue、PR 或评论的技术与 Web 规范准确性，并遵守 [`AGENTS.md`](./AGENTS.md#github--pr-etiquette) 中的礼仪规范。


## 选择你的贡献方向

你想在哪里做贡献？点击下方链接进入对应路径：

| 贡献方向 | 描述 | 文档 |
|---|---|---|
| **✍️ 指导内容** | 编写或更新 Web 平台指导内容（阶段 1 和 2：用例、`guide.md`、`demo.html`、`expectations.md`、自验证）。与评估基础设施隔离。 | **[`guides/CONTRIBUTING.md`](./guides/CONTRIBUTING.md)** |
| **🛡️ 类别管理** | 内容领域技术负责人（ATL）负责分诊用例、审查指导 PR，并维护领域类别健康。 | **[`docs/ATLS.md`](./docs/ATLS.md)** |
| **⚙️ 工具、基础设施与评估** | 开发统一的 `gd` CLI、提示基准测试框架、Playwright 评分器生成器、服务编译器以及仪表盘。 | **[`src/harness/README.md`](./src/harness/README.md)** 和 **[`docs/EVALS.md`](./docs/EVALS.md)** |
| **🏛️ 项目治理** | 贡献者角色（Contributors、Peers、Content ATLs、Owners）、权利、决策模型及会议节奏。 | **[`GOVERNANCE.md`](./GOVERNANCE.md)** |


## 仓库架构

为了营造开源贡献者环境，同时为最终用户保持干净、稳定的安装路径，我们采用双仓库架构：

* **源码仓库（[GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src)）**：包含源指导文件、开发脚本、评估框架、基础应用、测试和 CLI 工具。**所有 Issue 和 Pull Request 均在此提交。**
* **安装仓库（[GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance)）**：只读分发仓库，包含供编码代理使用的已编译 Skills 和插件配置。
* **同步与发布流程**：合并到 `modern-web-guidance-src` 的更改会按每周发布节奏编译并发布到分发仓库和 [`modern-web-guidance` npm 包](https://www.npmjs.com/package/modern-web-guidance)（参见 [`docs/RELEASING.md`](./docs/RELEASING.md)）。

关于仓库目录布局和架构的技术说明，参见 **[`docs/CONTEXT.md`](./docs/CONTEXT.md)**。


## 开发环境搭建与质量门禁

本项目使用 **pnpm**（要求 Node.js `>=22.18.0`）：

```bash
# Clone and install dependencies:
git clone https://github.com/GoogleChrome/modern-web-guidance-src.git
cd modern-web-guidance-src
pnpm install

# Link the unified CLI globally:
pnpm link --global && gd setup-completion

# Fast static check:
pnpm typecheck && pnpm lint

# Full preflight gate (builds distributions, typechecks, lints, and runs all unit tests):
pnpm preflight
```

使用编码代理编写更改或处理 PR 审查反馈时，请遵循 [`AGENTS.md`](./AGENTS.md#github--pr-etiquette) 中的工作流和 GitHub 礼仪。


## 项目 Agent Skills

本仓库在 [`.agents/skills/`](./.agents/skills/) 中包含一套精选的 **Agent Skills**，作为 AI 编码代理和人类贡献者的唯一规范性说明：

| Skill | 参考文档 | 描述 |
|---|---|---|
| **功能接收** | [`project-features`](./.agents/skills/project-features/SKILL.md) | 提交、调研和分诊 `new-feature` Issue 及待定 Web 功能。 |
| **用例** | [`project-use-cases`](./.agents/skills/project-use-cases/SKILL.md) | 制定面向行动的开发者任务和 frontmatter Schema（阶段 1）。 |
| **指南编写** | [`project-guides`](./.agents/skills/project-guides/SKILL.md) | 指令、代码片段约定、自包含约束和 Baseline 回退宏（阶段 2）。 |
| **指南验证** | [`project-guide-validation`](./.agents/skills/project-guide-validation/SKILL.md) | 自主 DevTools MCP 浏览器测试、可访问性和预期对齐（阶段 2）。 |
| **学科指南** | [`project-discipline-guides`](./.agents/skills/project-discipline-guides/SKILL.md) | 构建和精简类别根及概念中心学科指南。 |
| **评估与评分器** | [`project-evals`](./.agents/skills/project-evals/SKILL.md) | Playwright 测试评分器生成和校准标准（阶段 3）。 |
| **Baseline 状态** | [`web-baseline`](./.agents/skills/web-baseline/SKILL.md) | 检查 Web 平台功能的浏览器兼容性和 Baseline 状态。 |
| **编码规范** | [`project-coding-standards`](./.agents/skills/project-coding-standards/SKILL.md) | CLI 和工具代码的架构约定、严格类型、规范枚举和 PR 审查标准。 |
| **评估诊断** | [`nightly-eval-investigation`](./.agents/skills/nightly-eval-investigation/SKILL.md)、[`dev-prompt-canary`](./.agents/skills/dev-prompt-canary/SKILL.md)、[`grader-replay`](./.agents/skills/grader-replay/SKILL.md) | 调查夜间评估运行、验证 `gd dev` 提示更改以及重放评分器。 |
| **仓库维护** | [`project-maintenance`](./.agents/skills/project-maintenance/SKILL.md)、[`coherence-auditor`](./.agents/skills/coherence-auditor/SKILL.md) | 常规依赖更新、链接/文档一致性检查和工作区卫生。 |


## 提交 Pull Request

准备好提交 Pull Request 时：

1. **检查 PR 清单**：
   - [ ] 已签署 [Google CLA](https://developers.google.com/open-source/cla/individual)。
   - [ ] 所有预检通过（`pnpm preflight`）。
   - [ ] 对于指导类 PR：已遵循 [`guides/CONTRIBUTING.md`](./guides/CONTRIBUTING.md) 中的清单。
2. **推送分支并向 [GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src) 的 `main` 分支提交 PR**。
3. **审查流程**：维护者或指定的 Content ATL 将审查你的 PR、运行 CI 检查并提供反馈。一旦批准且所有检查通过，你的 PR 将被合并。


## 社区与行为准则

本项目遵循 [Google 开源社区准则](https://opensource.google/conduct/)。请在所有项目互动中遵守这些准则。

如有问题或想法，欢迎[提交 Issue](https://github.com/GoogleChrome/modern-web-guidance-src/issues) 或发起讨论。感谢你帮助构建更好、更现代的 Web！