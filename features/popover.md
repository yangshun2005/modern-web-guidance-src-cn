# Popover API

## 回退方案

Popover API 主要是**渐进增强**，但其核心行为——顶层提升、轻触关闭和 `popovertarget` 调用——没有纯 CSS 的等价实现。旧版浏览器需要 polyfill，或者如果你不想引入 polyfill，可以手动回退。

**Polyfill：** 要在旧版浏览器中支持 `popover` 属性，按需加载 [`@oddbird/popover-polyfill`](https://github.com/oddbird/popover-polyfill)。**强制要求：** 通过检查 `HTMLElement.prototype` 上的 `popover` 属性进行特性检测，**仅在**缺少原生支持时加载 polyfill——不要无条件加载。

使用打包工具或 import map 时：

```js
// 强制要求：在 HTMLElement.prototype 上检测 'popover'。
if (!Object.hasOwn(HTMLElement.prototype, "popover")) {
  import("@oddbird/popover-polyfill");
}
```

如果不使用包管理器，可从 CDN 导入，例如 `"https://unpkg.com/@oddbird/popover-polyfill@latest/dist/popover.min.js"`。

使用 polyfill 时的注意事项：
- 不要在 polyfill 完全加载前可能运行的代码中使用 `popover` IDL 属性。它不会被镜像为属性，polyfill 不会识别该 popover，且该属性会永久遮蔽 polyfill 添加的访问器。请改用 `setAttribute()`/`removeAttribute()`。
- **样式注意事项：** polyfill 无法定义真正的 `:popover-open` 伪类，因此改用 `.\:popover-open` 类。**强制要求：** 用 `:is()` 或 `:where()` 将两者组合，否则缺少 `:popover-open` 的浏览器会丢弃整条规则：

```css
[popover]:is(:popover-open, .\:popover-open) {
  display: block;
}
```

要防止 popover 在 polyfill 加载前显示，用 CSS 隐藏它们：

```css
@supports not selector(:popover-open) {
  [popover]:not(.\:popover-open) {
    display: none;
  }
}
```