# 预期行为：`autofill-highlight-inputs`

- 必须使用 `:autofill` CSS 伪类为自动填充的表单控件设置样式。不得使用错误拼写 `:auto-fill`（带连字符）。
- `:autofill` 伪类只能应用于 `<input>`、`<select>` 或 `<textarea>` 元素。
- 如果同时使用 `:-webkit-autofill`，必须通过 `:is()` 与 `:autofill` 组合，不得在同一规则中作为单独选择器列出。
- 不得使用 JavaScript 为自动填充的输入框应用内联样式（例如在自动填充时设置 `element.style.*`）。
- 自动填充状态不得改变影响布局的属性（例如 `border-width`、`padding`、`font-size`）。
- 必须至少使用一种非颜色指示器为自动填充状态设置样式（例如通过 `box-shadow` 实现更粗的内嵌环）。
- 如果为自动填充状态应用自定义填充，必须使用内嵌 `box-shadow`，而非 `background-color`。
- 不得对自动填充元素应用 `outline: none`。
- 如果使用 `outline` 为自动填充元素设置样式，`:autofill:focus-visible` 状态的外轮廓必须在颜色之外有所差异（例如粗细、样式或偏移）。