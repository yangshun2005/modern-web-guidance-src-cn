---
name: autofill-address-form
description: 构建一个带有正确 autocomplete 属性并支持自动填充的地址表单。
web-feature-ids:
  - input
---

# 构建遵循最佳实践的地址表单

创建一个让用户在桌面端和移动端都能尽可能轻松输入地址数据的表单。确保表单充分利用浏览器内置的自动填充、验证和数据输入约束功能。

## 如何实现

以下是构建成功地址表单最重要的准则。

### 使用有意义且有效的 HTML

充分利用专为创建表单而设计的元素和属性：

-   `<form>`、`<input>`、`<label>` 和 `<button>`
-   `type`、`autocomplete` 和 `inputmode`

这些属性能启用浏览器内置功能、提升无障碍性，并为标记增添语义。

### 使用 `<label>` 元素为数据输入表单字段添加标签

要为 `<input>`、`<select>` 或 `<textarea>` 添加标签，请使用 `<label>`。将标签的 `for` 属性设置为与输入框的 `id` 相同的值，即可将标签与输入框关联。

### 充分利用 HTML 属性

通过使用带有 `type` 属性的适当 `<input>` 元素，让用户轻松输入数据，在移动端提供正确的键盘，并启用浏览器内置的基本验证。

电子邮件地址始终使用 `type="email"`，电话号码始终使用 `type="tel"`。

```html
<!-- type="email"/"tel" gives mobile users the right keyboard and enables built-in validation -->
<input type="email" id="email" name="email" autocomplete="email" required>
<input type="tel" id="phone" name="phone" autocomplete="tel">
```

每个 `<input>`、`<select>` 和 `<textarea>` 元素都应具有适当的 `autocomplete` 属性，以提升无障碍性并帮助用户避免重复输入数据。

### 让按钮更有帮助

按钮使用 `<button>`。你也可以使用 `<input type="submit">`，但不要用 `div` 或其他随意元素充当按钮。按钮元素提供无障碍行为、内置表单提交功能，且易于设置样式。

为每个表单提交按钮赋予能说明其作用的值。对于结账流程中的每一步，使用描述性的行动号召，展示进度并使下一步显而易见。例如，将配送地址表单的提交按钮标记为 **继续付款**，而不是 **继续** 或 **保存**。

### 尽可能使用单个姓名输入框

允许用户使用单个输入框输入姓名，除非你有充分理由分别存储名、姓、称谓或其他姓名部分。使用单个姓名输入框可降低表单复杂度、支持复制粘贴，并使自动填充更简单。单个姓名输入框使用 `autocomplete="name"`。

```html
<input type="text" id="fullname" name="fullname" autocomplete="name">
```

允许国际姓名。验证时，避免使用仅匹配拉丁字符的正则表达式。仅限拉丁字符会排除姓名或地址包含非拉丁字母字符的用户。改为允许 Unicode 字母匹配——并确保后端在输入和输出时都能安全支持 Unicode。现代浏览器对正则表达式中的 Unicode 有良好支持。

### 支持多种地址格式

构建地址表单时，要意识到地址格式的多样性，即使在同一国家内也是如此。不要对"标准"地址做假设。

如有可能，使用单个 `<textarea>` 元素输入街道地址。

```html
<!-- textarea handles multi-line international address formats that split inputs can't accommodate -->
<textarea id="address" name="address" autocomplete="street-address" required></textarea>
```

这是适应各种本地和国际地址格式的最灵活选项。

如果你的数据模型需要单独的地址行，请每行使用一个 `<input>`，并分别使用 `address-line1`、`address-line2`（如有需要，还有 `address-line3`）`autocomplete` 标记，以便自动填充仍然有效。不要将街道地址拆分到缺少这些标记的输入框中，并且只将第一行设为必填。

```html
<!-- Alternative: one input per line, each with its own autocomplete token -->
<input type="text" id="address-line1" name="address-line1" autocomplete="address-line1" required>
<input type="text" id="address-line2" name="address-line2" autocomplete="address-line2">
```

### 帮助用户避免意外遗漏数据字段

为必填字段添加 `required` 属性。

```html
<input type="text" id="city" name="city" autocomplete="address-level2" required>
```

### 回退策略

{{ BASELINE_STATUS("input", "html.elements.input.autocomplete") }}

自动填充是一种渐进增强。在不支持自动填充的浏览器中，用户只需手动输入地址详情。语义化 HTML 约束（如 `type`、`inputmode` 和 `required`）仍会作为标准表单验证正常发挥作用。