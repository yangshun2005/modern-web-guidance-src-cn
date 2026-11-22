# 贡献者生态与治理

`GoogleChrome/modern-web-guidance-src` 是一个开源项目，为 AI 编码代理和人类开发者提供现代 Web 平台特性的权威指南。项目依赖 Web 社区和生态系统中各领域专家的贡献。

本文档说明贡献者角色、编写权限和决策流程。


## 角色与职责

### 贡献者

贡献者是提交改进、缺陷修复和文档更新的社区成员。任何人都可以成为贡献者。

贡献者：
* 通过 Pull Request 提交缺陷修复、文档更新、Baseline 兼容性改进以及对现有指南的改进。
* 通过[提交 Issue](https://github.com/GoogleChrome/modern-web-guidance-src/issues) 提议新的指南主题和用例。
* 审查 Pull Request（建议性）。
* 其 Pull Request 由指定的 [Content ATL](./docs/ATLS.md) 或 [Owner](#owners) 审查并合并。

> **关于编写新指南的说明**：由于指南会被 AI 编码助手直接摄取，**从零编写新指南仅限 [Peer](#peers)**，除非获得 [Content ATL](./docs/ATLS.md) 或 [Owner](#owners) 许可。


### Peer

Peer 是正式入职的主题专家，拥有仓库写入权限。

Peer：
* 编写全新指南（用例、`guide.md`、`demo.html` 和 `expectations.md`），可直接快速进入 Stage 2，无需单独的 Stage 1 用例签核。
* 审查整个代码库的 Pull Request（同行评审）。
* 提交的 Pull Request 需由指定的 [Content ATL](./docs/ATLS.md) 或 [Owner](#owners) 审查并合并。

成为 Peer 需满足：
* 展现现代 Web 开发领域的主题专业知识，并熟悉仓库编写标准。
* 以协作方式贡献高质量工作（PR、评审、讨论）。

候选人可由任何 Peer、Content ATL 或 Owner 提名，经 Owner 多数投票确认。Peer 身份通过积极参与维持；不活跃的 Peer 可能被 Owner 撤销，回归后可恢复。


### 内容领域技术负责人（Content ATL）

Content ATL 是对特定领域类别（如*性能*、*CSS*、*表单*、*UI 行为*、*无障碍*）或横向特性（如*动效*、*WebAuthn*）承担正式所有权的 Peer。领域分配定义于 **[`.github/atls.json`](./.github/atls.json)**，详见 **[`docs/ATLS.md`](./docs/ATLS.md)**。

Content ATL：
* 作为所分配类别的首要技术权威。
* 分类处理 Issue，验证提议的 Stage 1 用例，并参与其领域内的评估调查。
* 审查、批准并合并其领域内的 Stage 2 指南 Pull Request。
* 维护指南准确性、评估就绪度和 Baseline 回退对齐。
* 其自身的 Pull Request 在合并前需由另一位 Content ATL、熟悉该领域的 [Peer](#peers) 或 [Owner](#owners) 审查并批准。

Content ATL 由 Owner 根据领域专业知识和积极管理情况任命。


### Owner

Owner 与 Content ATL 和 Peer 协商治理项目。Owner 监督项目整体健康、技术方向、仓库基础设施、CLI 工具（`gd`）、分发服务和治理。Owner 小组包括浏览器厂商代表和独立 Web 平台专家。

Owner：
* 与 Peer 一同编写指南并在整个仓库中贡献。
* 管理基础设施、工具链、服务流水线、Schema 和仓库设置。
* 任命 Content ATL，确认新 Peer 和 Owner。
* 解决特性分类和项目政策上的跨领域分歧。
* 批准并合并治理、基础设施和 [`CODEOWNERS`](./CODEOWNERS) 保护路径的变更。
* 发布包并管理主版本发布。


#### Owner 小组

Owner 小组由代表浏览器和开发者生态利益相关方的席位以及独立 Web 平台专家组成，结构旨在确保多利益相关方治理和跨浏览器中立性：

* **Google Chrome**（2 席）：
  * Kadir Topal（[`@atopal`](https://github.com/atopal)）
  * Rick Viscomi（[`@rviscomi`](https://github.com/rviscomi)）
* **Microsoft Edge**（1 席）：
  * Patrick Brosset（[`@captainbrosset`](https://github.com/captainbrosset)）
* **Mozilla Firefox / MDN**（1 席）：
  * Jake Archibald（[`@jakearchibald`](https://github.com/jakearchibald)）
* **独立 Web 平台专家**（2 席）：
  * Dr. Lea Verou（[`@LeaVerou`](https://github.com/LeaVerou)）
  * *[独立代表，待定]* — 由 Owner 提名并任命

成为 Owner 需满足：
* 展现长期承诺（直接或通过其组织，例如担任或配备活跃的 [Content ATL](./docs/ATLS.md) 角色）、技术领导力，以及对项目架构和标准的深入了解。
* 积极推动项目方向、基础设施和社区健康。

新 Owner 由现有 Owner 提名，经多数投票确认。Owner 席位取决于持续积极参与，由 Owner 定期审查。


## 决策

决策遵循维护者之间的共识寻求模式。若无法达成共识，由 Owner 简单多数投票解决。决策以书面形式记录（如通过 Pull Request、Issue 或文档更新），并回应所提出的任何异议。任何 Peer、Content ATL 或 Owner 均可通过相同流程提议修改项目政策，包括本文档。关于特性稳定性争议，参见[易变性异议](./guides/CONTRIBUTING.md#volatility-objections)。


## 项目会议

Owner 每季度或按需开会，审查项目健康状况和路线图优先级。参与组织可向 Owner 会议派遣额外的无投票权观察员。