# 预期要求：`autofill-payment-form`

- `<input>`、`<select>` 和 `<textarea>` 元素**必须**位于 `<form>` 元素内。
- 表单中的每个 `<input>`、`<select>` 或 `<textarea>` 元素**必须**使用 `<label>` 元素提供可见标签。
- 每个 `<label>` 元素**必须**有 `for` 属性，其值与其关联输入框的 `id` 属性一致。
- 支付卡号**必须**使用单个 `<input>` 元素输入。不得将卡号拆分到多个输入框。
- 卡号输入框**必须**设置 `autocomplete="cc-number"`。
- 持卡人姓名输入框**必须**设置 `autocomplete="cc-name"`。
- 卡片有效期输入框**必须**设置 `autocomplete="cc-exp"`。
- 卡片安全码输入框**必须**设置 `autocomplete="cc-csc"`。
- 卡号输入框**必须**设置 `inputmode="numeric"`，以便移动端用户使用数字键盘。
- 卡片安全码输入框**必须**设置 `inputmode="numeric"`。
- 任何支付卡输入字段**不得**使用 `type="number"` 属性。
- 安全码输入框**不得**使用 `type="password"` 属性。
- 持卡人姓名输入框**不得**强制仅限拉丁字符（例如不得使用仅匹配 `[a-zA-Z]` 的 `pattern`）。
- 所有必填支付表单字段**必须**设置 `required` 属性。
- 有效期输入框**必须**有可见的格式提示（例如 `MM/YY`），渲染在输入框**上方**，以免被移动端键盘和自动填充弹窗遮挡，并通过 `aria-describedby` 与输入框关联。`placeholder` **不得**作为唯一的格式提示，也**不得**替代标签。