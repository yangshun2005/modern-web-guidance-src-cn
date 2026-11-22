# 预期要求：`autofill-sign-in-form`

- `<input>` 元素必须位于 `<form>` 元素内。
- 表单必须使用 `<button>` 或 `<input type="submit">` 提供提交按钮。
- 表单中的每个 `<input>` 必须有可见的 `<label>` 元素标注。
- 每个 `<label>` 必须有 `for` 属性，其值与其关联 `<input>` 的 `id` 匹配。
- 邮箱输入必须为 `type="email"`。
- 邮箱输入必须为 `autocomplete="username"`。即使使用 `type="email"`，密码管理器也能识别 `username`，从而启用凭据自动填充。
- 密码输入必须为 `type="password"`。
- 密码输入必须为 `autocomplete="current-password"`。
- 邮箱和密码输入都必须有 `required` 属性。
- 必须恰好有一个邮箱输入（不要重复邮箱字段）。
- 必须恰好有一个密码输入（不要重复密码字段）。