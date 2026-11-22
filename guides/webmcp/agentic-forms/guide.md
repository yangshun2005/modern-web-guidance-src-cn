---
name: agentic-forms
description: 通过为标准 HTML 表单添加 WebMCP 属性，将客户端功能作为工具暴露给 AI 代理。
web-feature-ids:
  - declarative-webmcp
---

# Agentic Forms

声明式 API 通过属性将标准 HTML `<form>` 元素转换为 WebMCP 工具。浏览器根据表单输入合成 JSON Schema，并处理代理交互。

**Angular Signal Forms：** **不要**为使用 `@angular/forms/signals` 中 `form()` 构建的表单添加 `toolname`/`tooldescription`。应改为在应用 providers 中添加 `provideExperimentalWebMcpForms()`，并向 `form()` 传入 `experimentalWebMcpTool: { name, description }` 选项。Angular 随后通过命令式 API 注册表单，从 signal 模型构建 Schema，并将其接入验证和提交。同时使用两种方式会导致同一表单被注册两次。

## 表单属性

*   `toolname`：工具的唯一名称。
*   `tooldescription`：工具的功能以及代理应在何时使用它。不要列出表单的字段、选项或限制：浏览器已将它们包含在根据表单控件构建的 Schema 中。
*   `toolautosubmit`：（可选）如果存在，代理无需等待用户交互即可提交表单。
*   `toolparamdescription`：（可选）用于在 JSON Schema 中定义属性描述。用它解释值的含义（例如，配合示例值），不要重复控件的选项或限制。
    *   **解析顺序**：如果存在 `toolparamdescription`，浏览器优先使用它。否则，使用关联 `<label>` 的 `textContent`（跳过可标记的后代元素）。如果没有标签，则回退到 `aria-description`。
    *   **分组（Fieldsets）**：要为相关元素组（如 `<input type="radio">` 按钮）附加描述，请将 `toolparamdescription` 放在最近的父级 `<fieldset>` 元素上，使其作用于整个参数组。

### 示例

```html
<form toolname="search-cars" 
      tooldescription="Perform a car make/model search" 
      toolautosubmit>
  <label for="make">Vehicle Make</label>
  <input type="text" id="make" name="make" required>
  
  <label for="model">Vehicle Model</label>
  <input type="text" id="model" name="model" toolparamdescription="e.g., 330i, F-150" required>
  
  <button type="submit">Search</button>
</form>
```

## 在 JavaScript 中处理提交

当代理提交表单时，`SubmitEvent` 包含 `agentInvoked`（布尔值）和 `respondWith(promise)`。

```javascript
document.querySelector('form').addEventListener('submit', (event) => {
  event.preventDefault();

  // Validate the form
  const formValidationErrors = myFormIsValid();

  if (formValidationErrors.length > 0) {
    if (event.agentInvoked) {
      const errorString =
        'Validation failed: ' +
        formValidationErrors
          .map((err) => `${err.field} (${err.message})`)
          .join(', ');

      event.respondWith(Promise.resolve(errorString));
    }
    return;
  }

  const resultPromise = performAsyncSearch(new FormData(event.target));

  // Return the result directly to the agent without navigation
  if (event.agentInvoked) {
    event.respondWith(resultPromise);
  }
});
```

## 生命周期事件

当代理开始或停止与工具交互时，`document.modelContext` 会发出事件：`toolactivated` 在工具执行开始时触发，`toolcancel` 在工具被取消时触发。

```javascript
// Starting in Chrome 156, these events are dispatched on `document.modelContext`.
// Older versions dispatch them on `window`. Feature-detect the event handler
// property to pick the right target for backward compatibility.
const targetFor = (type, listener, options) =>
  (`on${type}` in (document.modelContext ?? {}) ? document.modelContext : window)
    .addEventListener(type, listener, options);

targetFor('toolactivated', ({ toolName }) => {
  console.log(`Tool "${toolName}" started execution.`);
});

targetFor('toolcancel', ({ toolName }) => {
  console.log(`Tool "${toolName}" execution is cancelled.`);
});
```

## 视觉反馈（CSS）

使用伪类在代理与表单交互时高亮表单：

*   `:tool-form-active`：应用于代理正在使用的 `<form>` 元素。
*   `:tool-submit-active`：当浏览器暂停等待用户审查时（如果省略了 `toolautosubmit`），应用于提交按钮。

```css
form:tool-form-active {
  outline: 2px dashed blue;
  background-color: rgba(0, 0, 255, 0.05);
}

button:tool-submit-active {
  outline: 2px dashed red;
  animation: pulse 2s infinite;
}
```

## 表单适用性（何时避免使用）

声明式 API 最适合自包含的标准表单。以下场景不宜使用：

* **高度依赖的字段**：输入会根据其他输入改变选项或可见性的表单。合成的 Schema 无法很好地表达这些依赖关系。
* **自定义 UI 组件**：依赖非标准输入（如 canvas、富文本编辑器）且不会自动序列化值的表单。
* **多步骤向导**：需要多次提交表单的复杂工作流。命令式 API 或标准 DOM 交互更适合此类场景。

## 何时使用 toolautosubmit
* **只读操作与查询**：搜索、筛选、获取详情或检查状态（例如，搜索车型、搜索目录、检查库存可用性）。
* **低风险、可逆操作**：用户可以轻松撤销或手动调整的表单操作（例如，将商品加入购物车、应用优惠码、保存草稿或设置临时布局选项）。

## 何时省略 toolautosubmit
* **破坏性或不可逆操作**：删除记录、重置系统配置或清空数据库。
* **财务与交易操作**：提交结账表单、转账、授权订阅付款或最终下单。
* **高影响的用户沟通**：提交最终求职申请、向其他真实用户发送邮件/消息或发布面向公众的内容。
* **敏感账户设置**：更改密码、修改用户角色/权限或更新账单/个人资料信息。

## 回退策略

{{ BASELINE_STATUS("declarative-webmcp") }}

WebMCP 声明式 API 在所有浏览器中均可安全使用。不支持 WebMCP 的浏览器会忽略 `tool*` 属性，`<form>` 将继续作为普通 HTML 表单运行。无需进行特性检测。