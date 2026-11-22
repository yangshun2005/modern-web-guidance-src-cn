# DevTools 面板

## 配置

```json
{
  "devtools_page": "devtools/devtools.html"
}
```

devtools 页面仅在 DevTools 打开时运行。它不可见——其职责是创建面板。

## 创建面板

`devtools/devtools.html`：
```html
<!DOCTYPE html>
<html>
<body>
  <script src="devtools.js"></script>
</body>
</html>
```

`devtools/devtools.js`：
```js
chrome.devtools.panels.create(
  'My Panel',                    // 显示在 DevTools 标签页中的标题
  'icons/icon-16.png',           // 图标（可选，可为空字符串）
  'devtools/panel/panel.html',   // 面板内容页——相对于扩展根目录
  (panel) => {
    // panel.onShown.addListener((window) => { ... });
    // panel.onHidden.addListener(() => { ... });
  }
);
```

**关键：面板路径相对于扩展根目录**，而非相对于 devtools.js 文件。这是 DevTools 扩展最常见的 bug。

```js
// ❌ 错误——解析为 <ext-root>/panel/panel.html（文件不存在）
chrome.devtools.panels.create("My Panel", "", "panel/panel.html");

// ✅ 正确——解析为 <ext-root>/devtools/panel/panel.html
chrome.devtools.panels.create("My Panel", "", "devtools/panel/panel.html");
```

## 面板内容

`devtools/panel/panel.html` 是普通扩展页面，拥有完整的 chrome.* API 访问权限。

## 访问 DevTools API

仅在 devtools 页面和面板中可用：

```js
// 获取被检查窗口的标签页 ID
const tabId = chrome.devtools.inspectedWindow.tabId;

// 在被检查页面中执行 JS
chrome.devtools.inspectedWindow.eval('document.title', (result, isException) => {
  console.log('Page title:', result);
});

// 监控网络请求
chrome.devtools.network.onRequestFinished.addListener((request) => {
  // request.request.url、request.response.status 等
  // HAR 条目格式
});

// 获取所有已捕获的请求
chrome.devtools.network.getHAR((harLog) => {
  harLog.entries.forEach((entry) => { /* 处理 */ });
});
```

## 通信架构

DevTools 页面/面板并非在所有情况下都能通过 `chrome.runtime.sendMessage` 直接与 service worker 通信。请使用连接模式：

```js
// 在面板 JS 中——连接 service worker
const port = chrome.runtime.connect({ name: 'devtools-panel' });
port.postMessage({ type: 'INIT', tabId: chrome.devtools.inspectedWindow.tabId });
port.onMessage.addListener((msg) => { /* 处理 */ });

// 在 service worker 中
chrome.runtime.onConnect.addListener((port) => {
  if (port.name === 'devtools-panel') {
    port.onMessage.addListener((msg) => { /* 处理 */ });
  }
});
```

## 重要说明

- DevTools 页面按 DevTools 窗口存在（每个被检查标签页一个）
- DevTools 关闭时它们会被销毁
- `chrome.devtools.*` API 仅在 devtools 页面上下文中可用，在 service worker 中不可用
- 面板可通过 `chrome.devtools.inspectedWindow.eval()` 向被检查页面注入脚本