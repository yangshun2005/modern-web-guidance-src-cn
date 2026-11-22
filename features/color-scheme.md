# `color-scheme` 属性

## 问题

- Chrome 和 Firefox 对 iframe 遵循 `color-scheme`：它们以正确的配色方案渲染嵌入页面，并调整嵌入页面的 `prefers-color-scheme` 媒体查询，以反映嵌入上下文的 `color-scheme`。Safari 不会，即使在 iframe 内也将 `prefers-color-scheme` 解析为系统设置。
  - **如果你同时控制父页面和 iframe：** 将父页面的配色方案显式传给 iframe——在构建 iframe 时通过 URL 参数（`?theme=dark`）传递，或通过 `postMessage()` 传递（后者还能让你响应运行时变化）。在 iframe 中，根据该信号在 `<html>` 上设置 class（和/或在 `:root` 上设置 `color-scheme`），而不是依赖 `prefers-color-scheme`。
  - **如果你只控制嵌入页面：** 在 Safari 中，无法从 iframe 内部可靠地检测嵌入上下文的 `color-scheme`。在你的嵌入 API 上暴露一个显式的主题参数（例如查询字符串或 `postMessage` 协议），并为嵌入方编写文档。

## 回退方案

`color-scheme` 属性属于**渐进增强**。
不支持它的浏览器会忽略该属性，使用默认的浅色模式 UI。

要在旧版浏览器中适应用户偏好，请使用 `prefers-color-scheme` 媒体查询，在偏好深色模式时提供不同的颜色。

- 应使用媒体查询切换 `:root` 或 `html` 上的自定义属性
- 避免在单个组件上使用媒体查询，除非该组件需要超出颜色范围的非常特定的深色模式定制。

```css
:root {
  /* 为每种模式定义品牌颜色 */
  --color-brand-light: #0056b3;
  --color-brand-dark: #00e5ff;
  --color-brand-text-light: white;
  --color-brand-text-dark: #001f24;
  --color-brand: var(--color-brand-light);
  --color-brand-text: var(--color-brand-text-light);

  /* 必须：为不支持 light-dark 的浏览器提供回退 */
  @media (prefers-color-scheme: dark) {
    --color-brand: var(--color-brand-dark);
    --color-brand-text: var(--color-brand-text-dark);
  }

  /* 在旧版浏览器中被忽略 */
  color-scheme: light dark;
}

button.primary {
	background-color: var(--color-brand);
	color: var(--color-brand-text);
}
```