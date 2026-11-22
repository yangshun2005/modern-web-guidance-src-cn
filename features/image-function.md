# `image()` 函数

## 回退方案

使用只有一个色标且不带位置的渐变，例如 `linear-gradient(blue)`。

{{ BASELINE_STATUS("gradients", "css.types.gradient.linear-gradient.single_color_stop") }}

为获得更广泛的浏览器支持，可以显式地为该单一色标提供 `0 100%` 位置，例如 `linear-gradient(blue 0 100%)`。