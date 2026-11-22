# 内容脚本与 DOM 操作

## 两种注入方式

### 1. 静态注入（manifest 声明）
```json
{
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content/content.js"],
    "css": ["content/content.css"],
    "run_at": "document_idle"
  }]
}
```

### 2. 编程式注入（从 service worker 或 popup）
```js
// 需要 "scripting" 权限和主机访问权限
chrome.scripting.executeScript({
  target: { tabId: tabId },
  files: ['content/content.js']
});

// 或直接注入函数
chrome.scripting.executeScript({
  target: { tabId: tabId },
  func: (param) => {
    document.body.style.backgroundColor = param;
  },
  args: ['yellow']
});
```

点击时注入使用 `activeTab` 权限（无需 host_permissions）：
```json
{
  "permissions": ["activeTab", "scripting"]
}
```

## 隔离环境

内容脚本运行在隔离环境中：
- 与页面共享 DOM，但不共享 JavaScript 变量
- 可访问 chrome.runtime 消息 API
- 页面的 CSP 不限制内容脚本代码
- `window` 指向内容脚本的隔离环境

## 内容脚本的消息传递

```js
// content.js → service worker
chrome.runtime.sendMessage({ type: 'DATA', payload: data }, (response) => {
  console.log('Got response:', response);
});

// service worker → 指定标签页中的内容脚本
chrome.tabs.sendMessage(tabId, { type: 'UPDATE', data: newData });

// content.js：监听消息
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'GET_CONTENT') {
    const text = document.body.innerText;
    sendResponse({ text });
  }
  return true; // 保持通道打开以支持异步 sendResponse
});
```

## DOM 操作最佳实践

- **修改大量 DOM 元素时避免阻塞主线程**。用 `requestAnimationFrame` 批量处理视觉更新，用 `scheduler.yield()` 拆分长任务：

```js
// ❌ 错误：处理数百个元素时阻塞主线程
const emails = document.body.innerText.match(/[\w.+-]+@[\w-]+\.[\w.]+/g);
emails.forEach(email => {
  // ... 查找并高亮每个邮箱（可能导致页面卡死）
});

// ✅ 正确：使用 requestAnimationFrame 分批处理
async function highlightEmails(elements) {
  const BATCH_SIZE = 20;
  for (let i = 0; i < elements.length; i += BATCH_SIZE) {
    const batch = elements.slice(i, i + BATCH_SIZE);
    await new Promise(resolve => requestAnimationFrame(() => {
      batch.forEach(el => el.style.backgroundColor = 'yellow');
      resolve();
    }));
    // 批次之间让出主线程
    if (typeof scheduler !== 'undefined' && scheduler.yield) {
      await scheduler.yield();
    }
  }
}
```

- 动态页面（SPA、无限滚动）使用 `MutationObserver`
- 为 CSS 类名加命名空间避免冲突（如 `myext-highlight`）
- 注入页面的复杂 UI 使用 Shadow DOM
- 移除时清理：`chrome.runtime.onMessage` 监听器会一直存在，直到内容脚本上下文被销毁
- 在 DOM 中查找文本时，用 `TreeWalker` 或 `document.createNodeIterator` 代替对 `innerHTML` 用正则——更可靠，且不会破坏事件监听器

## `run_at` 时机

| 值 | 时机 |
|-------|------|
| `document_start` | DOM 构建之前（适合拦截） |
| `document_idle` | DOM 就绪后、所有资源加载前（默认，推荐） |
| `document_end` | DOM 完成后、图片/子框架加载前 |