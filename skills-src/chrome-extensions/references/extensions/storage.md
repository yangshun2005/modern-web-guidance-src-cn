# Chrome Storage API

## 存储区域

| 区域 | 持久化 | 同步 | 配额 | 用途 |
|------|----------|-------|-------|---------|
| `chrome.storage.local` | 是 | 否 | 10 MB | 大多数扩展数据 |
| `chrome.storage.sync` | 是 | 是（跨设备） | 总计 100 KB，每项 8 KB | 用户偏好、小数据 |
| `chrome.storage.session` | 直到浏览器关闭 | 否 | 10 MB | 临时状态，在 SW 重启后仍保留 |

所需权限：`"storage"`

## 基本操作

```js
// Set
await chrome.storage.local.set({ key: 'value', count: 42, items: [1,2,3] });

// Get (with defaults)
const { key = 'default', count = 0 } = await chrome.storage.local.get(['key', 'count']);

// Get all
const allData = await chrome.storage.local.get(null);

// Remove
await chrome.storage.local.remove('key');
await chrome.storage.local.remove(['key1', 'key2']);

// Clear all
await chrome.storage.local.clear();
```

## 变更监听器（跨所有上下文生效）

```js
chrome.storage.onChanged.addListener((changes, areaName) => {
  for (const [key, { oldValue, newValue }] of Object.entries(changes)) {
    console.log(`${areaName}.${key}: ${oldValue} → ${newValue}`);
  }
});
```

## storage.sync 配额

使用 sync 时需注意限制：
- `QUOTA_BYTES_PER_ITEM`：每个键值对 8,192 字节
- `MAX_ITEMS`：512 项
- `QUOTA_BYTES`：总计 102,400 字节
- `MAX_WRITE_OPERATIONS_PER_HOUR`：1,800
- `MAX_WRITE_OPERATIONS_PER_MINUTE`：120

对于大数据，拆分为多个键或使用 `chrome.storage.local`。

## storage.session 说明

- 仅在 MV3 中可用
- 浏览器关闭时清除（不仅仅是 SW 终止时）
- 可从 service worker、popup、侧边栏等访问
- 适用于：认证令牌、临时缓存、进行中的操作

## 为什么不用 localStorage？

`localStorage` 在 popup 和扩展页面中可用，但在 service worker 中不可用。
`chrome.storage` 在所有地方都可用，并支持跨上下文变更监听器。
始终优先使用 `chrome.storage`。