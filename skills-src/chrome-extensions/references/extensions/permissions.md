# 权限

## `permissions` 与 `host_permissions`

这是两个独立的 manifest 键——不要混为一谈：

```json
{
  "permissions": ["tabs", "scripting", "storage"],
  "host_permissions": ["https://api.example.com/*"]
}
```

- `permissions` 授予对 chrome.* API 的访问权限（`tabs`、`storage`、`scripting`、`desktopCapture` 等）
- `host_permissions` 授予对特定源的访问权限，用于 `fetch`、`chrome.scripting.executeScript` 以及跨源读取标签页 URL

将 `host_permissions` 限定到具体域名，而非 `<all_urls>`，除非扩展确实需要在所有网站上运行——宽泛的 host 权限会招致 Chrome Web Store 更严格的审核。

## `tab.url` 需要 `tabs` 权限

没有该权限时，`tab.url` 和 `tab.title` 会静默返回 `undefined`——不抛错误。

```js
// manifest.json — 若你在任何地方读取 tab.url 或 tab.title，则必须添加：
{ "permissions": ["tabs"] }
```

完整的 tabs/windows API 见 `references/extensions/tab-management.md`。

## `activeTab` 仅在直接用户手势下生效——侧边栏中无效

`activeTab` 仅在以下触发条件下授予对当前标签页的临时访问权限：
- 点击扩展操作图标
- 上下文菜单项（包括 `"tab"` 上下文）
- 来自 `commands` API 的键盘快捷键
- 接受 omnibox 建议

在侧边栏中点击按钮、点击稍后才打开的弹窗按钮，或任何程序化触发，均**不会**授予访问权限。

```js
// ❌ 无效——activeTab 在侧边栏按钮点击中不生效
document.getElementById('summarize').addEventListener('click', async () => {
  const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
  await chrome.scripting.executeScript({ target: { tabId: tab.id }, func: () => document.body.innerText });
});

// ✅ 修复——改用 "tabs" 权限 + 具体的 host_permissions
// manifest.json: { "permissions": ["tabs", "scripting"], "host_permissions": ["<all_urls>"] }
```

侧边栏相关的专门说明见 `references/extensions/side-panel.md`。

## `chrome.permissions.request()` 需要用户手势——手势可跨越一次 `sendMessage` 传递，但之后的 `await` 会使其失效

`chrome.permissions.request()`（用于可选/动态权限）仅在用户手势（点击、按键等）内生效。来自 UI 上下文（侧边栏、弹窗）的手势**确实**能通过 `chrome.runtime.sendMessage` 传递到 service worker 的 `onMessage` 监听器——但仅对那一个同步轮次有效。如果监听器在调用 `chrome.permissions.request()` 之前 `await` 了任何东西（定时器、存储读取、另一次消息往返），手势就会失效，调用会抛出 `"This function must be called during a user gesture, such as an onclick handler"`。

```js
// side-panel.js
button.addEventListener('click', () => {
  chrome.runtime.sendMessage({ type: 'REQUEST_PERMISSION' }, (granted) => { ... });
});

// service-worker.js

// ❌ 无效——chrome.permissions.request() 之前的 await 会消耗掉手势窗口
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'REQUEST_PERMISSION') {
    (async () => {
      await chrome.storage.local.get('someSetting'); // 手势在此之后失效
      const granted = await chrome.permissions.request({ permissions: ['downloads'] });
      sendResponse(granted);
    })();
    return true;
  }
});

// ✅ 正确——在监听器中第一件事就调用 chrome.permissions.request()，
// 之前不 await 任何东西
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === 'REQUEST_PERMISSION') {
    chrome.permissions.request({ permissions: ['downloads'] }).then(sendResponse);
    return true; // 可以 await *结果*——调用本身已同步发生
  }
});
```

**经验法则：** 在接收点击触发消息的消息监听器中，将 `chrome.permissions.request()`（或任何依赖用户手势的调用，如 `activeTab`）作为第一条语句——放在任何其他 `await` 之前。之后 await *返回的* promise 没问题；在调用*之前* await 任何东西才是问题所在。

## 检查与移除权限

```js
// 检查某个可选权限当前是否已授予
const hasIt = await chrome.permissions.contains({ permissions: ['downloads'] });

// 移除之前授予的可选权限
await chrome.permissions.remove({ permissions: ['downloads'] });
```

在 manifest 中声明可选权限，使其可在运行时被请求：

```json
{ "optional_permissions": ["downloads"] }
```