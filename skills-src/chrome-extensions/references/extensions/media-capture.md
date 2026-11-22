# 媒体捕获（标签页与桌面）

## 选择合适的 API

| 需求 | API |
|------|-----|
| 录制当前标签页的音视频 | `chrome.tabCapture.getMediaStreamId()` |
| 让用户选择屏幕、窗口或标签页 | `chrome.desktopCapture.chooseDesktopMedia()` |

只需捕获当前标签页时优先用 `tabCapture`——它不需要用户选择对话框，也不需要 `"tabs"` 权限。仅当必须由用户选择捕获对象时才用 `desktopCapture`。

## 标签页捕获

### 权限

```json
{ "permissions": ["tabCapture"] }
```

### 模式

`chrome.tabCapture.getMediaStreamId()` 在 **service worker** 中运行，返回一个流 ID。实际的 `getUserMedia()` 调用必须在 **offscreen document** 中进行（service worker 无法直接访问媒体流）。

```js
// service-worker.js
chrome.action.onClicked.addListener(async (tab) => {
  const streamId = await chrome.tabCapture.getMediaStreamId({ targetTabId: tab.id });
  // Pass the ID to the offscreen document to call getUserMedia
  await chrome.runtime.sendMessage({ type: 'START_CAPTURE', streamId });
});

// offscreen.js
chrome.runtime.onMessage.addListener((msg) => {
  if (msg.type !== 'START_CAPTURE') return;
  (async () => {
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: { mandatory: { chromeMediaSource: 'tab', chromeMediaSourceId: msg.streamId } },
      video: { mandatory: { chromeMediaSource: 'tab', chromeMediaSourceId: msg.streamId } }
    });
    const recorder = new MediaRecorder(stream, { mimeType: 'video/webm' });
    // ... handle recorder events
  })();
});
```

## 桌面捕获

### 权限

```json
{ "permissions": ["tabs", "desktopCapture"] }
```

`"tabs"` 是**必需**的——`chooseDesktopMedia` 需要一个 `url` 字段已填充的 `targetTab`，而这需要 `"tabs"` 权限。

### 模式

```js
// service-worker.js
chrome.action.onClicked.addListener(async (tab) => {
  // ❌ BROKEN — no targetTab
  // chrome.desktopCapture.chooseDesktopMedia(['screen', 'window'], cb);

  // ✅ CORRECT — pass the active tab
  chrome.desktopCapture.chooseDesktopMedia(['screen', 'window', 'tab'], tab, (streamId) => {
    if (!streamId) return; // User cancelled
    // Send streamId to offscreen document for getUserMedia
    chrome.runtime.sendMessage({ type: 'START_DESKTOP_CAPTURE', streamId });
  });
});
```

## 状态锁定——防止重复启动错误

若上一次捕获仍处于活动状态时再次调用，两个 API 都会失败：
- `tabCapture`：`"Cannot capture a tab with an active stream"`
- `desktopCapture`：会在第一个选择对话框之上再打开一个

使用存储在 `chrome.storage.session` 中的状态机（可跨 service worker 重启存活，浏览器关闭时清除）：

```js
// State: 'idle' → 'starting' → 'recording' → 'stopping' → 'idle'
chrome.action.onClicked.addListener(async (tab) => {
  const { recordingState = 'idle' } = await chrome.storage.session.get('recordingState');

  // Ignore clicks during transitions
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

同样的模式适用于 `chrome.offscreen.createDocument`（同一时间只允许一个 offscreen document）以及任何其他管理独占资源的 API。

## 保存录制内容

Offscreen document 无法调用 `chrome.downloads`——需将 blob 发回 service worker：

```js
// offscreen.js — when recording stops
recorder.ondataavailable = (e) => chunks.push(e.data);
recorder.onstop = async () => {
  const blob = new Blob(chunks, { type: 'video/webm' });
  const url = URL.createObjectURL(blob);
  // Service worker handles the download
  await chrome.runtime.sendMessage({ type: 'SAVE_RECORDING', url });
};

// service-worker.js
chrome.runtime.onMessage.addListener((msg) => {
  if (msg.type !== 'SAVE_RECORDING') return;
  chrome.downloads.download({ url: msg.url, filename: 'recording.webm' });
});
```

完整的 offscreen document 消息传递模式见 `references/extensions/message-passing.md`。