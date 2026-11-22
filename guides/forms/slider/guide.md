---
name: slider
description: 为原生范围滑块设置自定义轨道、滑块、填充和聚焦状态样式，同时保留原生功能和无障碍支持。
web-feature-ids:
  - accent-color
  - input-range
  - appearance
  - progress-function
  - attr
  - dir-pseudo
# later: ::slider-thumb, ::slider-track, ::slider-fill
guides:
  - slider-tooltip
  - brand-consistent-forms
---

# 品牌一致的范围滑块

## 使用原生范围输入

使用带标签的 `<input type="range">`，不要用 `<div>` 元素或自定义指针处理器重建滑块。原生控件保留键盘和指针交互、触摸行为及无障碍支持。根据用例配置 `min`、`max`、`step` 和 `value`。

```html
<label for="range-slider">Volume</label>
<input type="range" id="range-slider" value="70"/>
```

如果值需要格式化标签或单位，将其暴露在普通 DOM 内容中，并在变化时更新 `aria-valuetext`。不要在滑块伪元素上使用 `::before` 或 `::after`，因为表单控件上的生成内容不具备互操作性。如果需要跟随滑块的提示框，用 CSS 锚点定位将其锚定到滑块；关于翻转时保持正确方向的提示框，参见 {{ GUIDE_REF("position-aware-tooltips") }}。

## 选择样式方案

如需简单的颜色自定义，使用 {{ GUIDE_REF("brand-consistent-forms") }} 并保留浏览器的原生渲染：

```css
input[type="range"] {
  accent-color: var(--brand-color);
}
```

`accent-color` 不是完整的滑块样式 API。它对滑块、轨道和填充的效果取决于用户代理。当需要控制轨道、滑块、尺寸、形状或填充时，使用下面的伪元素。

## 设置轨道和滑块样式

用 `appearance: none`、`margin: 0` 和 `border: 0` 重置默认操作系统样式。为统一跨浏览器渲染，将 WebKit 和 Firefox 伪元素声明在单独的规则块中，并用 `margin-block-start: calc((var(--slider-track-size) - var(--slider-thumb-size)) / 2)` 居中 WebKit 滑块（Firefox 会自动居中滑块）。将面向使用者的值保留为输入上的自定义属性，这样主题无需直接针对伪元素。

```css
input[type="range"] {
  --slider-track-size: 0.5rem;
  --slider-thumb-size: 1.25rem;
  --slider-track-radius: 999px;
  --slider-thumb-radius: 50%;
  --slider-track-color: #d9d9d9;
  --slider-fill-color: #06c;
  --slider-thumb-color: var(--slider-fill-color);
  --slider-direction: to right;

  /* reverse gradient direction for right-to-left layouts */
  &:dir(rtl) {
    --slider-direction: to left;
  }

  /* Read min and max from attributes; --control-value is synced by the script below. */
  --attr-min: attr(min type(<number>), 0);
  --attr-max: attr(max type(<number>), 100);
  --control-value: attr(value type(<number>), 50);
  --slider-progress: calc(
    progress(var(--control-value), var(--attr-min), var(--attr-max)) * 100%
  );

  appearance: none;
  inline-size: 100%;
  block-size: 2.5rem; /* Preserve a generous interaction area. */
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  cursor: pointer;
}

/* WebKit/Blink. */
input[type="range"]::-webkit-slider-runnable-track {
  block-size: var(--slider-track-size);
  border: 0;
  border-radius: var(--slider-track-radius);
  background: linear-gradient(
    var(--slider-direction),
    var(--slider-fill-color) var(--slider-progress),
    var(--slider-track-color) var(--slider-progress)
  );
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  inline-size: var(--slider-thumb-size);
  block-size: var(--slider-thumb-size);
  margin-block-start: calc(
    (var(--slider-track-size) - var(--slider-thumb-size)) / 2
  );
  border: 0;
  border-radius: var(--slider-thumb-radius);
  background: var(--slider-thumb-color);
}

/* Firefox. */
input[type="range"]::-moz-range-track {
  block-size: var(--slider-track-size);
  border: 0;
  border-radius: var(--slider-track-radius);
  background: var(--slider-track-color);
}

input[type="range"]::-moz-range-progress {
  block-size: var(--slider-track-size);
  border-radius: var(--slider-track-radius);
  background: var(--slider-fill-color);
}

input[type="range"]::-moz-range-thumb {
  inline-size: var(--slider-thumb-size);
  block-size: var(--slider-thumb-size);
  border: 0;
  border-radius: var(--slider-thumb-radius);
  background: var(--slider-thumb-color);
}
```

确保轨道颜色与背景色的对比度至少为 `3:1`。

移除原生外观后，始终提供可见的键盘聚焦指示器。确保聚焦指示器与背景色或相邻轨道的对比度至少为 `3:1`。不要仅依赖颜色：

```css
input[type="range"]:focus-visible {
  outline: 2px solid var(--slider-fill-color);
  outline-offset: 4px;
}
```

## 设置活动轨道样式

当轨道的填充部分与未填充部分样式不同时，Firefox 通过 `::-moz-range-progress` 暴露它。WebKit/Blink 不提供等效伪元素，因此在 `input` 事件中将实时输入值同步到 `--control-value`：

```js
const slider = document.querySelector('#range-slider');

function syncSliderValue() {
  slider.style.setProperty('--control-value', slider.value);
}

syncSliderValue();
slider.addEventListener('input', syncSliderValue);
```

## 限制

- 不要将 WebKit 和 Firefox 伪选择器合并到单个逗号分隔列表中（例如 `::-webkit-slider-thumb, ::-moz-range-thumb`）；浏览器遇到无法识别的厂商伪元素时会丢弃整条规则。

## 回退

{{ FEATURE_FALLBACKS("accent-color") }}

### `progress()` 和 `attr()` 的回退与浏览器支持

{{ BASELINE_STATUS("progress-function") }}

当 `progress()` 不受支持时，用 `calc()` 计算 `--slider-progress`：

```css
@supports not (opacity: progress(1, 0, 100)) {
  input[type="range"] {
    --slider-progress: calc(
      (var(--control-value) - var(--attr-min)) /
      (var(--attr-max) - var(--attr-min)) * 100%
    );
  }
}
```

{{ BASELINE_STATUS("attr") }}

当类型化 `attr()` 不受支持时，在 JavaScript 中从输入的 `min` 和 `max` 属性同步 `--attr-min` 和 `--attr-max`：

```js
function syncSliderValue() {
  slider.style.setProperty('--attr-min', slider.min || 0);
  slider.style.setProperty('--attr-max', slider.max || 100);
  slider.style.setProperty('--control-value', slider.value);
}
```

{#
## Future standard pseudo-elements

CSS Forms Level 1 specifies standard `::slider-track`, `::slider-thumb`, and
`::slider-fill` pseudo-elements, alongside the `control-value()` function to
query live slider values directly in CSS without JavaScript syncing. When
browser support matures, prefer these native primitives over vendor
pseudo-elements and script-driven CSS custom properties.
#}