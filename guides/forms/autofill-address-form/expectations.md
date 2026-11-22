# 预期要求：`autofill-address-form`

- `<input>`、`<select>` 和 `<textarea>` 元素必须位于 `<form>` 元素内。
- 每个表单控件必须使用 `<label>` 元素提供可见标签。
- 每个 `<label>` 必须带有 `for` 属性，其值与其关联表单控件的 `id` 匹配。
- 街道地址必须通过以下方式之一收集：单个带有 `autocomplete="street-address"` 的 `<textarea>`（首选），或分开的行输入，其中第一行带有 `autocomplete="address-line1"`（后续行使用 `address-line2`、`address-line3`）。街道地址不得拆分到缺少这些 `autocomplete` 标记的输入框中。
- 邮政编码输入必须带有 `autocomplete="postal-code"`。
- 不得对姓名或地址输入强制仅限拉丁字符。正则模式不得将输入限制为仅 `[a-zA-Z]`。
- 必填字段（至少包括姓名和街道地址 `<textarea>` 或第一行地址）必须带有 `required` 属性。