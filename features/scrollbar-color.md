# scrollbar-color

## 用法

使用 `scrollbar-color` 时，用 CSS 变量分别保存滑块和轨道颜色，便于阅读和维护（尤其在使用回退方案时）。

```css
.scroller {
  --scrollbar-thumb: var(--color-neutral-70);
  --scrollbar-track: var(--color-neutral-90);

  scrollbar-color: var(--scrollbar-thumb) var(--scrollbar-track);
}
```

## 问题

- 不要对 `scrollbar-color` 做动画或过渡。一个 [WebKit bug](https://bugs.webkit.org/show_bug.cgi?id=311752) 会导致每次 `scrollbar-color` 变化时滚动条闪烁。
- 计算值不为 `auto` 的 `scrollbar-width`（`thin` 或 `none`），或计算值不为 `auto` 的 `scrollbar-color`（包括继承来的 `scrollbar-color`），都会使该元素上的旧式 `::-webkit-scrollbar` 伪元素失效。
- 使用覆盖式滚动条（如 macOS 默认）时，`scrollbar-color` 只给滚动中的滑块上色，轨道可能保持透明。要确保滑块颜色与轨道颜色和可滚动容器的背景都有足够对比度，而不是依赖轨道来衬托滑块。

## 回退方案

此特性属于渐进增强，并不总是需要回退方案。

如果样式很重要，且用户的 Baseline 目标是"Baseline Widely Available"或更早，你应当加入非标准的 `::-webkit-scrollbar` 伪元素作为回退。

将旧式回退放在基础规则之后、`@supports not (scrollbar-color: auto)` 块内，以避免在原生同时支持两者的浏览器中，标准属性与旧式 WebKit 选择器冲突。如果你还在可滚动容器上设置了 `scrollbar-width: thin`，请在同一选择器的 `@supports not (scrollbar-color: auto)` 块内重置 `scrollbar-width: auto`，这样支持 `scrollbar-width` 但不支持 `scrollbar-color` 的浏览器 {# 如 Safari 18.2–26.1 #} 就不会抑制 `::-webkit-scrollbar` 样式。

如果你用自定义属性定义颜色，这些属性会自动级联到旧式 WebKit 选择器，无需重复定义。

```css
/* Legacy fallback for WebKit/Blink browsers */
@supports not (scrollbar-color: auto) {
  .scroller {
    /* Reset scrollbar-width: thin (if set) so it does not disable
       ::-webkit-scrollbar in browsers without scrollbar-color */
    scrollbar-width: auto;
  }

  .scroller::-webkit-scrollbar {
    /* Must define a base size (e.g., 8px for thin or 12px for default) for custom colors to render in WebKit */
    width: 12px;
    height: 12px;
  }

  .scroller::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
  }

  .scroller::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
  }
}
```