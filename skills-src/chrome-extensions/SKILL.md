---
name: chrome-extensions
description: >
  使用 Manifest V3 最佳实践构建并发布 Chrome 扩展。当用户要求创建、修改、调试或理解 Chrome 浏览器扩展、附加组件，或任何涉及 Chrome Extensions API 的内容时，使用本技能。触发词包括：'Chrome extension'、'browser extension'、'manifest.json'、'content script'、'service worker'（浏览器语境）、'popup'（浏览器扩展语境）、'side panel'、'chrome.* API'、'declarativeNetRequest'、'omnibox'、'context menu'（扩展语境）、'userScripts'、'user script'、'script manager'，或任何构建与 Chrome 浏览器 UI 集成功能的请求。发布到 Chrome Web Store 时同样触发：'publish extension'、为发布准备扩展、应对审核拒绝、撰写权限理由，或起草隐私政策。
---

# Chrome 扩展

使用 Manifest V3 构建生产级 Chrome 扩展，并发布到 Chrome Web Store。

## 第 1 部分 — 构建扩展

### 强制规则

以下规则针对扩展损坏的最常见原因。违反任何一条都会导致构建不可用。

#### 1. 图标：只引用你创建的文件——或完全省略图标

```
❌ 错误——引用不存在的文件，或所有尺寸复用同一个文件：
   "icons": { "16": "icon.png", "48": "icon.png", "128": "icon.png" }

✅ 正确——每个尺寸是独立文件，像素尺寸正确：
   "icons": { "16": "icons/icon-16.png", "48": "icons/icon-48.png", "128": "icons/icon-128.png" }
   （其中 icon-16.png 为 16×16px，icon-48.png 为 48×48px，icon-128.png 为 128×128px）

✅ 同样正确——无法生成真实 PNG 文件时，从 manifest 中省略图标：
   （直接删除 "icons" 和 "default_icon" 字段——Chrome 会使用默认图标）
```

**如果包含图标引用，就必须创建实际的图片文件。** 用脚本生成（见 `references/extensions/icons.md`），或干脆不写。绝不要引用不存在的文件。

#### 2. 侧边栏：必须提供打开它的方式

定义 `"side_panel": {"default_path": "..."}` 并不会让它可打开。需要添加触发器：

```js
// 在 service-worker.js 中——点击扩展图标时打开侧边栏
// 重要：chrome.action.onClicked 仅在未设置 default_popup 时触发
chrome.action.onClicked.addListener(async (tab) => {
  await chrome.sidePanel.open({ windowId: tab.windowId });
});
```

如果扩展同时有 popup 和侧边栏，在 popup 中添加一个按钮调用 `chrome.sidePanel.open()`。或者使用 `chrome.sidePanel.setPanelBehavior({ openPanelOnActionClick: true })`——但属性名是 `openPanelOnActionClick`，不是 `openPanelOnActionIconClick`；带 "Icon" 的变体会同步抛出 TypeError，静默中止 service worker。使用 `setPanelBehavior` 时不要同时定义 `default_popup`。见 `references/extensions/side-panel.md`。

#### 3. 代码执行：仅限沙箱 iframe

扩展 CSP 会在所有扩展页面中阻止 `eval()`、`new Function()` 和内联 `<script>`。

```js
// ❌ 错误——直接访问 iframe DOM 会抛出 SecurityError
iframe.contentDocument.write(html);

// ❌ 错误——在扩展页面中 eval
eval(userCode); // CSP 会阻止

// ✅ 方案 A：manifest 中声明沙箱 + postMessage
// manifest.json: { "sandbox": { "pages": ["sandbox.html"] } }
iframe.contentWindow.postMessage({ html, css, js }, '*');
// sandbox.html 接收并执行：
window.addEventListener('message', (e) => { eval(e.data.js); /* 沙箱中允许 */ });

// ✅ 方案 B：Blob URL（创建独立源，绕过扩展 CSP）
iframe.src = URL.createObjectURL(new Blob([doc], { type: 'text/html' }));

// ✅ 方案 C：srcdoc
iframe.srcdoc = `<style>${css}</style>${html}<script>${js}<\/script>`;
```

完整细节见 `references/extensions/csp-sandbox.md`。

#### 4. `tab.url` 需要 `tabs` 权限

没有该权限时，`tab.url` 会静默返回 `undefined`——不抛错。见 `references/extensions/permissions.md`。

#### 5. 始终使用 async/await——绝不用 `.then()` 链

```js
// ❌ 差
chrome.tabs.query({active: true, currentWindow: true}).then(tabs => {
  chrome.scripting.executeScript({target: {tabId: tabs[0].id}, files: ['content.js']}).then(() => {});
});

// ✅ 好
const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
await chrome.scripting.executeScript({ target: { tabId: tab.id }, files: ['content.js'] });
```

对于执行异步工作的 `runtime.onMessage` 监听器：

```js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  (async () => {
    const data = await chrome.storage.local.get('key');
    sendResponse({ data });
  })();
  return true; // 保持通道打开
});
```

#### 6. 内容脚本：不要阻塞主线程

修改大量 DOM 元素时，用 `requestAnimationFrame` 分批处理，并在批次之间让出：

```js
async function highlightAll(elements) {
  const BATCH = 20;
  for (let i = 0; i < elements.length; i += BATCH) {
    await new Promise(r => requestAnimationFrame(() => {
      elements.slice(i, i + BATCH).forEach(el => el.style.backgroundColor = 'yellow');
      r();
    }));
    if (globalThis.scheduler?.yield) await scheduler.yield();
  }
}
```

见 `references/extensions/content-scripts.md`。

#### 7. Service worker 是临时的——绝不要在变量中存储状态

```js
// ❌ 错误——SW 终止后状态丢失（约 30 秒无活动）
let count = 0;
chrome.tabs.onUpdated.addListener(() => { count++; });

// ✅ 正确——持久化到 chrome.storage，每次事件时读取
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo) => {
  if (changeInfo.status !== 'complete') return;
  const { count = 0 } = await chrome.storage.local.get('count');
  await chrome.storage.local.set({ count: count + 1 });
  await chrome.action.setBadgeText({ text: String(count + 1) });
});
```

用 `chrome.alarms` 代替 `setTimeout`/`setInterval`。见 `references/extensions/service-worker.md`。

#### 8. chrome.identity：扩展 ID 在开发和生产环境不同

使用 Google 登录时，OAuth client_id 绑定到特定扩展 ID。未打包开发与 Chrome Web Store 的 ID 不同。

开发期间要稳定 ID，可在 manifest.json 中添加 `"key"` 字段：
1. 打包一次扩展（chrome://extensions → 打包）
2. 从 .crx 中提取公钥
3. 将 `"key": "MIIBIjANBgkqh..."` 添加到 manifest.json

务必注明："发布到 Chrome Web Store 后，用商店分配的扩展 ID 更新 OAuth 客户端。"见 `references/extensions/auth-identity.md`。

#### 9. 上下文菜单：操作后给用户反馈

上下文菜单项执行操作（保存、复制等）后，要向用户确认。使用通知、徽章闪烁或注入 toast——不要让操作静默发生。完整 toast 实现见 `references/extensions/context-menus.md`。

#### 10. Prompt API：在 service worker、popup 和侧边栏中可用

`LanguageModel` API 在所有扩展上下文中可用——service worker、popup 和侧边栏——无需额外 manifest 权限。扩展还能使用 `LanguageModel.params()`，这在 Web 上不可用：

```js
const params = await LanguageModel.params();
// { defaultTopK: 3, maxTopK: 128, defaultTemperature: 1, maxTemperature: 2 }
```

通用 Prompt API 模式（可用性检查、会话创建、流式传输）请使用 `modern-web-guidance` 技能。扩展特定的接线示例见 `references/extensions/prompt-api.md`。

#### 11. `chrome.action` API 需要 manifest 中有 `action`

使用 `chrome.action.setBadgeText`、`chrome.action.setIcon` 或 `chrome.action.onClicked` 需要 manifest.json 中有 `"action"` 键——即使为空。没有它，`chrome.action` 为 `undefined`。

```js
// ❌ 错误——manifest 没有 "action" 键
await chrome.action.setBadgeText({ text: '5' });
// TypeError: Cannot read properties of undefined (reading 'setBadgeText')

// ✅ 修复——在 manifest.json 中添加 "action"（至少为空对象）
{ "action": {} }
// 或带 popup：
{ "action": { "default_popup": "popup/popup.html" } }
```

#### 12. `activeTab` 仅对直接用户手势有效——侧边栏中无效

`activeTab` 仅在直接用户手势（点击 action 图标、上下文菜单项、键盘快捷键、omnibox 建议）时授予当前标签页临时访问权限——侧边栏或 popup 内的按钮点击无效。改用 `tabs` + `host_permissions`。见 `references/extensions/permissions.md` 和 `references/extensions/side-panel.md`。

#### 13. DevTools 面板 URL 相对于扩展根目录

创建 DevTools 面板时，面板 HTML 路径相对于**扩展根目录**，而非调用 `chrome.devtools.panels.create()` 的 devtools 页面。

```js
// ❌ 错误——路径相对于 devtools/ 目录
chrome.devtools.panels.create("My Panel", "", "panel/panel.html");

// ✅ 正确——从扩展根目录的完整路径
chrome.devtools.panels.create("My Panel", "", "devtools/panel/panel.html");
```

见 `references/extensions/devtools.md`。

#### 14. Offscreen 文档无法访问大多数 chrome.* API

Offscreen 文档（`chrome.offscreen`）**受到严格限制**。大多数 `chrome.*` API 不可用，包括 `chrome.downloads`、`chrome.tabs`、`chrome.action` 等。

```js
// ❌ 错误——offscreen 文档中 chrome.downloads 为 undefined
chrome.downloads.download({ url, filename: 'recording.webm' }); // TypeError

// ❌ 错误——offscreen 文档中 chrome.action 为 undefined
chrome.action.setBadgeText({ text: 'REC' }); // TypeError
```

**Offscreen 文档中唯一可用的 API：**
- `chrome.runtime.sendMessage` / `chrome.runtime.onMessage`
- `chrome.runtime.getURL`
- 标准 Web API（DOM、fetch、MediaRecorder、Canvas、Web Audio 等）

**经验法则：** Offscreen 文档做 Web API 工作（录制、解析、音频）。Service worker 做所有 chrome.* API 工作（下载、徽章更新、通知）。用 `chrome.runtime.sendMessage` 在两者之间桥接。见 `references/extensions/message-passing.md`。

#### 15. 通知和徽章图标必须引用真实图片文件

`chrome.notifications.create()` 需要有效的 `iconUrl` 指向实际图片文件。文件不存在或路径错误时，调用失败并报 `"Unable to download all specified images."`

```js
// ❌ 错误——图标文件不存在
chrome.notifications.create('reminder', {
  type: 'basic',
  iconUrl: 'icons/icon-128.png', // 扩展中没有此文件！
  title: 'Reminder',
  message: 'Time is up!'
});

// ✅ 运行时通过 OffscreenCanvas 生成 data URL——无需文件。
// 可复用实现见 `references/extensions/icons.md`。
const iconUrl = await getIconDataUrl();
chrome.notifications.create('reminder', { type: 'basic', iconUrl, title: 'Reminder', message: 'Time is up!' });
```

这适用于 chrome.* API 中所有图片引用——通知、`chrome.action.setIcon`、上下文菜单图标等。**引用文件就必须存在。**

#### 16. 标签页捕获：用状态锁定防止重复启动

`chrome.tabCapture.getMediaStreamId()` 在先前捕获仍活跃时调用会失败并报 `"Cannot capture a tab with an active stream"`。快速双击扩展图标很容易触发。使用显式状态锁定：

```js
// ❌ 错误——没有防止快速点击的保护
let isRecording = false;
chrome.action.onClicked.addListener(async (tab) => {
  if (isRecording) { stopRecording(); isRecording = false; }
  else { isRecording = true; startRecording(tab); } // 第二次点击 = "active stream" 错误
});

// ✅ 正确——使用过渡状态锁定并发操作
// 状态机：'idle' → 'starting' → 'recording' → 'stopping' → 'idle'
// 状态存于 chrome.storage.session（SW 重启后保留，浏览器关闭时清除）
chrome.action.onClicked.addListener(async (tab) => {
  const { recordingState = 'idle' } = await chrome.storage.session.get('recordingState');

  if (recordingState === 'starting' || recordingState === 'stopping') return;

  if (recordingState === 'idle') {
    await chrome.storage.session.set({ recordingState: 'starting' });
    try {
      await startRecording(tab);
      await chrome.storage.session.set({ recordingState: 'recording' });
      await chrome.action.setBadgeText({ text: 'REC' });
      await chrome.action.setBadgeBackgroundColor({ color: '#FF0000' });
    } catch (err) {
      console.error('Failed to start recording:', err);
      await chrome.storage.session.set({ recordingState: 'idle' });
    }
  } else if (recordingState === 'recording') {
    await chrome.storage.session.set({ recordingState: 'stopping' });
    try { await stopRecording(); }
    finally {
      await chrome.storage.session.set({ recordingState: 'idle' });
      await chrome.action.setBadgeText({ text: '' });
    }
  }
});
```

此模式适用于任何管理独占资源的 chrome API：`chrome.tabCapture`、`chrome.desktopCapture`、`chrome.offscreen.createDocument`（同一时间只允许一个 offscreen 文档）。见 `references/extensions/media-capture.md`。

#### 17. `chrome.desktopCapture` 需要具有 URL 访问权限的目标标签页

从 service worker 调用 `chrome.desktopCapture.chooseDesktopMedia()` 时，必须将活动标签页作为 `targetTab` 参数传入。标签页对象的 `url` 字段必须已填充，这需要 `"tabs"` 权限。

```js
// ❌ 错误——从 service worker 调用时未传 targetTab
chrome.desktopCapture.chooseDesktopMedia(['screen', 'window'], (streamId) => { ... });
// Error: "A target tab is required when called from a service worker context."

// ❌ 错误——标签页没有 url 字段（缺少 "tabs" 权限）
const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
chrome.desktopCapture.chooseDesktopMedia(['screen', 'window'], tab, (streamId) => { ... });
// Error: "targetTab doesn't have URL field set."

// ✅ 正确——manifest 中有 "tabs" 权限 + 传入标签页对象
// manifest.json: { "permissions": ["tabs", "desktopCapture"] }
const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
chrome.desktopCapture.chooseDesktopMedia(['screen', 'window'], tab, (streamId) => {
  if (!streamId) return; // 用户取消
});
```

**注意：** 仅录制标签页时优先用 `chrome.tabCapture.getMediaStreamId()`。仅在用户应选择捕获哪个屏幕/窗口时使用 `chrome.desktopCapture`。见 `references/extensions/media-capture.md`。

#### 18. 用户脚本：四个不明显的陷阱

`chrome.userScripts` 在运行时执行**用户提供的代码**。用于脚本管理器和用户自动化——不用于扩展自带的脚本。

- **未启用时访问属性会抛错。** Chrome 138+ 要求用户在扩展详情页开启"允许用户脚本"；Chrome < 138 需要开发者模式。任何 `chrome.userScripts.*` 调用前都要先调用 `isUserScriptsAvailable()`，返回 false 时显示错误 UI。
- **扩展更新时已注册脚本会被清除。** 将配置持久化到 `chrome.storage`；在 `runtime.onInstalled` 的 `"update"` 原因中重新注册。
- **消息传递需要显式启用。** 先调用 `configureWorld({ messaging: true })`；监听 `runtime.onUserScriptMessage`，而非 `runtime.onMessage`。
- **`ScriptSource` 约束：** 每个 `js` 条目必须恰好有 `code` 或 `file` 之一。**`id` 约束：** 不能以下划线开头。

见 `references/extensions/user-scripts.md`。

#### 19. `chrome.windows` 没有 `.query()` 方法——用 `getAll`、`getLastFocused` 或 `getCurrent`

与 `chrome.tabs.query()` 不同，`chrome.windows` API 没有 `.query()` 方法。

```js
// ❌ 错误——chrome.windows.query 不存在
const windows = await chrome.windows.query({ focused: true });
// TypeError: chrome.windows.query is not a function

// ✅ 正确——按需使用正确的方法
const focused = await chrome.windows.getLastFocused({ populate: true });
const current = await chrome.windows.getCurrent({ populate: true });
```