# 锚点定位

## Popover 回退方案

### 锚点定位的回退方案与浏览器支持

{{ BASELINE_STATUS("anchor-positioning") }}

在不支持 CSS 锚点定位时，`[popover]` 浮层等绑定元素会回退到用户代理的默认位置（popover 居中于视口）。根据你的用例是否需要空间绑定，可选择按需加载 polyfill，或用 `@supports` 限定定位规则。

#### 方案 1：Polyfill（`@oddbird/css-anchor-positioning`）

当定位元素在不支持的浏览器中必须与触发器保持空间绑定时，按需加载 `@oddbird/css-anchor-positioning` 包。

**强制要求：** 通过 `CSS.supports("anchor-name: --a")`（或 `"anchorName" in CSSStyleDeclaration.prototype`）检测特性支持，**仅在**缺少原生支持时加载 polyfill——**不要**无条件加载。

使用打包工具或 import map 时：

```js
// 强制要求：加载 polyfill 前先检测锚点定位支持。
if (!CSS.supports("anchor-name: --a")) {
  import("@oddbird/css-anchor-positioning");
}
```

不使用打包工具时，在 `<script type="module">` 中从 CDN 导入：

```html
<!-- 强制要求：按需安装锚点定位 polyfill -->
<script type="module">
  if (!CSS.supports("anchor-name: --a")) {
    import("https://unpkg.com/@oddbird/css-anchor-positioning");
  }
</script>
```

**Polyfill 注意事项：**
- **不支持隐式锚点：** polyfill 不支持隐式锚点关联（如由 `popovertarget`、`commandfor`、`interestfor` 或 `showPopover({ source })` 创建的关联）。你**必须**在每个触发器上设置唯一的显式 `anchor-name`，并在定位元素上设置匹配的 `position-anchor`。
- **`position-area` 不能与 `position-try-fallbacks` 同用：** polyfill 不支持 `position-area` 搭配 `position-try-fallbacks`。对需要回退的元素使用 polyfill 时，改用 inset 属性（`top`、`left`、`right`、`bottom`）上的 `anchor()` 函数配合 `inset: auto` 来定位，而非 `position-area`。
- **不要将 polyfill 规则包在 `@supports` 中：** 不支持原生锚点定位的浏览器会将 `@supports (position-area: ...)` 和 `@supports (anchor-name: --a)` 求值为 `false`，在 polyfill 处理前就丢弃该块。

```css
.trigger {
  /* 强制要求：每个触发器与定位元素对必须共享唯一的锚点名称。 */
  anchor-name: --my-trigger;
}

[popover].anchored-overlay {
  position-anchor: --my-trigger;
  /* 强制要求：对需要 position-try-fallbacks 的元素使用 polyfill 时，用 inset 属性上的 anchor() 而非 position-area。 */
  inset: auto;
  top: anchor(bottom);
  left: anchor(left);
  margin: 0;
}
```

#### 方案 2：用 `@supports` 渐进增强（不使用 polyfill）

如果不提供 polyfill，**不要**在依赖 `position-area` 或 `anchor()` 的 `[popover]` 元素上无条件设置 `inset: auto` 或 `margin: 0`。在不支持锚点定位的浏览器中，`inset: auto` 会覆盖用户代理的居中（`inset: 0; margin: auto`），而 `position-area` 被忽略，导致 popover 停在 `(0, 0)` 或与触发器重叠。

应针对所用的具体定位特性，将 `inset: auto` 和 `margin: 0` 限定在 `@supports` 内（例如 `@supports (position-area: block-end)` 或 `@supports (top: anchor(bottom))`），让不支持的浏览器保留默认的居中 popover 位置；或提供匹配的 `@supports not (...)` 回退布局（如固定底部面板）：

```css
@supports (position-area: block-end) {
  [popover].anchored-overlay {
    inset: auto;
    margin: 0;
    position-area: block-end span-inline-end;
    position-try-fallbacks: flip-block, flip-inline, flip-block flip-inline;
  }
}

@supports not (position-area: block-end) {
  /* 可选：用固定底部栏覆盖默认的 popover 居中 */
  [popover].anchored-overlay {
    position: fixed;
    inset: auto 0 0 0;
    width: auto;
    margin: 0;
  }
}
```