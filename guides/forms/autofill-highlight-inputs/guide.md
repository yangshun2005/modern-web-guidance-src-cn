---
name: autofill-highlight-inputs
description: 自定义已被自动填充且用户未编辑的表单字段样式
web-feature-ids:
  - autofill
  - text-stroke-fill
---

# 自定义自动填充表单字段的样式

使用 CSS `:autofill` 伪类替换浏览器对自动填充的 `<input>`、`<select>` 和 `<textarea>` 元素的默认高亮。它匹配由浏览器填充或用户选择自动填充建议的字段，一旦用户编辑该字段就不再匹配，即使值完全相同。

它从不匹配复选框、单选按钮、文件或按钮输入、表单关联的自定义元素或 `contenteditable`：浏览器不会自动填充这些。自定义元素 shadow DOM 内的原生控件会匹配；可从内部设置样式，或通过 `::part(name):autofill`。

## 如何实现

- 至少添加一个非颜色指示器（例如更粗的内嵌环）。浏览器默认高亮仅使用颜色。
- 不要移除焦点轮廓（`outline: none`）。如果用 `outline` 实现自动填充样式，请为 `:autofill:focus-visible` 设置一个在颜色之外也有差异的轮廓（例如粗细、样式或偏移），以便焦点仍可感知。
- 不要修改影响布局的属性（`border-width`、`padding`）；该状态随用户操作切换，否则会导致内容位移。

```css
input:autofill {
  /* 无布局位移的非颜色指示器：更粗的内嵌环 */
  box-shadow: inset 0 0 0 2px var(--autofill-ring-color);
  border-color: var(--autofill-ring-color);
}
```

### 覆盖背景色和文字颜色

建议保留浏览器的背景色和文字颜色。用户已习惯默认高亮表示"此字段已自动填充"，替换它会破坏这种关联。
所有浏览器还会在用户代理样式表中以 `!important` 为自动填充字段设置 `background-color`、`background-image` 和 `color`，因此覆盖它们需要依赖变通方法。

如果设计确实需要：

```css
input:autofill {
  /* 填充：内嵌 box-shadow 覆盖 UA 的 `!important` 背景 */
  box-shadow:
    inset 0 0 0 2px var(--autofill-ring-color),
    inset 0 0 0 100vmax var(--autofill-background-color);
  /* 文字颜色：UA 未设置 -webkit-text-fill-color */
  -webkit-text-fill-color: var(--autofill-text-color);
  /* 光标仍跟随 UA 的 `color`，因此需显式设置 */
  caret-color: var(--autofill-text-color);
}
```

注意事项：
- UA 文字颜色为 `FieldText`：浅色方案下为黑色，深色方案下为白色。确保填充色在其上清晰可读（例如通过 `light-dark()`），以便即使文字颜色覆盖未生效，字段仍可读。
- 填充绘制在浏览器颜色之上，而非页面背景之上，因此请使用不透明的 `--autofill-background-color`。
- 该规则会替换字段上的其他 `box-shadow`（例如用 `box-shadow` 实现的焦点环）；请在同一声明中重复这些阴影。
- `-webkit-text-fill-color` 仅改变绘制的文字。`color` 以及由此派生的 `currentColor` 仍解析为 UA 的 `FieldText`。

## 回退策略

{{ BASELINE_STATUS("autofill") }}

{# https://bugzilla.mozilla.org/show_bug.cgi?id=1923525 #}

Firefox 仅在自动填充的用户名和密码字段上匹配 `:autofill`（bug 1923525）。无变通方法；其他字段保留默认色调。

Chrome 110 之前和 Safari 15 之前仅支持 `:-webkit-autofill`。如果它们在你的 Baseline 目标内，请将两个名称包裹在 `:is()` 中，以免无法识别的伪类使整条规则失效：

```css
input:is(:autofill, :-webkit-autofill) {
  /* 与上述声明相同 */
}
```

这是渐进增强：不支持的浏览器保留默认高亮。不要添加 JavaScript 回退。