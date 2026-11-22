# CSS `transition-behavior`

## 回退方案

{{ BASELINE_STATUS("transition-behavior", "css.properties.transition-behavior.transitionable_display") }}

Firefox 129+ 能解析 `transition-behavior: allow-discrete`（`CSS.supports('transition-behavior', 'allow-discrete')` 返回 `true`），但并不会真正对 `display` 属性执行过渡（Firefox bug 1882408），导致元素在退出时立即消失。

要可靠检测离散 `display` 过渡支持，应探测临时元素过渡到 `none` 时其计算后的 `display` 是否仍可见，而不是仅依赖 `CSS.supports('transition-behavior', 'allow-discrete')`：

```javascript
let supportsDisplayTransition;
function canTransitionDisplay() {
  if (supportsDisplayTransition !== undefined) return supportsDisplayTransition;
  if (!window.CSS?.supports?.('transition-behavior', 'allow-discrete') || !document.body) {
    return false;
  }
  const probe = document.createElement('div');
  // 此处故意使用简写：无法解析 allow-discrete 的浏览器会丢弃整条声明，
  // 于是 display: none 立即生效，探测正确返回 false。!important 用于防止
  // 全局 reduced-motion 重置（如 `* { transition: none !important }`）。
  probe.style.cssText = 'transition: display 1s allow-discrete !important; display: block;';
  document.body.appendChild(probe);
  getComputedStyle(probe).display;
  probe.style.display = 'none';
  supportsDisplayTransition = getComputedStyle(probe).display === 'block';
  probe.remove();
  return supportsDisplayTransition;
}
```