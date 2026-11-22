# 消息传递

## 基本模式

### 单向消息（发后即忘）

```js
// 发送方（popup、内容脚本等）
chrome.runtime.sendMessage({ type: 'LOG', data: 'hello' });

// 接收方（service worker）
chrome.runtime.onMessage.addListener((message, sender) => {
  if (message.type === 'LOG') console.log(message.data);
});
```

### 请求/响应 — IIFE + return true（兼容性最好）

```js
// 发送方
const response = await chrome.runtime.sendMessage({ type: 'GET_DATA' });
console.log(response.data);

// 接收方 — IIFE 保持通道打开，直到调用 sendResponse
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'GET_DATA') {
    (async () => {
      const data = await chrome.storage.local.get('key');
      sendResponse({ data });
    })();
    return true; // 必需 — 告知 Chrome 保持通道打开
  }
});
```

### 请求/响应 — 返回 Promise（Chrome 99+）

现在支持直接从监听器返回 Promise，比 IIFE 模式更简洁：

```js
chrome.runtime.onMessage.addListener((message, sender) => {
  if (message.type === 'GET_DATA') {
    return chrome.storage.local.get('key'); // 返回的 Promise 解析为响应
  }
  // 对此监听器不处理的消息，不返回任何内容（或返回 undefined）
});
```

**注意：** 需要 Chrome 99+，仅在最低 Chrome 版本设为 99 时使用。
**注意：** 不要混用两种风格。如果返回 Promise，就不要再调用 `sendResponse` 或 `return true`。

## 内容脚本 ↔ service worker

```js
// 内容脚本 → service worker
const result = await chrome.runtime.sendMessage({ type: 'FETCH_DATA', url: location.href });

// service worker → 指定标签页的内容脚本
await chrome.tabs.sendMessage(tabId, { type: 'HIGHLIGHT', selector: '.important' });
```

## service worker → 内容脚本（定向）

始终检查标签页是否存在、内容脚本是否已注入：

```js
async function sendToContentScript(tabId, message) {
  try {
    return await chrome.tabs.sendMessage(tabId, message);
  } catch (err) {
    // 内容脚本尚未注入，或标签页已跳转
    console.warn('Could not reach content script:', err.message);
    return null;
  }
}
```

## 长连接（端口）

需要持久通道时使用端口（例如流式数据、DevTools 面板）：

```js
// 发起方（popup 或内容脚本）
const port = chrome.runtime.connect({ name: 'my-channel' });
port.postMessage({ type: 'START' });
port.onMessage.addListener((msg) => console.log('received:', msg));
port.onDisconnect.addListener(() => console.log('disconnected'));

// 接收方（service worker）
chrome.runtime.onConnect.addListener((port) => {
  if (port.name !== 'my-channel') return;
  port.onMessage.addListener((msg) => {
    if (msg.type === 'START') {
      port.postMessage({ status: 'ok' });
    }
  });
});
```

## 常见错误

### 缺少 `return true` 导致响应永远无法到达

```js
// ❌ 错误 — 异步操作完成时通道已关闭
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetchSomething().then(data => sendResponse(data)); // 太晚了
  // 缺少：return true
});

// ✅ 正确
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetchSomething().then(data => sendResponse(data));
  return true;
});
```

### 在内容脚本就绪前向标签页发送消息

内容脚本在页面加载后注入。如果 service worker 在 `tabs.onUpdated` 时立即发送消息，内容脚本可能还没开始监听。使用握手或重试：

```js
// 内容脚本 — 宣告已就绪
chrome.runtime.sendMessage({ type: 'CONTENT_READY' });

// service worker — 等待 CONTENT_READY 后再发送
chrome.runtime.onMessage.addListener((msg, sender) => {
  if (msg.type === 'CONTENT_READY' && sender.tab) {
    chrome.tabs.sendMessage(sender.tab.id, { type: 'INIT_DATA', ... });
  }
});
```

### 多个监听器同时响应

对给定消息类型，只应有一个监听器响应。如果多个监听器都调用 `sendResponse`，只有第一个生效，其余会被静默忽略。