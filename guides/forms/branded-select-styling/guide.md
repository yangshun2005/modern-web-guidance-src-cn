---
name: branded-select-styling
description: 创建自定义 select 元素，使其按钮、选择器、箭头图标和勾选标记在排版、颜色、间距和边框处理上都与你的品牌或设计系统无缝匹配。
web-feature-ids:
  - customizable-select
---

# 品牌化 Select 样式

可自定义 select API 提供了一种声明式、由 CSS 驱动的方式，让 `<select>` 元素完美匹配品牌的设计系统。启用 `appearance: base-select` 后，你就能访问 select 元素的内部 shadow DOM，从而用标准 CSS 属性为按钮、选项选择器列表、箭头图标和勾选标记设置样式。

过去，要实现完全品牌化的 select，需要用 JavaScript 从头重建控件，这往往会破坏无障碍性、键盘导航和原生表单集成。使用 `appearance: base-select` 后，你既能获得自定义外观，又由浏览器负责焦点管理、顶层渲染和无障碍绑定。

## 实现方法

要实现品牌化 select 样式：

1. **启用自定义：** 对 `<select>` 元素和 `::picker(select)` 伪元素（用于定位选项下拉列表）都应用 `appearance: base-select`。
2. **构建自定义按钮（可选）：** 在 `<select>` 内直接定义 `<button>` 元素，替换默认触发器。在此按钮内使用 `<selectedcontent>` 元素来表示当前选中选项的文本或内容。
3. **为选择器列表设置样式：** 使用 `::picker(select)` 伪元素为下拉列表应用排版、背景色、边框和阴影。浏览器会在顶层渲染它，`z-index` 冲突从此成为过去。
4. **为内部图标设置样式：**
   - 使用 `select::picker-icon` 设置或替换箭头图标。
   - 使用 `option::checkmark` 设置当前选项旁勾选标记的样式。
5. **为选项设置样式：** 对 `<option>` 元素应用样式，处理悬停状态、内边距和布局。

## 示例代码：品牌化快递 Select

以下示例展示了一个自定义 select，使用等宽字体和虚线边框，以匹配特定的"包裹"品牌美学。

```css
/* 为 select 及其选择器启用自定义 */
.brand-select,
.brand-select::picker(select) {
  appearance: base-select;
}

/* 为可见的触发按钮设置样式 */
.brand-select {
  font-family: 'Courier New', monospace;
  background-color: #fffaf0;
  color: #8b4513;
  border: 2px dashed #8b4513;
  border-radius: 4px;
  padding: 0.75rem;
  font-size: 1rem;
  cursor: pointer;
}

/* 为下拉选项列表设置样式 */
.brand-select::picker(select) {
  font-family: 'Courier New', monospace;
  background-color: #fffaf0;
  border: 2px dashed #8b4513;
  border-radius: 4px;
  padding: 0.5rem;
}

/* 自定义内部部件颜色以匹配文本 */
.brand-select::picker-icon {
  color: #8b4513;
}

.brand-select option::checkmark {
  color: #8b4513;
}

/* 为各个选项和悬停效果设置样式 */
.brand-select option {
  padding: 0.5rem;
  border-radius: 4px;
  color: #8b4513;
  cursor: pointer;
}

.brand-select option:hover {
  background-color: #fdf5e6;
}
```

```html
<label for="preferences">Select shipping preference</label>
<select class="brand-select" id="preferences" name="preferences">
  <button>
    <selectedcontent></selectedcontent>
  </button>
  <option value="standard">Standard Shipping</option>
  <option value="express" selected>Express Shipping</option>
  <option value="overnight">Overnight Delivery</option>
</select>
```

## 策略性实现与最佳实践

- **应当**在设计系统要求所有表单控件保持高保真视觉一致性、而标准跨浏览器 select 覆盖无法实现时，使用 `appearance: base-select`。
- **不应**在依赖操作系统原生选择器体验时使用（例如 iOS 设备上的标准滚轮选择器）。启用 `base-select` 即放弃原生移动端 UI 控件，改用 Web 渲染的顶层菜单。
- **应当**验证颜色对比度符合 WCAG 标准（与背景色至少达到 `3:1`）。可自定义选择器允许你设置临时颜色，但你需要负责确保文本在自定义背景上保持清晰可读。
{{ FEATURE("customizable-select", "usage") }}

## 回退策略

{{ FEATURE_FALLBACKS("customizable-select") }}