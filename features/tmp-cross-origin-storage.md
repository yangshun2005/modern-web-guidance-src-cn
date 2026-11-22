## 浏览器支持

跨源存储（COS）正在 Chromium 中实现，目前尚无主流浏览器原生支持。在正式发布前，跨源存储浏览器扩展可将其引入 Chrome 及其他基于 Chromium 的浏览器（通过 Chrome 应用商店）、桌面端和 Android 上的 Firefox（通过 Firefox 附加组件），以及 macOS、iOS 和 iPadOS 上的 Safari（通过 App Store）。该扩展支持 `navigator.crossOriginStorage` API 的字面语法、HTML `crossoriginstorage` 属性以及 CSS `cross-origin-storage()` 修饰符。它不支持字面 `crossOriginStorage` 导入属性，因为导入属性语法无法通过 polyfill 实现。

## 共享范围

每个跨源存储（COS）接口都接受一个共享范围，用于控制之后哪些源可以获取所存储的文件。请选择与资源实际分发情况相符的范围：

- **仅同站点（默认）：** 适用于仅你自己的站点使用的资源。
- **显式源列表：** 适用于少量受信任的源，例如在你自己的多个资产之间共享的专有资源。
- **全局（`*`）：** **强制要求：** 仅适用于真正流行且非专有的资源，例如开放权重的 AI 模型或广泛使用的开源库。

| 接口 | 仅同站点 | 源列表 | 全局 |
|---|---|---|---|
| `getFileHandle()` 的 `origins` 选项 | 省略 `origins` | 源字符串数组 | `'*'` |
| HTML `crossoriginstorage` 属性 | 无值属性 | 以空格分隔的源 | `"*"` |
| `crossOriginStorage` 导入属性 | 空字符串（`''`） | 以空格分隔的源 | `'*'` |
| CSS `cross-origin-storage()` 修饰符 | 无参数 | 以逗号分隔的源字符串 | `*` |

**强制要求：** 源列表仅对 `Cross-Origin-Storage-Allow-Origin` 响应头同时列出的源生效，例如 `Cross-Origin-Storage-Allow-Origin: https://a.example, https://b.example`。该响应头来自提供字节的一方：对于 HTML、导入属性和 CSS 形式，来自调用 `getFileHandle()` 的页面的响应，或来自所获取资源本身（脚本、样式表、模块或字体）的响应。响应头未列出的源会被丢弃；若一个都不剩，文件将以同站点默认范围存储。同站点默认范围和 `*` 无需响应头。

## 命名

- **不要**将 HTML `crossoriginstorage` 属性与无关的 `crossorigin` 属性混淆。`crossorigin` 设置 CORS 请求模式，两者可在同一元素上共存。
- **不要**将 CSS `cross-origin-storage()` 修饰符与无关的 CSS `cross-origin()` 修饰符混淆，后者同样设置 CORS 请求模式。