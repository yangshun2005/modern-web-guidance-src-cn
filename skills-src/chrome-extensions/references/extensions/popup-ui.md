# Popup UI

## 配置

```json
{
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon-16.png",
      "48": "icons/icon-48.png",
      "128": "icons/icon-128.png"
    },
    "default_title": "My Extension"
  }
}
```

## 关键约束

- 用户点击 Popup 外部时 Popup 会关闭——不要依赖它保持打开
- 默认最大尺寸：800x600 px。通过 body/html 上的 CSS 设置尺寸
- 所有脚本必须是外部文件（CSP——禁止内联脚本）
- 所有事件监听器必须使用 `addEventListener`（禁止内联处理器）

## Popup HTML 模板

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <style>
    body { width: 350px; min-height: 200px; padding: 16px; font-family: system-ui; }
  </style>
</head>
<body>
  <h1>My Extension</h1>
  <div id="content"></div>
  <script src="popup.js"></script>
</body>
</html>
```

## 持久化

Popup 关闭后状态会丢失。使用 `chrome.storage` 进行持久化：

```js
// Save on change
document.getElementById('input').addEventListener('input', (e) => {
  chrome.storage.local.set({ savedInput: e.target.value });
});

// Restore on open
document.addEventListener('DOMContentLoaded', async () => {
  const { savedInput = '' } = await chrome.storage.local.get('savedInput');
  document.getElementById('input').value = savedInput;
});
```

注意：`localStorage` 在技术上可用于 Popup（它们有持久化的源），但强烈推荐使用 `chrome.storage`，因为它适用于所有扩展上下文并支持同步。

## 与 Service Worker 通信

```js
// From popup
const response = await chrome.runtime.sendMessage({ type: 'GET_STATUS' });

// Long-lived connection
const port = chrome.runtime.connect({ name: 'popup' });
port.postMessage({ type: 'INIT' });
port.onMessage.addListener((msg) => { /* handle */ });
```

## 动态 Popup 与无 Popup

如果希望点击图标执行操作而非显示 Popup，移除 `default_popup` 并使用 `chrome.action.onClicked`：

```js
// In service worker — only fires if NO popup is set
chrome.action.onClicked.addListener((tab) => {
  // Open side panel, inject script, etc.
});
```

可以在 Popup 和无 Popup 之间动态切换：
```js
chrome.action.setPopup({ popup: 'popup/popup.html' }); // Enable popup
chrome.action.setPopup({ popup: '' }); // Disable popup (enables onClicked)
```