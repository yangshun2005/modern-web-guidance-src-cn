# 可自定义的 select 列表框

## 回退方案

{# {{ BASELINE_STATUS("tmp-customizable-select-listbox") }} #}

可自定义的 select 列表框仅在 Chrome 145 中受支持。
旧版 Chrome 和 Firefox 确实支持比 Safari 更丰富的列表框样式，但仍受限于非自定义 `<select>` 的解析限制。

**特性检测：** `CSS.supports('appearance: base-select')` 可检测可自定义 select 的总体支持情况，但其中可能包含也可能不包含可自定义列表框支持。
要专门检测可自定义列表框，可将其作为前置判断，若返回 true，则向页面插入以下 HTML：

```html
<select size="2">
  <option>Option 1</option>
  <option>Option 2</option>
</select>
<select size="2" style="appearance: base-select">
  <option>Option 1</option>
  <option>Option 2</option>
</select>
```

然后检查第二个 select 是否以不同的尺寸渲染。

**回退策略：** 视使用场景而定：
1. 如果可接受样式受限或无样式的普通列表框，直接将其作为渐进增强使用。
2. 否则，完全不使用可自定义 select。