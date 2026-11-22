## 用法

- 测试布局行为。设置 `appearance: base-select` 会移除浏览器默认行为，即根据最长选项宽度来确定 select 尺寸。你可能需要设置固定宽度或使用 flex/grid 约束来防止布局偏移。
- 确保 `<select>` 具有 `name` 属性和关联的 `<label>`。这样即使使用自定义 UI，组件对屏幕阅读器仍可访问，并能与标准表单提交正常工作。
- 在选项上使用标准 `value` 属性，确保表单提交与之前完全一致。

## 回退方案

对于尚不支持 `appearance: base-select` 的浏览器，`<select>` 元素会优雅降级为标准操作系统下拉框。

- **非文本内容被忽略**：旧版浏览器会剥离 `<option>` 标签内的 HTML 标签（如 `<svg>` 或 `<div>`），仅渲染文本节点。确保 `<option>` 的文本内容本身可读且有意义。
- **HTML 结构处理**：标准解析器可能忽略 `<select>` 内的 `<button>` 和 `<selectedcontent>` 标签，或将其视为无效。如果将标准文本视为可读的回退方案，则无需为渐进增强引入大量 JavaScript polyfill。


```javascript
document.addEventListener("DOMContentLoaded", () => {
  // Check if browser supports base-select value
  if (!CSS.supports("appearance", "base-select")) {
    // Custom select overrides are not supported natively.
  }
});
```