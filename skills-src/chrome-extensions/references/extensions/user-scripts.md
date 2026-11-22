# 用户脚本 API

`chrome.userScripts` API 让扩展在运行时执行**用户提供的任意代码**——这些代码无法作为扩展包的一部分发布。这与内容脚本（随扩展打包）和 `chrome.scripting`（以编程方式执行扩展自有代码）有本质区别。

**使用 userScripts 的场景：** 你正在构建脚本管理器、自定义自动化工具，或任何需要用户提供 JavaScript 并在网页上运行的功能。
**使用内容脚本的场景：** 注入的代码由你编写，并随扩展一起发布。
**使用 `chrome.scripting.executeScript` 的场景：** 你需要一次性执行已知的、扩展自有的代码。

## Manifest

```json
{
  "manifest_version": 3,
  "minimum_chrome_version": "120",
  "permissions": ["userScripts"],
  "host_permissions": ["https://example.com/*"]
}
```

- 必须声明 `"userScripts"` 权限——否则 `chrome.userScripts` 为 undefined。
- `host_permissions` 必须覆盖脚本将要注入的站点。

## 用户启用——关键

**`chrome.userScripts` API 需要用户显式启用。否则访问属性时 API 会抛错。** 行为因 Chrome 版本而异：

| Chrome 版本 | 要求 |
|----------------|-------------|
| < 138 | 用户必须在 `chrome://extensions` 启用**开发者模式** |
| ≥ 138 | 用户必须在扩展详情页开启 **"允许用户脚本"** |

```js
// ❌ 错误——用户未启用 API 时会崩溃
await chrome.userScripts.register([{ id: 'foo', matches: [...], js: [...] }]);
// TypeError: Cannot read properties of undefined

// ✅ 正确——调用任何 chrome.userScripts.* 前始终先检查
function isUserScriptsAvailable() {
  try {
    chrome.userScripts; // 未启用时抛错
    return true;
  } catch {
    return false;
  }
}

if (!isUserScriptsAvailable()) {
  document.getElementById('warning').style.display = 'block';
  document.getElementById('main-ui').style.display = 'none';
  return;
}
```

## 执行环境

| 环境 | 常量 | 行为 |
|-------|----------|----------|
| **USER_SCRIPT**（默认） | `ExecutionWorld.USER_SCRIPT` | 与宿主页面 JS 隔离；不受页面 CSP 限制 |
| **MAIN** | `ExecutionWorld.MAIN` | 与宿主页面共享 JS 上下文；可访问页面变量 |

为安全起见使用 `USER_SCRIPT`（默认值）。仅当用户脚本明确需要与宿主页面的 JavaScript 环境交互时才使用 `MAIN`。

Chrome 133+ 通过 `worldId` 支持多个隔离环境，让不同用户脚本在各自独立的执行环境中运行，互不干扰。

## 注册脚本（持久化）

`register()` / `update()` / `getScripts()` / `unregister()` 管理跨页面加载和浏览器会话持久存在的脚本。已注册的脚本在 Service Worker 终止后仍然存在。

```js
// ❌ 错误——同时指定了 code 和 file（ScriptSource 必须且只能有一个）
await chrome.userScripts.register([{
  id: 'my-script',
  matches: ['https://example.com/*'],
  js: [{ code: 'alert(1)', file: 'user-script.js' }] // 错误：只能指定 code 或 file，不能同时指定
}]);

// ❌ 错误——id 以下划线开头（保留前缀）
await chrome.userScripts.register([{ id: '_my-script', ... }]);
// 错误：用户脚本 ID 不能以 '_' 开头

// ✅ 正确——从扩展打包的文件注册
await chrome.userScripts.register([{
  id: 'my-user-script',
  matches: ['https://example.com/*'],
  js: [{ file: 'user-script.js' }],
  runAt: 'document_idle'  // 可选，这是默认值
}]);

// ✅ 正确——注册用户提供的内联代码
await chrome.userScripts.register([{
  id: 'my-user-script',
  matches: ['https://example.com/*'],
  js: [{ code: userProvidedCode }]
}]);
```

调用 `register()` 还是 `update()` 前务必先检查——注册已存在的 ID 会抛错：

```js
const existing = await chrome.userScripts.getScripts({ ids: ['my-user-script'] });
if (existing.length > 0) {
  await chrome.userScripts.update([{ id: 'my-user-script', js: [{ code: updatedCode }] }]);
} else {
  await chrome.userScripts.register([{
    id: 'my-user-script',
    matches: ['https://example.com/*'],
    js: [{ code: userProvidedCode }]
  }]);
}

// 移除指定脚本
await chrome.userScripts.unregister({ ids: ['my-user-script'] });

// 移除所有已注册的用户脚本
await chrome.userScripts.unregister();
```

## 持久化：扩展更新后恢复

**扩展更新时，已注册的用户脚本会被清除。**

```js
// ❌ 错误——安装时注册一次的脚本在每次更新后都会丢失
chrome.runtime.onInstalled.addListener(({ reason }) => {
  if (reason === 'install') {
    chrome.userScripts.register([{ id: 'foo', matches: [...], js: [...] }]);
    // 下次扩展更新后，'foo' 就没了——没有重新注册
  }
});

// ✅ 正确——将配置持久化到 storage，在安装和更新时都恢复
chrome.runtime.onInstalled.addListener(async ({ reason }) => {
  if (reason === chrome.runtime.OnInstalledReason.UPDATE) {
    const { scripts = {} } = await chrome.storage.local.get('scripts');
    for (const s of Object.values(scripts)) {
      await chrome.userScripts.register([s]).catch(() => {});
    }
  }
});
```

用户每次保存脚本时，将完整脚本配置（id、matches、js）保存到 `chrome.storage`，以便更新后恢复。

## 一次性注入（Chrome 135+）

`execute()` 立即将脚本注入指定标签页/框架，无需注册。它不会跨页面加载持久存在。

```js
// 需要 Chrome 135+
const results = await chrome.userScripts.execute({
  target: { tabId: tabId },
  js: [{ code: userProvidedCode }],
  world: 'USER_SCRIPT'  // 可选，默认值
});

for (const result of results) {
  if (result.error) {
    console.error('框架注入失败', result.frameId, result.error);
  } else {
    console.log('来自框架的结果', result.frameId, result.result);
  }
}
```

检查 Chrome 135+ 是否可用：

```js
if (typeof chrome.userScripts.execute === 'function') {
  await chrome.userScripts.execute({ ... });
} else {
  // 回退到基于注册的方式
}
```

## 用户脚本中的消息传递

用户脚本运行在 `USER_SCRIPT` 环境中，默认无法访问 `chrome.runtime`。消息传递需要显式启用。

```js
// ❌ 错误——未启用消息传递；用户脚本中 chrome.runtime 为 undefined
// sw.js
chrome.runtime.onMessage.addListener((msg) => { ... }); // 永远不会从用户脚本触发

// 用户脚本代码（在页面中运行）
chrome.runtime.sendMessage({ type: 'hello' }); // TypeError: chrome is not defined

// ✅ 正确——先启用，再使用专用监听器
// sw.js（运行一次，例如在安装时）
await chrome.userScripts.configureWorld({ messaging: true });

// sw.js——监听专用处理器，而非 onMessage
chrome.runtime.onUserScriptMessage.addListener((message, sender, sendResponse) => {
  sendResponse({ ok: true });
  return true; // 保持通道开放以支持异步响应
});

// 用户脚本代码——现在 chrome.runtime 可用了
chrome.runtime.sendMessage({ type: 'FROM_USER_SCRIPT', data: 42 });
```

长连接使用 `chrome.runtime.onUserScriptConnect`（类似于内容脚本的 `runtime.onConnect`）。

Chrome 133+ 通过 `worldId` 支持按环境传递消息：

```js
await chrome.userScripts.configureWorld({ worldId: 'my-world', messaging: true });
```

## `configureWorld()`——CSP 与消息传递

```js
await chrome.userScripts.configureWorld({
  csp: "script-src 'self'",  // 该环境的自定义 CSP
  messaging: true             // 启用 chrome.runtime 消息传递
});

// Chrome 133+：配置命名环境
await chrome.userScripts.configureWorld({
  worldId: 'my-world',
  messaging: true
});

// Chrome 133+：重置环境配置
await chrome.userScripts.resetWorldConfiguration('my-world');

// Chrome 133+：列出所有环境配置
const configs = await chrome.userScripts.getWorldConfigurations();
```

## 完整的脚本管理器模式

```js
// options.js——保存用户脚本并（重新）注册
async function saveScript(id, matches, code) {
  // 持久化配置，以便扩展更新后恢复
  const { scripts = {} } = await chrome.storage.local.get('scripts');
  scripts[id] = { id, matches, js: [{ code }] };
  await chrome.storage.local.set({ scripts });

  // 注册或更新运行中的脚本
  const existing = await chrome.userScripts.getScripts({ ids: [id] });
  if (existing.length > 0) {
    await chrome.userScripts.update([{ id, matches, js: [{ code }] }]);
  } else {
    await chrome.userScripts.register([{ id, matches, js: [{ code }] }]);
  }
}
```

## 与内容脚本的关键区别

| | 内容脚本 | userScripts |
|--|-----------------|-------------|
| 代码来源 | 随扩展打包 | 用户在运行时提供 |
| 持久化 | 自动（manifest） | 手动（注册 + 更新时恢复） |
| 任意代码 | 否 | 是 |
| 用户启用 | 否 | 是（开发者模式 / 允许用户脚本） |
| 消息传递 | `runtime.sendMessage` | `runtime.onUserScriptMessage`（需先 `configureWorld`） |
| CSP 豁免 | 是 | 是（USER_SCRIPT 环境） |
| 多环境 | 否 | 是（Chrome 133+ 通过 `worldId`） |