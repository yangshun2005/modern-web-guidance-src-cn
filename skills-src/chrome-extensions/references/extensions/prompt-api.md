# Chrome Prompt API (LanguageModel) — 扩展专属说明

`LanguageModel` API（Prompt API）可在所有扩展上下文中使用——Service Worker、弹出窗口、侧边栏及其他扩展页面——无需额外声明 manifest 权限。

通用 Prompt API 用法（可用性检查、创建会话、流式输出、会话管理）请参阅 `modern-web-guidance` 技能。

## 已弃用的命名空间

曾使用源试用（origin trial）的扩展可能仍保留旧 API。请移除：

```js
// ❌ 旧版 — 已弃用
const session = await self.ai.languageModel.create();

// ✅ 当前版本（Chrome 138+）
const session = await LanguageModel.create({ ... });
```

同时从 manifest.json 中移除已过期的权限：
```json
"permissions": ["aiLanguageModelOriginTrial"]  // ❌ 移除此项
```

## 扩展专属：`LanguageModel.params()`

扩展可访问 `LanguageModel.params()`，返回 Web 端不可用的模型约束：

```js
const params = await LanguageModel.params();
// { defaultTopK: 3, maxTopK: 128, defaultTemperature: 1, maxTemperature: 2 }

const session = await LanguageModel.create({
  temperature: 0.7,
  topK: 5
});
```

## 完整扩展示例：页面摘要器

完整串联示例，展示 manifest + Service Worker + 侧边栏。注意使用 `tabs` + `host_permissions` 而非 `activeTab`——点击侧边栏按钮**不会**激活 `activeTab`（见规则 12）。

### manifest.json
```json
{
  "manifest_version": 3,
  "name": "AI Page Summarizer",
  "version": "1.0",
  "permissions": ["sidePanel", "tabs", "scripting"],
  "host_permissions": ["<all_urls>"],
  "background": { "service_worker": "service-worker.js" },
  "side_panel": { "default_path": "sidepanel/sidepanel.html" },
  "action": { "default_title": "Summarize Page" }
}
```

### service-worker.js
```js
chrome.action.onClicked.addListener(async (tab) => {
  await chrome.sidePanel.open({ windowId: tab.windowId });
});
```

### sidepanel/sidepanel.js
```js
const statusEl = document.getElementById('status');
const summaryEl = document.getElementById('summary');

document.getElementById('summarize').addEventListener('click', async () => {
  if (!globalThis.LanguageModel) {
    statusEl.textContent = 'Prompt API not available in this browser.';
    return;
  }

  const availability = await LanguageModel.availability({
    expectedInputs: [{ type: "text", languages: ["en"] }],
    expectedOutputs: [{ type: "text", languages: ["en"] }]
  });
  if (availability === 'unavailable') {
    statusEl.textContent = 'AI model not available on this device.';
    return;
  }

  // 需要 "tabs" + "host_permissions" — activeTab 无法从侧边栏按钮生效
  const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
  const [{ result: pageText }] = await chrome.scripting.executeScript({
    target: { tabId: tab.id },
    func: () => {
      const body = document.body.cloneNode(true);
      body.querySelectorAll('script, style, nav, footer, header').forEach(el => el.remove());
      return body.innerText.substring(0, 4000);
    }
  });

  const session = await LanguageModel.create({
    expectedInputs: [{ type: "text", languages: ["en"] }],
    expectedOutputs: [{ type: "text", languages: ["en"] }],
    initialPrompts: [{ role: 'system', content: 'Summarize web page content in 3-5 bullet points.' }],
    monitor(m) {
      m.addEventListener('downloadprogress', (e) => {
        const pct = e.total ? Math.floor((e.loaded / e.total) * 100) : 0;
        statusEl.textContent = `Downloading model: ${pct}%`;
      });
    }
  });

  summaryEl.textContent = '';
  for await (const chunk of session.promptStreaming(`Summarize:\n\n${pageText}`)) {
    summaryEl.textContent += chunk; // 追加 — 不要替换
  }
  session.destroy();
});
```