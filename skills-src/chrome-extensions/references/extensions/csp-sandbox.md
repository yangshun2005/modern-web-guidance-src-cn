# CSP 与沙箱代码执行

## 扩展 CSP 限制

Chrome 扩展对扩展页面（popup、侧边栏、选项页、新标签页等）强制执行严格的内容安全策略，且无法放宽。

默认被阻止：
- `eval()`、`new Function()`、`setTimeout("string")`
- 内联 `<script>` 标签
- 内联事件处理器（`onclick="..."`、`onload="..."` 等）
- `javascript:` URL

## HTML 最佳实践

```html
<!-- ❌ 错误：内联脚本 -->
<script>
  document.getElementById('btn').onclick = () => alert('hi');
</script>

<!-- ❌ 错误：内联事件处理器 -->
<button onclick="doThing()">Click</button>

<!-- ✅ 正确：外部脚本文件 -->
<script src="popup.js"></script>
```

在 `popup.js` 中：
```js
document.getElementById('btn').addEventListener('click', () => {
  // 处理点击
});
```

## 执行用户代码（代码演练场模式）

如果需要执行任意代码（例如类似 CodePen 的演练场），必须采用以下方案之一。**扩展 CSP 会完全阻止普通扩展页面中的 `eval()`、`new Function()` 和内联脚本。** 无法绕过——必须使用沙箱。

### 方案 1：在 Manifest 中声明沙箱页面（推荐）

在 manifest.json 中声明沙箱页面。沙箱页面的 CSP 较宽松，允许 `eval()` 和内联脚本，但无法访问 chrome.* API。

```json
{
  "sandbox": {
    "pages": ["sandbox.html"]
  }
}
```

在扩展页面中用 iframe 嵌入沙箱：

```html
<!-- playground.html（扩展页面） -->
<iframe id="preview" src="sandbox.html"></iframe>
```

**关键：** 扩展页面与沙箱 iframe 之间的通信必须使用 `postMessage`。不能直接访问 `iframe.contentDocument` 或 `iframe.contentWindow.document`——否则会抛出：

```
SecurityError: Blocked a frame with origin "chrome-extension://..." from accessing a cross-origin frame.
```

正确写法：

```js
// playground.js — 向沙箱发送代码
const iframe = document.getElementById('preview');
iframe.contentWindow.postMessage({
  html: htmlCode,
  css: cssCode,
  js: jsCode
}, '*');

// sandbox.js — 接收并执行
window.addEventListener('message', (event) => {
  const { html, css, js } = event.data;
  // 清除之前的内容
  document.body.innerHTML = '';
  document.head.querySelectorAll('style.user-style').forEach(s => s.remove());

  // 应用 HTML
  const container = document.createElement('div');
  container.innerHTML = html;
  document.body.appendChild(container);

  // 应用 CSS
  const style = document.createElement('style');
  style.className = 'user-style';
  style.textContent = css;
  document.head.appendChild(style);

  // 执行 JS（沙箱中允许 eval！）
  try {
    eval(js);
  } catch (e) {
    const errEl = document.createElement('pre');
    errEl.style.color = 'red';
    errEl.textContent = e.message;
    document.body.appendChild(errEl);
  }
});
```

### 方案 2：iframe 中的 Blob URL

通过 blob URL 创建自包含的 HTML 文档：

```js
function updatePreview(htmlCode, cssCode, jsCode) {
  const html = `
<!DOCTYPE html>
<html>
<head><style>${cssCode}</style></head>
<body>
  ${htmlCode}
  <script>${jsCode}<\/script>
</body>
</html>
`;
  const blob = new Blob([html], { type: 'text/html' });
  const url = URL.createObjectURL(blob);
  const iframe = document.getElementById('preview');
  // 释放之前的 URL
  if (iframe.dataset.blobUrl) URL.revokeObjectURL(iframe.dataset.blobUrl);
  iframe.dataset.blobUrl = url;
  iframe.src = url;
}
```

### 方案 3：srcdoc 属性

```js
const iframe = document.getElementById('preview');
iframe.srcdoc = `
  <!DOCTYPE html>
  <style>${cssCode}</style>
  ${htmlCode}
  <script>${jsCode}<\/script>
`;
```

blob URL 和 srcdoc 都会创建独立的源，因此能绕过扩展的 CSP。但它们同样无法访问 chrome.* API，也无法从扩展页面直接访问其 DOM（与沙箱相同的跨源限制）。

### 不要这样做

```js
// ❌ 会失败：试图直接设置 iframe 内容
iframe.contentDocument.open();
iframe.contentDocument.write(html);
iframe.contentDocument.close();

// ❌ 会失败：访问跨源沙箱 DOM
const doc = iframe.contentWindow.document;
doc.body.innerHTML = html;

// ❌ 会失败：在普通扩展页面中使用 eval
eval(userCode); // CSP 会阻止
```

## 远程资源的 CSP

扩展页面默认无法加载远程脚本。如需外部库：

1. **打包引入**——下载并包含在扩展中
2. **使用 chrome.scripting 注入网页**——网页有自己的 CSP

对于注入网页的内容脚本，网页的 CSP 不适用于内容脚本自身的代码。内容脚本运行在隔离世界中。