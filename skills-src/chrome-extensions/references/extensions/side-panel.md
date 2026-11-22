# 侧边栏

## 配置

在 manifest.json 中添加：
```json
{
  "permissions": ["sidePanel"],
  "side_panel": {
    "default_path": "sidepanel/sidepanel.html"
  }
}
```

## 打开侧边栏——必须配置

**仅定义侧边栏并不能让它可打开。** 你必须提供显式的触发方式来打开它。缺少触发方式，用户将无法访问面板：

### 最常见：点击操作图标打开

如果扩展的主要功能是侧边栏，移除 action 中的 `default_popup`，改用 `chrome.action.onClicked` 打开侧边栏：

```js
// service-worker.js
chrome.action.onClicked.addListener(async (tab) => {
  await chrome.sidePanel.open({ windowId: tab.windowId });
});
```

⚠️ `chrome.action.onClicked` 仅在未设置 `default_popup` 时触发。如果同时有 popup 和侧边栏，请从 popup 中通过按钮打开侧边栏，或使用其他触发方式。

### 其他触发方式

```js
// 从上下文菜单项打开
chrome.contextMenus.onClicked.addListener(async (info, tab) => {
  if (info.menuItemId === 'open-panel') {
    await chrome.sidePanel.open({ windowId: tab.windowId });
  }
});

// 从键盘快捷键打开（在 manifest commands 中定义）
chrome.commands.onCommand.addListener(async (command) => {
  if (command === 'open-side-panel') {
    const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
    await chrome.sidePanel.open({ windowId: tab.windowId });
  }
});
```

也可以为特定标签页打开：
```js
await chrome.sidePanel.open({ tabId: tab.id });
```

### 最简单：通过 setPanelBehavior 自动打开

如果希望用户点击扩展图标时打开侧边栏，用 `setPanelBehavior` 一行代码代替 `onClicked` 监听器：

```js
// service-worker.js
chrome.sidePanel.setPanelBehavior({ openPanelOnActionClick: true });
```

⚠️ **属性名是 `openPanelOnActionClick`——不是 `openPanelOnActionIconClick`。**
用错名称会触发同步 TypeError，导致 service worker 静默中止。

使用 `setPanelBehavior` 时，不要同时定义 `default_popup`——popup 优先级更高。

## 按标签页设置面板

```js
// 不同标签页使用不同的侧边栏内容
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
  if (tab.url?.includes('github.com')) {
    await chrome.sidePanel.setOptions({
      tabId,
      path: 'sidepanel/github-panel.html',
      enabled: true
    });
  }
});
```

## 与侧边栏通信

侧边栏是扩展页面，可直接使用所有 chrome.* API，并通过 `chrome.runtime.sendMessage` / `chrome.runtime.onMessage` 与 service worker 通信。

从活动标签页的内容脚本获取数据：

```js
// 在侧边栏 JS 中
async function getPageContent() {
  const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
  const response = await chrome.tabs.sendMessage(tab.id, { type: 'GET_CONTENT' });
  return response;
}
```

或在侧边栏中使用 `chrome.scripting.executeScript`（需要 `scripting` 和 `activeTab` 权限）：

```js
const [{ result }] = await chrome.scripting.executeScript({
  target: { tabId: tab.id },
  func: () => document.body.innerText
});
```

## 侧边栏 vs Popup

| 特性 | 侧边栏 | Popup |
|---------|-----------|-------|
| 保持打开 | 是 | 点击别处即关闭 |
| 可调整大小 | 是（由用户调整） | 固定大小 |
| 与页面共存 | 是（并排显示） | 覆盖页面 |
| 适用场景 | 扩展交互、阅读 | 快速操作、设置 |

## 注意事项

- 侧边栏在每个窗口中共享单一实例——打开它会替换现有内容
- 使用 `chrome.sidePanel.setOptions({ enabled: false })` 为特定标签页禁用
- 侧边栏 HTML 文件拥有 chrome.* API 的完整访问权限
- 侧边栏在标签页切换时保持存在（按窗口）

### ⚠️ `activeTab` 无法通过侧边栏交互生效

`activeTab` 仅在直接用户手势下授予标签页访问权限：点击扩展图标、上下文菜单项、键盘快捷键或地址栏建议。**点击侧边栏内的按钮不会激活 `activeTab`。**

如果侧边栏需要读取或修改页面内容（例如"总结"按钮），改用 `tabs` + `host_permissions`：

```json
{
  "permissions": ["tabs", "scripting", "sidePanel"],
  "host_permissions": ["<all_urls>"]
}
```

不要依赖 `activeTab` 实现侧边栏功能。