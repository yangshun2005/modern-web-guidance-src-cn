# `ariaNotify()` 方法

## 回退方案

{{ BASELINE_STATUS("arianotify") }}

使用 ARIA 实时区域，确保容器在页面加载时就存在，然后在需要时将消息添加到实时区域容器中。

```html
<!-- 预先填充的隐藏实时区域容器 -->
<div id="notification" aria-live="assertive"></div>
```

该内容会存在于 DOM 中，屏幕阅读器在播报消息后仍可读取它。因此，为了给该区域提供上下文，当实时区域有内容时，在容器前紧邻位置添加文本，为播报提供上下文。这样，当用户在阅读页面时识别到该内容，也能理解其用途。

```html
<p class="inclusively-hidden">屏幕阅读器播报</p>
<!-- 预先填充的隐藏实时区域容器 -->
<div id="notification" aria-live="assertive">
  注入到现有容器中的动态播报。
</div>
```