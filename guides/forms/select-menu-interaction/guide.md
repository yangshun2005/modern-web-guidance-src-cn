---
name: select-menu-interaction
description: 仅在用户与控件交互后，才校验下拉菜单中是否选择了非默认选项。
web-feature-ids:
  - user-pseudos
---

# 下拉菜单交互

## 问题
对于必填下拉菜单（例如“选择国家/地区”），如果默认选项的值为空，标准校验会立即将该字段标记为无效，造成视觉干扰。我们希望仅在用户与菜单交互后未选择选项就离开，或尝试提交表单时未选择选项的情况下才显示错误。

## 解决方案
`:user-invalid` 伪类可与 `<select>` 元素无缝配合。它遵循用户的交互流程：仅加载页面，或聚焦/失焦而未做更改，都不算作交互，因此字段保持中性状态，直到用户主动尝试选择。

### 实现策略

1.  **HTML 约束**：使用带 `required` 的 `<select>`。第一个选项应设置 `value=""`，最好禁用/隐藏，以强制用户做出有效选择。
2.  **视觉反馈**：使用 `:user-invalid` 设置选择框边框样式。
3.  **时机**：如果用户在失焦前更改了值（即使改回默认的无效状态），或在表单提交时，浏览器会认为该字段已“交互”。部分浏览器也会将打开并关闭菜单而未选择选项计为交互。
4.  **错误关联**：仅在选择无效时，将错误消息列入 select 的 `aria-describedby`，以便屏幕阅读器随 select 一起朗读。若在标记中直接引用，页面加载时就会朗读隐藏的消息。“其他注意事项”中“无障碍”部分的脚本会为每个 `data-error` 属性设为其错误消息 `id` 的字段执行此操作。

## 实现指南

### 1. HTML 结构
“占位符”选项是关键。

```html
<form>
  <div class="field">
    <label for="country">国家/地区</label>
    <!-- 将 data-error 设为错误消息的 id。aria-invalid 脚本会在 select 无效时将该 id 列入 aria-describedby。 -->
    <select
      id="country"
      name="country"
      required
      data-error="country-error"
    >
      <option value="" disabled selected>请选择国家/地区...</option>
      <option value="us">美国</option>
      <option value="ca">加拿大</option>
      <option value="uk">英国</option>
    </select>
    <div id="country-error" class="error-msg">
      <span role="img" aria-label="错误">❌</span> 请选择国家/地区。
    </div>
  </div>
</form>
```

### 2. CSS
```css
.error-msg {
  display: none;
  color: #d93025;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

/*
  仅在用户访问下拉菜单后显示错误。
*/
select:user-invalid {
  border-color: #d93025;
  background-color: #fce8e6;
}

select:user-invalid + .error-msg {
  display: block;
}

select:user-valid {
  border-color: #188038;
}
```

## 回退与浏览器支持

{{ FEATURE_FALLBACKS("user-pseudos") }}

## 其他注意事项

1.  **移动端行为**：在移动设备上，“失焦”的触发方式可能因操作系统选择器而异。建议在真实设备上测试。
2.  **无障碍**：{{ FEATURE("user-pseudos", "aria-invalid") }}