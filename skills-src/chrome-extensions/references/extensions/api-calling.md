# 从扩展调用外部 API

## 权限

通常，扩展发起的 fetch 请求遵循常规 CORS 规则。

要判断这是否足够，用 `curl` 以测试源调用该 API。例如：

```
curl -H "Origin: https://example.com" -I https://api.openweathermap.org/data/2.5/weather?q=London&appid=KEY`
```

如果响应的 `Access-Control-Allow-Origin` 头值为 `*` 或 `https://example.com`，说明该 API 支持 CORS。

如果 API 不支持 CORS，申请主机权限以绕过这些限制：

```json
{
  "host_permissions": [
    "https://no-cors-api.example.com/*"
  ]
}
```

**不要仅为 API 调用使用 `<all_urls>`。** 应限定到具体的 API 域名。

## 在何处发起 API 调用

API 调用可在任何扩展上下文中进行（service worker、popup、侧边栏、内容脚本）：

```js
// From popup or service worker
const response = await fetch('https://api.openweathermap.org/data/2.5/weather?q=London&appid=KEY');
const data = await response.json();
```

**内容脚本**也能发起 fetch 调用，但遵循网页的 CORS 规则。

## 错误处理模式

```js
async function callAPI(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return await response.json();
  } catch (err) {
    if (err instanceof TypeError) {
      // Network error (offline, DNS failure, etc.)
      console.error('Network error:', err.message);
    } else {
      console.error('API error:', err.message);
    }
    return null;
  }
}
```

## API 密钥

- 绝不要在已发布的扩展中硬编码 API 密钥
- 用户提供的密钥使用 `chrome.storage.local` 存储
- 对于自己的后端，使用 `chrome.identity` 进行认证，而非内嵌密钥
- 明确标记占位密钥：`const API_KEY = 'YOUR_API_KEY_HERE';`

## Service Worker 注意事项

如果从 service worker 发起 API 调用，请记住它可能被终止。对于长轮询或 webhook 式模式，使用 `chrome.offscreen` 创建保持存活的离屏文档，或使用 `chrome.alarms` 进行周期性轮询。