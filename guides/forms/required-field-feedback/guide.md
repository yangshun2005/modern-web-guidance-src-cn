---
name: required-field-feedback
description: 仅在用户交互后，为被跳过或留空的必填表单字段提供错误消息，避免过早报错，确保反馈及时且与用户操作流程相关。
web-feature-ids:
  - user-pseudos
---

# 必填字段反馈

## 问题
页面加载时立即将必填字段标记为错误状态会令人困惑。理想情况下，必填字段只有在用户尝试填写但失败后才应显示为"无效"。

## 解决方案
`:user-invalid` 伪类完美解决了这个问题。对于必填字段，它不会在页面加载时匹配。仅在以下情况匹配：
1.  用户与字段交互（例如输入一个字符后删除）然后离开（失焦），使字段留空。
2.  用户在字段为空时尝试提交表单。

### 实现策略

1.  **HTML 约束**：为输入框添加 `required` 属性。
2.  **视觉反馈**：使用 `:user-invalid` 将边框样式设为红色并显示"必填"辅助文本。
3.  **时机**：依赖浏览器原生的视觉反馈时机。不再需要 `onBlur` 处理器来添加 `touched` 类，但仍需一些 JavaScript 来同步 ARIA 属性（见下文）。
4.  **错误文本**：显示可见的错误文本并配合错误样式，避免仅通过颜色传达错误。每条消息前加一个带文本替代（如"Error"）的错误图标（最安全的选择），或将每条消息措辞为错误提示，例如"Name is required."，并对表单中所有错误使用相同方式。
5.  **错误链接**：仅在字段无效时将错误消息列入字段的 `aria-describedby`，以便屏幕阅读器随字段一起朗读。若在标记中引用，则会在页面加载时就朗读隐藏消息。第 3 节的脚本会为每个 `data-error` 属性设为其错误消息 `id` 的字段执行此操作。

## 实现指南

### 1. HTML 结构
```html
<form id="feedback-form">
  <div class="field">
    <label for="full-name">Full Name</label>
    <!-- Set data-error to the error message's id. The aria-invalid script lists that id in aria-describedby while the field is invalid. -->
    <input
      type="text"
      id="full-name"
      name="full-name"
      required
      data-error="name-error"
    >
    <!-- MANDATORY: Include an icon or distinct non-color indicator alongside error text -->
    <div id="name-error" class="error-msg">
      <span role="img" aria-label="Error">❌</span> This field is required.
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
  Only highlight empty required fields AFTER the user interacts with them or attempts submission.
  MANDATORY: Provide multiple indicators (border shift + helper text/icon) to avoid color-only state communication.
*/
input:user-invalid {
  border-color: #d93025;
  background-color: #fce8e6;
}

input:user-invalid + .error-msg {
  display: block;
}

/* Optional: Subtle indicator for required fields that are valid */
input:required:user-valid {
  border-color: #188038;
  border-width: 2px;
}
```

### 3. JavaScript 状态同步

强制要求：由于 `:user-invalid` 是视觉状态，当用户使无效字段失焦或尝试提交时，必须提供 JavaScript 桥接来动态同步 `aria-invalid="true"` 以支持辅助技术。该状态应派生自 `:user-invalid`，而非 `checkValidity()`——后者在用户用 Tab 键经过未触碰的字段时已为 false。

{{ FEATURE("user-pseudos", "aria-invalid") }}

## 回退与浏览器支持

{{ FEATURE_FALLBACKS("user-pseudos") }}

## 其他注意事项

1.  **星号**：仍建议在标签中视觉标记必填字段（例如使用星号 `*`），让用户在交互*之前*就知道预期。在表单前解释一次星号含义（例如"标有 * 的字段为必填。"），并将其包裹在 `<span aria-hidden="true">*</span>` 中，因为 `required` 属性已让屏幕阅读器朗读"required"。
2.  **提交按钮**：与 `disabled` 按钮不同，保持提交按钮启用。用户点击时，浏览器会自动对所有空的必填字段触发 `:user-invalid` 并聚焦第一个。这对可访问性和用户体验都非常好。