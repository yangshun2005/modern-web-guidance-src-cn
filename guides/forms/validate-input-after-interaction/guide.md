---
name: validate-input-after-interaction
description: 仅在用户完成初始交互后显示表单字段验证反馈（如密码复杂度或邮箱格式要求），避免在页面加载时或用户输入过程中过早报错。
web-feature-ids:
  - user-pseudos
---

# 在交互后验证输入

## 问题

用户刚聚焦字段并开始输入时就显示验证错误，既过早又分散注意力。例如，用户输入邮箱地址（如 "user@gm"）或需要满足复杂要求的密码时，在完成之前字段技术上都是无效的。标准的 `:invalid` 样式会立即显示错误状态，令用户沮丧。

## 解决方案

`:user-invalid` 伪类可将错误状态推迟到用户"确认"了某个值（通过修改并失焦字段）或尝试提交表单之后。这样确保验证反馈只在用户完成字段交互后才出现。

### 实施策略

1.  **HTML 约束**：务必使用 `type="email"`、`pattern`、`required` 等标准 HTML5 属性来触发浏览器内置验证逻辑。
2.  **视觉反馈**：务必使用 `:user-invalid` 仅在交互后应用错误样式。
3.  **正向强化**：可选使用 `:user-valid` 在满足要求后显示绿色"成功"指示。
4.  **优雅恢复**：用户一旦将输入修正为有效格式，`:user-invalid` 立即停止匹配，错误状态即刻移除。
5.  **错误文本**：务必在显示错误样式的同时显示可见的错误文本，不能仅靠颜色传达错误。务必让每条消息以带文本替代（如 "Error"，最安全的选择）的错误图标开头，或将每条消息措辞为错误提示，例如 "Please enter a valid email address."，并对表单中所有错误采用相同方式。知道如何修复时务必说明修复方法，例如 "Enter an email address in the format you@example.com."
6.  **错误链接**：务必仅在字段无效时，将错误消息列在字段的 `aria-describedby` 中（位于任何提示之后），以便屏幕阅读器随字段一起朗读。若在标记中引用，则页面加载时就会朗读隐藏消息。"一致的 ARIA 体验"中的脚本会为每个 `data-error` 属性设为其错误消息 `id` 的字段执行此操作。

## 实施指南

### 用例 1：邮箱验证

强制要求：邮箱字段依赖标准 HTML5 属性。错误消息默认隐藏，仅在浏览器判定用户以无效状态离开字段时才显示。

```html
<form>
  <div class="field">
    <label for="email">Email Address</label>
    <!-- 强制要求：将格式提示放在输入框上方，避免自动完成弹窗在编辑时遮挡 -->
    <span id="email-hint" class="hint">Format: you@example.com</span>
    <!-- 务必：使用 type="email" 和 required 等标准 HTML 验证属性 -->
    <!-- 将 data-error 设为错误消息的 id。aria-invalid 脚本会在字段无效时将该 id 列入 aria-describedby。 -->
    <input
      type="email"
      id="email"
      name="email"
      required
      autocomplete="email"
      aria-describedby="email-hint"
      data-error="email-error"
    >
    <div id="email-error" class="error-msg">
      <span role="img" aria-label="Error">❌</span> Enter an email address in the format you@example.com.
    </div>
  </div>
</form>
```

```css
.hint {
  display: block;
  color: #5f6368;
  font-size: 0.85rem;
  margin-bottom: 0.25rem;
}

.error-msg {
  display: none;
  color: #d93025;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

/*
  务必：仅在用户交互后显示错误样式。
  使用多种指示（边框/背景变化 + 图标/文本），避免仅靠颜色传达状态。
*/
input:user-invalid {
  border-color: #d93025;
  background-color: #fce8e6;
}

/* 务必：使用相邻兄弟选择器显示错误消息 */
input:user-invalid + .error-msg {
  display: block;
}

/* 务必：在 :user-valid 时提供明确的成功指示 */
input:user-valid {
  border-color: #188038;
}
```

### 用例 2：密码复杂度

强制要求：在 `pattern` 属性中使用正则前瞻模式定义复杂度规则。规则列表显示在输入框上方以引导用户，出错时高亮显示。

```html
<form>
  <div class="field">
    <label for="password">New Password</label>
    <!-- 强制要求：将提示和规则放在输入框上方，避免移动端键盘遮挡 -->
    <ul id="password-rules" class="rules-list">
      <li>At least 8 characters</li>
      <li>One uppercase letter</li>
      <li>One lowercase letter</li>
      <li>One number</li>
      <li>One special character</li>
    </ul>
    <!-- 务必：使用 pattern 和 minlength 进行复杂密码验证
         务必：通过 pattern 属性用前瞻匹配所有约束
     -->
    <!-- 将 data-error 设为错误消息的 id。aria-invalid 脚本会在字段无效时将该 id 列入 aria-describedby。 -->
    <input
      type="password"
      id="password"
      autocomplete="new-password"
      required
      pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?=.*[\W_]).{8,}"
      minlength="8"
      aria-describedby="password-rules"
      data-error="password-error"
    >
    <!-- 强制要求：为密码提供独立的错误文本。将规则列表变红并不能说明哪里出错，
         屏幕阅读器也无法传达颜色变化。 -->
    <div id="password-error" class="error-msg">
      <span role="img" aria-label="Error">❌</span> Password doesn't meet all the requirements listed above.
    </div>
  </div>
</form>
```

```css
/* 务必：默认样式保持中性 */
.rules-list { 
  color: #5f6368; 
  margin-bottom: 0.5rem;
}

/* 务必：显示无效状态（交互后）：错误 */
input:user-invalid {
  border-color: #d93025;
  background-color: #fce8e6;
}

/* 务必：使用相邻兄弟选择器显示错误消息 */
input:user-invalid + .error-msg {
  display: block;
}

/* 务必：使用现代 :has() 选择器在显示错误时高亮规则列表 */
.field:has(input:user-invalid) .rules-list {
  color: #d93025;
  font-weight: 600;
}

/* 务必：为 :user-valid 状态添加成功指示 */
input:user-valid {
  border-color: #188038;
}

/* 务必：满足后隐藏规则。visibility 保留其空间：在失焦时折叠列表
   会使提交按钮在 mousedown 与 mouseup 之间移动，导致点击丢失。 */
.field:has(input:user-valid) .rules-list {
  visibility: hidden;
}
```

## 回退与浏览器支持

{{ FEATURE_FALLBACKS("user-pseudos") }}

## 其他注意事项

1.  **无障碍**：
    *   强制要求：使用 `aria-describedby` 将规则列表关联到输入框。
    *   禁止：在输入有效前完全隐藏规则列表；用户需要知道该输入什么！
2.  **Pattern 属性限制**：强制要求：`pattern` 属性执行完整匹配（隐含 `^...$`）。确保密码正则覆盖整个字符串。
3.  **验证严格度**：务必注意浏览器默认的 `type="email"` 验证相当宽松（如 `user@localserver` 可能通过）。若需更严格的验证，可能需要使用更健壮的验证库或自定义验证函数配合 `type="email"`。
4.  **焦点管理**：强制要求：若用户提交表单时存在无效字段，浏览器会自动聚焦第一个无效字段。由于提交尝试算作交互，`:user-invalid` 样式会立即生效。
5. **一致的 ARIA 体验**：{{ FEATURE("user-pseudos", "aria-invalid") }}