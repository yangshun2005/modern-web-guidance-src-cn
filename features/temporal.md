# Temporal

## 回退方案

对于尚不支持原生 `Temporal` API 的浏览器，使用特性检测并动态加载 `temporal-polyfill`（约 15 KB min+gz）。

注意：导入 `temporal-polyfill` 不会自动将 `Temporal` 对象挂到全局作用域。如果你的代码依赖 `globalThis.Temporal`，需显式赋值。

```javascript
// 检查是否原生支持 Temporal
(async () => {
  if (typeof Temporal === 'undefined') {
    // 按需加载 polyfill
    const module = await import('https://esm.sh/temporal-polyfill');
    globalThis.Temporal = module.Temporal;
    // 如需扩展 Date.prototype
    Date.prototype.toTemporalInstant = module.toTemporalInstant;
  }
  initializeApp();
})();
```