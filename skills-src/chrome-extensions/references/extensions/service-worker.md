# Service Worker 生命周期与状态管理

## 核心问题

Chrome 会在扩展的 Service Worker 空闲约 30 秒后终止它。与 Manifest V2 的持久后台页面不同，你**不能**依赖内存中的状态。

## 规则

1. **绝不在全局变量中存储状态**——把每个事件处理器都当作 SW 刚刚启动来处理
2. **所有持久状态都用 chrome.storage**——按需读取，变更后写入
3. **定时器用 chrome.alarms**——不要用 setTimeout/setInterval（它们会随 SW 一起终止）
4. **临时会话状态用 chrome.storage.session**——SW 重启后仍保留，但浏览器重启后丢失

## 存储层级选择

| 需求 | 使用 |
|------|-----|
| 浏览器重启后保留，跨设备同步 | `chrome.storage.sync`（每项 8KB，总计 100KB） |
| 浏览器重启后保留，仅本地 | `chrome.storage.local`（默认 10MB） |
| 仅 SW 重启后保留 | `chrome.storage.session`（默认 10MB） |
| 永不持久化（避免使用） | 全局变量 ❌ |

## 模式：按需读取状态

```js
// ❌ 错误：状态存在内存中
let count = 0;
chrome.webNavigation.onCompleted.addListener(() => {
  count++;
  chrome.action.setBadgeText({ text: String(count) });
});

// ✅ 正确：状态存在存储中
chrome.webNavigation.onCompleted.addListener(async (details) => {
  if (details.frameId !== 0) return; // 仅主框架
  const data = await chrome.storage.local.get({ visitCount: 0 });
  data.visitCount++;
  await chrome.storage.local.set(data);
  chrome.action.setBadgeText({ text: String(data.visitCount) });
});
```

## 模式：用 Alarms 替代定时器

```js
// ❌ 错误：SW 终止时定时器随之失效
setInterval(() => checkForUpdates(), 60000);

// ✅ 正确：Alarm 可持久保留
chrome.alarms.create('check-updates', { periodInMinutes: 1 });
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'check-updates') {
    checkForUpdates();
  }
});
```

最小 alarm 间隔：0.5 分钟。

## 模式：一次性初始化

```js
// 在安装时设置默认值和上下文菜单
chrome.runtime.onInstalled.addListener(async (details) => {
  if (details.reason === 'install') {
    await chrome.storage.local.set({ settings: defaultSettings });
  }
  // 上下文菜单必须重新创建（它们会持久保留，但重新创建是幂等的）
  chrome.contextMenus.create({
    id: 'myItem',
    title: 'My Context Menu Item',
    contexts: ['selection']
  });
});
```

## 模式：保持 SW 存活（必要时）

偶尔你需要让 SW 在长时间操作期间保持存活。使用以下方式之一：

1. **chrome.offscreen**——为长任务创建离屏文档
2. **定期写入存储**——每次调用 chrome.storage 都会重置空闲计时器
3. **活跃的端口连接**——打开的端口会让 SW 保持存活

```js
// 从 popup/侧边栏基于端口保持存活
const port = chrome.runtime.connect({ name: 'keepalive' });
// 只要此端口保持打开，SW 就会保持存活
```

⚠️ 不要滥用保活模式。Chrome 可能在后续版本中实施更严格的限制。

## 模式：事件注册

所有事件监听器**必须**在 Service Worker 顶层同步注册。Chrome 会向重启的 SW 重放事件，但仅限同步注册的监听器。

```js
// ✅ 正确：顶层注册
chrome.runtime.onMessage.addListener(handleMessage);
chrome.tabs.onUpdated.addListener(handleTabUpdate);
chrome.webNavigation.onCompleted.addListener(handleNavigation);

// ❌ 错误：条件或异步注册
async function setup() {
  const { enabled } = await chrome.storage.local.get('enabled');
  if (enabled) {
    chrome.tabs.onUpdated.addListener(handleTabUpdate); // 太晚了！
  }
}
setup();
```

应改为注册所有监听器，并在监听器内部检查条件：

```js
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
  const { enabled } = await chrome.storage.local.get('enabled');
  if (!enabled) return;
  // 处理...
});
```

## 基于日期的重置

对于每日计数器，将日期与计数一起存储：

```js
function getToday() {
  return new Date().toISOString().split('T')[0]; // "2025-01-15"
}

async function incrementDailyCount() {
  const { dailyCount = 0, countDate = '' } = await chrome.storage.local.get(['dailyCount', 'countDate']);
  const today = getToday();

  if (countDate !== today) {
    // 新的一天——重置
    await chrome.storage.local.set({ dailyCount: 1, countDate: today });
    return 1;
  } else {
    const newCount = dailyCount + 1;
    await chrome.storage.local.set({ dailyCount: newCount });
    return newCount;
  }
}
```