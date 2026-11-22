# Focusgroup

## 问题

- 将 `focusgroup` 放在 `[popover]` 内部的元素上，而不是放在 `[popover]` 本身。

{# 维护者注：https://issues.chromium.org/issues/564673920（2026-09 修复），https://github.com/whatwg/html/issues/11641 #}

## 回退方案

仅在缺少原生支持时加载 `@microsoft/focusgroup-polyfill`：

```js
if (!(Object.hasOwn(HTMLElement.prototype, 'focusGroup'))) {
  const { polyfill } = await import('@microsoft/focusgroup-polyfill');
  polyfill();
}
```

- 使用 `focusgroup` 和 `focusgroupstart` 属性，不要使用 `focusGroup` 属性，polyfill 不会添加后者。
- polyfill 不会感知之后隐藏或显示的元素：在元素隐藏期间为其添加 `focusgroup="none"`。

{# 维护者注：限制来自 v1.6.0 README 和 Issue（https://github.com/microsoft/polyfills/tree/main/packages/focusgroup）：#74 已关闭 popover 中的元素，#72 祖先移除时泄漏。menu、toolbar 和 scroll-snap-realtime-feedback 仍自带回退方案；需单独迁移。 #}