# 发布 Modern Web Guidance 技能

为发布 AI 技能（Claude Code、Gemini CLI / Antigravity、VS Code、Cursor、GitHub Copilot CLI、Grok、Kimi Code 和 OpenAI Codex）的更新，我们使用自动化流水线将源文件打包为轻量分发包（`dist/skills-cli/`），并发布到 `GoogleChrome/modern-web-guidance`。

## 自动化 CI/CD 发布

发布通过 GitHub Actions 自动完成：
* **工作流**：[`.github/workflows/publish.yml`](../.github/workflows/publish.yml)
* **计划**：每周一 14:50 UTC（太平洋时间 6:50 AM PST / 7:50 AM PDT）运行，发布到 npm 的 `latest` dist-tag 和 `GoogleChrome/modern-web-guidance` 的 `main` 分支。
* **手动触发（`workflow_dispatch`）**：可通过 GitHub Actions 的 **Run workflow** 手动触发，参数：
  * `version`：可选，显式指定 semver 版本（如 `1.0.0`）；留空则自动递增补丁版本。

## 通过 CLI 手动发布

在终端中手动发布：

```bash
# 1. 预览变更和发布说明（dry run）
pnpm run publish-skills --dry-run

# 2. 发布到 @latest（或传入 --version X.Y.Z）
pnpm run publish-skills
pnpm run publish-skills --version 1.0.0
```

**发布流水线的内部流程：**
1. 从 `gh-pages` 收集最新的夜间评估基准摘要（`public-evals/suites.gen.json`）。
2. 确定目标版本（`vX.Y.Z`，来自显式 `--version` 或自动递增的补丁版本），并更新所有分发和插件清单。
3. 将源指南和技能编译为分发格式（`dist/skills-cli/`）。
4. 运行分发校验和安装测试套件。
5. 将编译后的 `dist/skills-cli/` 目录推送到 `GoogleChrome/modern-web-guidance` 的 `main` 分支。
6. 为发布打标签 `vX.Y.Z`。
7. 使用 Gemini 生成面向用户的发布说明，并通过 `gh release create` 创建官方 GitHub Release。

## 生成与预览发布说明

可使用 `generate-release-notes` CLI 为任意标签或 diff 生成并预览发布说明：

```bash
# 使用 Gemini 预览与最新标签的对比说明
node --env-file=.env src/build/generate-release-notes.ts v0.0.185

# 测试确定性回退（无需 API key）
node src/build/generate-release-notes.ts v0.0.185
```

### 发布说明结构

发布说明会自动将变更归入清晰的顶级章节：
* `## 🆕 New Guides`：新增指南及其用例的高层摘要。
* `## 🔄 Updated Guides`：描述实质性指南演进和 API 模式的要点。
* `## 🗑️ Removed Guides`：已弃用或删除的指南。
* `## 🌐 Browser Support Updates`：按 Baseline 状态变化分组（`Widely available` → `Newly available` → `Limited availability`）。
* `## 🔌 Plugins`：代理市场清单和插件定义的更新。
* `## 📊 Benchmark Evaluations`：最新的代理通过率提升。

## 本地开发与全局链接

在本地构建、安装并将编译包测试为全局 CLI：

```bash
cd "$(git rev-parse --show-cdup)" && node src/build/build-dist.ts && cd dist/skills-cli && npm install --global .
```

这会将包注册到全局，并将 `modern-web-guidance` 二进制放入 `PATH`。

## 架构说明：“单一包”方案

对于 Claude Code，`modern-web-guidance` 仓库作为**单一打包插件**（`googlechrome-skills`），而非单个插件的市场目录。

* **简化安装**：用户只需运行一条安装命令即可访问整套精选 Web 开发技能。
* **生态对齐**：Gemini CLI 和 VS Code 原生将仓库视为单一扩展。将项目整合为单一插件可确保所有环境结构一致，减少为每个技能文件解析嵌套清单的技术开销。