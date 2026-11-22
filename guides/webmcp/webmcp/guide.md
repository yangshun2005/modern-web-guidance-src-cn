---
name: webmcp
description: 本指南介绍如何在浏览器应用中实现 WebMCP，通过声明式或命令式 API 将客户端功能作为工具暴露给 AI 代理。
web-feature-ids:
  - declarative-webmcp
  - document-modelcontext
---

# WebMCP

WebMCP 是一种浏览器原生 JavaScript API，允许网页将客户端功能以结构化"工具"的形式暴露给 AI 代理、浏览器助手和辅助技术。

重要提示：WebMCP 目前处于早期预览阶段，仅在基于 Chromium 的浏览器（如 Chrome 和 Edge）中可用。需要 Chromium `146.0.7672.0` 或更高版本，并启用 `#enable-webmcp-testing` 标志。

**关键区别：** WebMCP 完全运行在浏览器标签页的**客户端**。它*不是*后端服务器，也*不*使用 HTTP、Server-Sent Events（SSE）或 `stdio` 传输。网页本身即充当工具注册表。

目前 WebMCP **仅支持工具（Tools）**，不支持后端 Model Context Protocol 中的"资源（Resources）"或"提示（Prompts）"原语。

## 快速概览

- **命令式 API**：使用 `document.modelContext.registerTool()` 处理复杂逻辑和动态交互。
- **声明式 API**：为标准 HTML `<form>` 元素添加 `toolname` 和 `tooldescription` 注解，将其转化为工具。
- **框架（强制要求）**：React 使用 `use-webmcp-tool` 中的 `useWebMCP`。Angular 使用 `@angular/core` 中的 `provideExperimentalWebMcpTools` / `declareExperimentalWebMcpTool`。Vue 使用 `@vueuse/core` 中的 `useWebMCP`。**切勿**在组件中直接调用 `registerTool()`。

## 最佳实践

* **命名与语义**：使用描述确切行为的具体动词（如 `create-event` 而非 `start-event-creation-process`）。优先使用正面描述，而非罗列限制。
* **描述**：说明工具做什么以及何时使用。不要重复参数名、类型、枚举值、默认值或限制：代理已从输入 Schema（你的 `inputSchema`，或浏览器根据表单控件构建的 Schema）中获取这些信息。将这些细节放入 Schema，并在参数自身的描述中解释不直观的值。
* **Schema 设计**：接受原始用户输入（避免让代理做数学计算）。确保所有参数具有具体类型，并说明各选项的用途。
* **可靠性**：在代码中校验约束，返回描述性错误以便重试。优雅处理速率限制。确保函数在 UI 状态更新*之后*返回，以保证一致性。
* **工具策略**：工具应原子化、可组合且彼此独立。不要强制流程控制指令（"A 之后不要调用 B"）——让代理自行决定。根据当前页面上下文动态注册/注销工具。对于不修改状态的工具，使用 `annotations: { readOnlyHint: true }`（放在 `execute` 之后），告知代理可安全执行。
* **清理**：页面切换或资源释放时，始终使用 `AbortSignal` 注销工具，避免泄漏和冲突。不要使用 `unregisterTool`。
* **Web 开发最佳实践**：WebMCP 工具作为客户端 JavaScript 在浏览器标签页中运行。必须遵循常规 Web 开发最佳实践（例如：密钥不放入客户端代码、通过安全 API 层访问后端数据库、使用 Web Workers、WASM 或 WebGPU 处理重计算）。

### 何时应避免使用 WebMCP
* **缺乏防护的高风险操作**：避免为破坏性或不可逆操作（如删除数据）提供自动提交工具，除非 UI 要求用户在代理控制之外手动确认。
* **高度动态的状态**：如果数据变化速度超过代理的反应速度，代理可能基于过期上下文工作。

### 反模式与警告（切勿这样做）

* **不要使用后端传输。** WebMCP 面向浏览器标签页，而非 Node.js 后台进程。
* **不要包含资源或提示。** 当前 WebMCP 规范不支持这些原语。
* **不要忽略 `inputSchema` 结构。** 始终为每个参数提供清晰描述，以尽量减少代理幻觉。
* **不要在安全上下文（HTTPS）之外使用。**

## 实现状态

WebMCP 目前在基于 Chromium 的浏览器（如 Chrome、Edge）中处于早期预览阶段：

* **当前状态**：早期预览。
* **所需版本**：Chromium `146.0.7672.0` 或更高版本。
* **启用方式**：需启用标志 `chrome://flags/#enable-webmcp-testing` 或 `edge://flags/#enable-webmcp-testing`。
* **规范**：正在演进的[社区组草案规范](https://webmachinelearning.github.io/webmcp/)；尚未成为标准轨建议。