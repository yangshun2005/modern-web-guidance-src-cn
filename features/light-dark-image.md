# 带图片值的 `light-dark()`

## 回退方案

对于不支持图片 `light-dark()` 的浏览器，用 `image-set()` 作为开关：
通过自定义属性给每个选项一个 `type()`，把未激活的那个设为不支持的 MIME 类型，浏览器就会丢弃该选项。
激活的选项取空值，于是它就是一个不带 `type()` 的普通 `image-set()` 选项。

在 `:root` 上定义一次开关。之后每个使用处只需内联两张图片，就像 `light-dark()` 一样。

```css
:root {
  --light-type: ;
  --dark-type: type("image/do-not-use");

  /* 必须：为不支持 light-dark() 图片的浏览器提供回退 */
  @media (prefers-color-scheme: dark) {
    --light-type: type("image/do-not-use");
    --dark-type: ;
  }
}

.hero {
  background-image: image-set(url(hero-light.png) var(--light-type), url(hero-dark.png) var(--dark-type));

  @supports (background-image: light-dark(url("a"), url("b"))) {
    background-image: light-dark(url(hero-light.png), url(hero-dark.png));
  }
}
```

- 只会下载选中的那张图片。
- 分辨率仍然有效：`image-set(url(a-1x.png) 1x var(--light-type), url(a-2x.png) 2x var(--light-type), url(b-1x.png) 1x var(--dark-type), url(b-2x.png) 2x var(--dark-type))`。
- **不要**给激活的选项用 `type("")`。浏览器会把空字符串视为不支持，从而丢弃所有选项。
- 与 `light-dark()` 不同，这个开关不跟随子树设置的 `color-scheme`。手动切换，或任何设置了 `color-scheme` 的子树，都必须同时翻转 `--light-type` 和 `--dark-type`。

### 替代方案：每张图片一个自定义属性

只有少数几张图片需要变化时更简单，但每张图片都需要自己的媒体查询分支，手动切换时也必须逐个翻转。

```css
:root {
  --hero-image: url(hero-light.png);

  @media (prefers-color-scheme: dark) {
    --hero-image: url(hero-dark.png);
  }

  @supports (background-image: light-dark(url("a"), url("b"))) {
    --hero-image: light-dark(url(hero-light.png), url(hero-dark.png));
  }
}

.hero {
  background-image: var(--hero-image);
}
```