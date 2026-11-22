---
name: forms
description: 构建无障碍、安全且易用的 Web 表单的最佳实践。创建或修改表单、输入控件及提交流程时使用本指南。
---

# 表单

## 1. 语义结构与表单元素

### 指南

- **应** 使用 `<form>` 元素包裹用于数据收集的交互控件。
- **应** 将搜索表单和筛选控件包裹在 `<search>` 元素中。
- **应** 当页面有多个表单时，通过 `aria-labelledby`（指向表单标题）为 `<form>` 元素提供无障碍名称。
- **应** 对敏感数据和变更操作使用 `method="POST"`；对幂等请求（如搜索）使用 `method="GET"`。
- **应** 指定 `action` 属性以设置目标 URL。
- **应** 为每个表单控件指定 `name` 属性，以便提交时标识数据。
- **应** 使用语义化标签，如 `<button type="submit">`、`<textarea>`、`<select>` 及具体的 `<input>` 类型。
- **应** 在次要或辅助按钮上显式设置 `type="button"`，避免其默认为 type="submit"。
- **应** 使用 `<fieldset>` 和 `<legend>`（`<legend>` 作为第一个子元素）对相关控件分组。**不应** 在没有 `<legend>` 的情况下使用 `<fieldset>` 仅做视觉样式。
- **应** 在提交按钮上使用行动导向的语言（如"保存更改"）。

- **不应** 在其他 `<form>` 元素内嵌套 `<form>` 元素。
- **不应** 对敏感数据使用 `GET`（会在历史记录/日志中暴露数据）。
- **不应** 使用通用 `<div>`、`<span>` 或 `<a>` 元素作为表单控件。
- **不应** 对主提交按钮使用 `type="button"`。
- **不应** 在没有替代布局方案的情况下禁用 textarea 缩放。
- **不应** 使用 CSS 使表单控件的视觉顺序偏离其 DOM/键盘 `Tab` 顺序。
- **不应** 在用户更改 `<select>`、单选按钮或复选框时自动提交表单，除非有明确的提交操作。

### 代码示例

```html
<search>
  <form action="/search" method="GET">
    <label for="q">Search query</label>
    <input type="search" id="q" name="q">
  
    <fieldset>
      <legend>Search filters</legend>
      <input type="checkbox" id="exact" name="exact" value="true">
      <label for="exact">Exact match only</label>
      <input type="checkbox" id="archived" name="archived" value="true">
      <label for="archived">Include archived results</label>
    </fieldset>

    <button type="submit">Search</button>
  </form>
</search>
```

### 选择控件决策矩阵

| 选项数量 | 选择类型 | 推荐元素 | 可用性与无障碍逻辑 |
| :--- | :--- | :--- | :--- |
| **1** | 二元切换（是/否） | `<input type="checkbox">` | **可逆状态**：可用空格键勾选和取消，不像单选按钮一旦选中就无法取消。 |
| **2–5** | 单选（互斥） | `<input type="radio">` | **零点击浏览**：所有选项立即可见，浏览更快。 |
| **6+** | 单选（互斥） | `<select>` | **节省空间**：仅在垂直空间紧张或列表较长时使用。 |
| **10+ / 动态** | 单选（互斥） | `<input list="id">`（`<datalist>`） | **模糊搜索**：避免在大量选项（如国家）中滚动疲劳。 |
| **2+** | 多选 | `<input type="checkbox">` | **标准语义**：原生非互斥切换。 |

**一句话心智模型**："当选项少于六个时，将 2 到 5 个互斥选项以可见的单选按钮呈现；仅在空间受限或列表较长时使用 `<select>`。"

## 2. 无障碍标签与状态

### 指南

- **应** 始终使用 `for` 和 `id` 将 `<label>` 与其输入控件关联。
- **应** 在 DOM 中将 `<label>` 元素放在文本输入框、`<textarea>` 和 `<select>` 控件之前，并在视觉上堆叠在控件上方，以便更快浏览。
- **应** 将 `<label>` 元素紧跟在 `<input type="checkbox">` 和 `<input type="radio">` 控件之后内联放置，以保持一致对齐。
- **应** 使用可见标签，并将关键格式说明（如 `3-12 characters`）保留在标签或持久辅助文本中；不要仅依赖 `placeholder`。
- **应** 确保标签与其输入控件之间的垂直间距小于表单组之间的间距（**格式塔接近原则**）。
- **应** 对未通过自定义验证的表单控件设置 `aria-invalid="true"`，并在问题解决后清除。
- **应** 使用 `aria-describedby` 将输入控件与帮助文本或错误消息关联。
- **应** 在 `<html>` 上定义 `lang` 属性，以便设备正确翻译。
- **应** 使用非颜色视觉提示（图标、文本）传达状态（不要仅依赖颜色）。
- **应** 在标签中视觉上、并通过 `required` 属性在程序上清晰标明哪些字段为必填。
- **应** 使用 `aria-live` 进行动态错误播报。
  
- **不应** 在可以使用可见 `<label>` 时使用 `aria-label`，或用不匹配的 `aria-label` 覆盖可见文本。

### 代码示例

```html
<div class="field">
  <label for="username">Username: <span aria-hidden="true">*</span></label>
  <input type="text" id="username" name="username" aria-describedby="user-help" required>
  <span id="user-help" class="hint">3-12 characters.</span>
</div>

<style>
  .field {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.25rem;
  margin-block-end: 1.5rem;
  }

  input:focus-visible {
    outline: 3px solid #0b57d0;
    outline-offset: 2px;
  }
</style>
```

## 3. 自动填充与输入提示

### 指南

- **应** 使用 `autocomplete` 属性指定预期数据类型（如 `email`、`tel`、`current-password`、`new-password`）。
- **应** 使用 `inputmode` 优化屏幕键盘（如 PIN 码使用 `inputmode="numeric"`）。
- **应** 使用 `enterkeyhint` 设置回车键标签（如 `next`、`done`）。
- **应** 对存放标识符而非自然语言的字段关闭文本纠正：在用户名、邮箱、URL、代码和搜索输入上设置 `autocorrect="off"`（以及 `autocapitalize="none"` 和 `spellcheck="false"`）。对消息和评论等自由文本字段保留默认设置。
- **应** 对复杂数字（信用卡、电话）使用单字段输入，以帮助自动填充。
- **应** 使用单个 `<input type="text" inputmode="numeric" autocomplete="one-time-code">` 和 WebOTP API 进行短信一次性验证码验证。参见指南 {{ GUIDE_REF("sms-otp-form") }}。

- **不应** 对信用卡、邮政编码或一次性验证码使用 `type="number"`（会导致 UI 滚动问题并移除前导零）。
- **不应** 将单个值（如 6 位 OTP、电话号码或信用卡号）拆分到多个按键时自动前进焦点的 `<input>` 框中。应使用 `type="text"` 配合 `inputmode="numeric"`（`type=number` 会添加微调按钮）。如有需要，可通过样式实现类似 UI。JS 可设置 `--value-length` CSS 变量，实现更高保真度的样式以模拟这些输入框，同时避免其可用性和无障碍问题。
- **不应** 阻止复制粘贴（`paste` 事件）或在密码、验证码或其他认证字段上禁用 `autocomplete`。

### 代码示例

```html
<label for="zip">ZIP Code:</label>
<input type="text" id="zip" name="zip" autocomplete="postal-code" inputmode="numeric" pattern="\d{5}" enterkeyhint="next">

<label for="username">Username:</label>
<input type="text" id="username" name="username" autocomplete="username" autocorrect="off" autocapitalize="none" spellcheck="false" enterkeyhint="done">
```

## 4. 约束与验证

### 指南

- **应** 使用原生约束：`required`、`minlength`、`maxlength`、`pattern`。
- **应** 使用 CSS 伪类 `:user-invalid` 进行非侵入式样式设置。
- **应** 使用 ValidityState API（`setCustomValidity`）进行自定义消息提示，并编写清晰的错误消息说明如何修复问题。
- **应** 当表单提交验证失败时，将焦点移至第一个无效字段（或错误摘要）。

- **不应** 单独使用原始 `:invalid` CSS 伪类进行错误样式设置，以确保空的必填字段在用户与表单交互之前不会被标记为无效。
- **不应** 通过禁用提交按钮来阻止验证；让用户提交并高亮错误。但是，**应** 在有效提交被点击*之后*禁用按钮以防止重复提交。
- **不应** 在用户输入时每次按键都触发验证错误或 `aria-live` 错误播报（在提交或 `blur` 时验证，问题解决后在 `input` 时清除错误）。

### 代码示例

```html
<label for="code">Activation Code (4 digits):</label>
<input type="text" id="code" name="code" required pattern="\d{4}">

<script>
  const input = document.getElementById('code');
  input.addEventListener('invalid', () => {
    input.setCustomValidity('Please enter exactly 4 digits.');
  });
  input.addEventListener('input', () => {
    input.setCustomValidity('');
  });
</script>
```

### 验证事件时机矩阵

| 事件触发 | 阶段 | 允许的操作 | UX / 无障碍逻辑 |
| :--- | :--- | :--- | :--- |
| **`input`** | 正在输入 | **仅清除**现有错误。 | **非侵入式**：不要在用户输入完成前就报错。 |
| **`blur` / `focusout`** | 离开字段 | **运行**检查（若已编辑）并显示错误。 | **上下文验证**：当用户表示已"完成"某字段时进行验证。 |
| **`submit`** | 最终尝试 | **阻止**并引导焦点。 | **最终把关**：拦截错误载荷，将键盘和屏幕阅读器焦点移至第一个无效字段或错误摘要。 |

**一句话心智模型**："在 `blur`（若已编辑）时验证以避免输入时的过早警告，在用户修正值后立即在 `input` 时重置错误状态，在 `submit` 时将焦点引导至第一个错误或摘要。"

**安全与 UX 权衡**：客户端验证用于用户体验；服务端验证用于安全。永远不要将浏览器约束视为数据完整性的防御手段。

## 5. 响应式设计与排版

### 指南

- **应** 使用单列布局以便浏览。
- **应** 在表单控件上设置 `font: inherit` 和 `font-size: max(1rem, 1em)`，使其随更大容器缩放，同时保持至少 `1rem`（16px）以防止 iOS 缩放。
- **应** 使用内边距技巧扩大移动端点击目标的可点击区域。
- **应** 确保点击目标至少为 `48px`。
- **应** 使用相对于根元素的单位（`rem`）和无单位的 `line-height`。
- **应** 使用 CSS 逻辑属性（如 `margin-inline-start`）以支持 RTL。

### 代码示例

```css
.form-group {
  margin-block-end: 1.5rem;
}

/* Expand clickable tap area without layout shift */
label {
  display: inline-block;
  padding: 10px 0;
  margin: -10px 0;
}

input {
  font: inherit;
  font-size: max(1rem, 1em);
  padding: 0.75rem;
  min-height: 48px;
  box-sizing: border-box;
}

@media (pointer: coarse) {
  input {
    min-height: 52px;
  }
}
```

## 6. 表单控件样式

### 指南

- **应** 使用 `accent-color` 为原生单选按钮/复选框快速设置品牌色。
- **应** 使用 `appearance: none` 自定义下拉箭头而不破坏语义。
- **应** 确保输入框清晰可见，边框对比度充足（如白色背景上使用 `#767676` 或更深颜色）。
- **应** 在所有控件上保持可见的 `:focus-visible` 指示器。永远不要在没有提供高对比度替代方案的情况下禁用焦点轮廓（`outline:none`）。
- **应** 使用标准 `.visually-hidden` 方案（`clip-path: inset(50%)` 配合 1px 尺寸）在视觉上隐藏输入框——而非 `display: none`，后者会将其从无障碍树中移除。

### 代码示例

```html
<div class="checkbox-container">
  <input type="checkbox" id="sub" name="sub" class="visually-hidden">
  <label for="sub" class="checkbox-label">Subscribe</label>
</div>

<style>
  .visually-hidden {
    position: absolute;
    clip-path: inset(50%);
    overflow: hidden;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    border: 0;
    white-space: nowrap;
  }
  .checkbox-label::before {
    content: "";
    display: inline-block;
    width: 1.25rem;
    height: 1.25rem;
    border: 2px solid #767676;
  }
  input:focus-visible + .checkbox-label::before {
    outline: 2px solid #0b57d0;
  }
</style>
```

## 7. JavaScript 与 AJAX

### 指南

- **应** 在聊天或文本输入中将 `Enter` 键视为提交操作之前检查 `KeyboardEvent.isComposing`，以确保 IME 文本组合不被打断。参见指南 {{ GUIDE_REF("ime-safe-enter-submit") }}。
- **应** 在 AJAX 表单提交时阻止默认导航（`e.preventDefault()`）。
- **应** 使用 `ValidityState` 接口进行实时验证检查。
- **应** 通过 `role="status"` 或 `aria-live` 播报 AJAX 提交结果。
- **应** 对动态 UI 展示使用 `aria-expanded` 和 `aria-controls`。

- **不应** 在 JS 失败时阻止页面提交；确保服务端回退。

### 代码示例

```js
form.addEventListener('submit', (e) => {
  e.preventDefault();
  const data = new FormData(form);
  // fetch('/submit', { method: 'POST', body: data });
});
```

## 8. 身份、支付与高级安全

### 指南

- **应** 注册时使用 `autocomplete="new-password"`，登录时使用 `autocomplete="current-password"`。
- **应** 允许粘贴到密码字段。
- **应** 提供带 `aria-pressed` 的无障碍切换 `<button type="button">`，允许用户取消密码掩码。
- **应** 在支付按钮上标明确切金额（如"支付 $100"）。
- **应** 使用 `autocomplete="cc-number"`、`cc-exp`、`cc-csc`。
- **应** 所有页面使用 HTTPS。
- **应** 对变更操作（POST/PUT/DELETE）实现加密安全的反 CSRF 令牌。
- **应** 在将用户输入注入 DOM 之前进行净化（如通过 DOMPurify）以防止 XSS。
- **应** 对开放表单实施垃圾信息防护，使用无障碍 CAPTCHA（如隐形 reCAPTCHA）或对辅助技术隐藏的蜜罐字段（`tabindex="-1"` 和 `aria-hidden="true"`）

- **不应** 对执行状态更改的端点使用 HTTP `GET`。
- **不应** 在表单标记中直接使用内联 JavaScript（如 `onclick="..."`），以满足严格的内容安全策略（CSP）。

### 代码示例

```html
<form method="post">
  <input type="hidden" name="csrf_token" value="secure_token_abc123">

  <h1>Sign up</h1>

  <div class="form-group">        
    <label for="name">Full name</label>
    <input id="name" name="name" autocomplete="name" required pattern="[\p{L}\.\- ]+">
  </div>

  <div class="form-group">        
    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="username" required>
  </div>

  <div class="form-group">
    <label for="password">Password</label>
    <button id="toggle-password" type="button" aria-pressed="false" aria-label="Show password" aria-describedby="toggle-warning">
      <img class="icon-eye" src="/icons/eye.svg" alt="" width="20" height="20">
      <img class="icon-eye-off" src="/icons/eye-off.svg" alt="" width="20" height="20">
    </button>
    <span id="toggle-warning" class="visually-hidden">Warning: this will display your password on the screen.</span>
    <input id="password" name="password" type="password"
```