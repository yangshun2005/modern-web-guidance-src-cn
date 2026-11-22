---
name: autofill-payment-form
description: 构建一个支付表单，收集银行卡信息，并使用正确的 autocomplete 值和自动填充支持。
web-feature-ids:
  - enterkeyhint
  - input-email-tel-url
  - inputmode
---

# 构建遵循最佳实践的支付表单

支付表单是结账流程中最关键的一环。支付表单设计不佳是购物车被放弃的常见原因。

创建一个表单，让用户在桌面端和移动端都能尽可能轻松地输入支付信息。确保表单充分利用浏览器内置的自动填充、验证和数据输入约束功能。

## 如何实现

以下是构建成功支付表单的最重要准则。

### 使用有意义且有效的 HTML

充分利用专为创建表单而设计的元素和属性：

-   `<form>`、`<input>`、`<label>` 和 `<button>`
-   `type`、`autocomplete` 和 `inputmode`

这些属性能启用浏览器内置功能、提升可访问性，并为标记增添语义。

### 使用 `<label>` 元素为数据输入字段添加标签

要为 `<input>`、`<select>` 或 `<textarea>` 添加标签，请使用 `<label>`。将标签的 `for` 属性设置为与输入框的 `id` 相同的值，即可将标签与输入框关联。

### 充分利用 HTML 属性

使用带有 `type` 属性的适当 `<input>` 元素，让用户轻松输入数据：在移动端提供正确的键盘，并启用浏览器内置的基本验证。

电子邮件地址始终使用 `type="email"`，电话号码始终使用 `type="tel"`。

```html
<!-- type="email"/"tel" gives mobile users the right keyboard and enables built-in validation -->
<input type="email" id="email" name="email" autocomplete="email" required>
<input type="tel" id="phone" name="phone" autocomplete="tel">
```

每个 `<input>`、`<select>` 和 `<textarea>` 元素都应具有适当的 `autocomplete` 属性，以提升可访问性并帮助用户避免重复输入数据。

### 让按钮更有帮助

按钮请使用 `<button>`。也可以使用 `<input type="submit">`，但不要用 `div` 或其他随意元素充当按钮。按钮元素提供可访问的行为、内置的表单提交功能，且易于设置样式。

为每个表单提交按钮赋予能说明其作用的值。对于结账的每一步，使用描述性的行动号召，展示进度并让下一步一目了然。例如，配送地址表单的提交按钮应标注为 **继续付款**，而不是 **继续** 或 **保存**。

### 尽可能使用单个姓名输入框

允许用户使用单个输入框输入姓名，除非你有充分理由分别存储名、姓、称谓或其他姓名部分。使用单个姓名输入框可以降低表单复杂度、支持复制粘贴，并简化自动填充。姓名使用单个输入框时，设置 `autocomplete="name"`。

```html
<input type="text" id="fullname" name="fullname" autocomplete="name">
```

允许国际姓名。验证时，避免使用仅匹配拉丁字符的正则表达式。仅限拉丁字符会排除姓名或地址中包含非拉丁字母字符的用户。应允许 Unicode 字母匹配——并确保后端在输入和输出时都能安全支持 Unicode。现代浏览器对正则表达式中的 Unicode 有良好支持。

### 兼容多种地址格式

在支付表单中添加地址字段时，要注意地址格式的多样性，即使在同一国家内也是如此。不要对"正常"地址做假设。

如果在数据要求允许的范围内，考虑使用单个 `<textarea>` 元素作为地址输入。这是兼容各种本地和国际地址格式的最灵活方案。

### 为账单地址使用自动填充

默认将账单地址设置为与配送地址相同。通过提供编辑账单地址的链接（或使用 summary 和 details 元素）来减少视觉杂乱，而不是在表单中显示账单地址。

为账单地址使用适当的 autocomplete 值，就像为配送地址所做的那样，这样用户就不必重复输入数据。如果不同区块中同名输入框的值不同，请在 autocomplete 属性中添加前缀词。例如：

```html
<!-- Prefix with shipping or billing; note that the valid token is address-line1 (no hyphen before the digit) -->
<input autocomplete="shipping address-line1" ...>
...
<input autocomplete="billing address-line1" ...>
```

### 显示结账进度

对于付款的每一步，使用页面标题和描述性的按钮值，清楚说明当前需要做什么以及下一步是什么结账步骤。

在表单输入框上使用 `enterkeyhint` 属性设置移动端键盘回车键的标签。例如，在多页表单中使用 `enterkeyhint="previous"` 和 `enterkeyhint="next"`，表单最后一个输入框使用 `enterkeyhint="done"`，搜索输入框使用 `enterkeyhint="search"`。

### 帮助用户避免重复输入支付数据

确保在支付卡表单中添加适当的 `autocomplete` 值。没有自动填充，用户可能会保留支付卡信息的纸质记录，或以不安全的方式存储。

```html
<!-- cc-number tells autofill this is a card number, not a generic number field -->
<!-- inputmode="numeric" gives a numeric keyboard without the increment/decrement spinner -->
<!-- DO NOT use type="number" — it adds increment/decrement controls and strips leading zeros -->
<input id="cc-number" name="cc-number" type="text" autocomplete="cc-number"
       inputmode="numeric" maxlength="19" pattern="[\d ]{13,19}" required>

<!-- cc-name autofills with the name exactly as it appears on the card; Unicode pattern allows international names -->
<input id="cc-name" name="cc-name" type="text" autocomplete="cc-name"
       maxlength="50" pattern="[\p{L} \-\.]+" required>

<!-- cc-exp autofills the full expiry date as MM/YY -->
<!-- MANDATORY: Place format hints above the input so autocomplete popovers or virtual keyboards do not obscure them during editing -->
<span id="exp-hint" class="hint">Format: MM/YY</span>
<input id="cc-exp" name="cc-exp" type="text" autocomplete="cc-exp"
       aria-describedby="exp-hint" maxlength="5" required>

<!-- cc-csc autofills the security code; DO NOT use type="password" here -->
<input id="cc-csc" name="cc-csc" type="text" autocomplete="cc-csc"
       inputmode="numeric" maxlength="4" pattern="[0-9]{3,4}" required>
```

### 在输入框上方显示格式提示

如果字段有必需的格式（例如 `MM/YY` 有效期），请将提示放在输入框**上方**的可见元素中，并用 `aria-describedby` 将两者关联，如上面的 `cc-exp` 示例所示。位于输入框下方的提示很容易在用户需要时被移动端键盘或自动填充弹窗遮挡。

不要依赖 `placeholder` 属性作为唯一的格式提示，也绝不要用它代替 `<label>`：占位符文本在用户开始输入后就会消失，因此编辑时无法对照检查。占位符可以作为标签和常驻提示之外的额外线索。

### 不要遮蔽安全码

卡安全码使用 `type="text"` 配合 `inputmode="numeric"`，不要使用 `type="password"`。用户无法检查被遮蔽的三四位安全码是否有拼写错误，因此输错的数字要到付款被拒时才会被发现；而且遮蔽几乎没有保护作用，因为安全码就印在用户手中的卡上。

### 支付卡号和电话号码使用单个输入框

支付卡号和电话号码使用单个输入框：不要将号码拆分成多段。这样用户更容易输入数据、验证更简单，也能让浏览器自动填充。对于 PIN 码和银行代码等其他数字数据，也可以考虑同样处理。

### 仔细验证

应在实时和表单提交前都验证数据输入。一种方法是为支付卡输入框添加 pattern 属性。如果用户尝试提交包含无效值的支付表单，浏览器会显示警告消息并将焦点设置到该输入框。

但是，pattern 正则表达式必须足够灵活，以处理支付卡号长度的范围：从 14 位（或可能更少）到 20 位（或更多）。卡安全码（也称为 CSC、CVC、CVV 或其他名称）由 3 或 4 位数字组成。

允许用户在输入新支付卡号时包含空格，因为实体卡上就是这样显示号码的。这对用户更友好（你不必告诉他们"做错了什么"），更不容易中断转化流程，而且在处理前去除号码中的空格也很简单。

### 帮助用户避免意外遗漏数据字段

为必填字段添加 `required` 属性。现代浏览器会自动提示并聚焦缺失的数据。

## 回退策略

{{ BASELINE_STATUS("enterkeyhint") }}
{{ BASELINE_STATUS("input-email-tel-url") }}
{{ BASELINE_STATUS("inputmode") }}

自动填充是一种渐进增强。在不支持自动填充的浏览器中，用户只需手动输入支付信息。语义化 HTML 约束（如 `type`、`inputmode`、`pattern` 和 `required`）仍会正常发挥作用，验证用户输入并提供正确的虚拟键盘。