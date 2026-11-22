# `light-dark()` 函数

## 回退方案

对于支持 `color-scheme` 但尚不支持 `light-dark()` 的浏览器，应先将颜色的浅色和深色版本定义为自定义属性，再使用 `prefers-color-scheme` 媒体查询为对应模式设置颜色，如下例所示：

```css
:root {
  /* 为每种模式定义浏览器 UI 强调色 */
  --brand-accent-light: #0056b3;
  --brand-accent-dark: #00e5ff;
  --accent-color: var(--brand-accent-light);

  /* 必需：为不支持 light-dark 的浏览器提供回退 */
  @media (prefers-color-scheme: dark) {
    --accent-color: var(--brand-accent-dark);
  }

  /* 可选：使用 light-dark() 更精细地控制内置 UI 颜色 */
  @supports (color: light-dark(white, black)) {
    --accent-color: light-dark(var(--brand-accent-light), var(--brand-accent-dark));
  }

  /* 必需：让原生 UI 自动适应用户系统偏好 */
  color-scheme: light dark;

  /* 继承颜色属性示例 */
  accent-color: var(--accent-color);
}

pre, code {
  color-scheme: dark;

  /* **必需**：任何继承的颜色属性都必须重新设置，即使使用相同的设计令牌 */
  accent-color: var(--accent-color);
}
```