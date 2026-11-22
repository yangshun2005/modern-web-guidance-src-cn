# 发布前审核清单

每次提交到 Chrome Web Store 前，请逐项检查本清单。每一项都对应一个常见的拒绝原因或发布失败原因。

## Manifest 与包

- [ ] **manifest_version 为 3** — 新提交不再接受 Manifest V2。
- [ ] **版本号已递增** — CWS 会拒绝版本号 ≤ 当前已发布版本的上传。使用 semver：修复递增 patch，新功能递增 minor，破坏性变更递增 major。
- [ ] **名称与 CHROMEWEBSTORE.md 一致** — manifest.json 中的 `name` 字段必须与商店列表中的完全一致。
- [ ] **manifest 中的描述 ≤ 132 字符** — 这是 chrome://extensions 中显示的简短描述。应与 CWS 简短描述一致或接近。
- [ ] **ZIP 中无多余文件** — 排除：`.git/`、`node_modules/`、`.env`、`*.map`、测试文件、构建配置、`CHROMEWEBSTORE.md` 本身、`README.md`、`.DS_Store`、`thumbs.db`。使用构建脚本或类似 `.cws-ignore` 的排除机制。
- [ ] **ZIP 小于 2GB** — 包大小上限。大多数扩展应小于 10MB。
- [ ] **无绝对文件路径** — manifest.json 中的所有路径必须为相对路径。

## 权限

- [ ] **最小权限** — 只申请需要的权限。`<all_urls>` 是危险信号。尽可能使用具体的主机权限，如 `*://*.example.com/*`。
- [ ] **每个权限都有正当理由** — 检查 CHROMEWEBSTORE.md 中的权限理由部分。CWS 后台为每个权限设有字段——提交时需填写。
- [ ] **优先使用 activeTab 而非 tabs + <all_urls>** — 如果只需在用户点击图标时访问当前标签页，`activeTab` 是正确的权限。
- [ ] **无未使用的权限** — 如果移除了使用某权限的功能，也要从 manifest.json 中移除该权限。残留权限会导致拒绝。
- [ ] **host_permissions 有正当理由** — 说明哪些功能需要访问哪些域名及原因。

## 商店列表内容

- [ ] **详细描述具体明确** — 准确描述扩展的功能。不要用模糊的营销语言。审核团队会阅读此内容。
- [ ] **单一用途范围狭窄** — 用一句话清楚说明主要功能。"将书签整理到分类文件夹"而非"生产力增强工具"。
- [ ] **无误导性声明** — 不要声称没有的功能。不要夸大性能声明。
- [ ] **无关键词堆砌** — 不要在描述中不自然地重复关键词。
- [ ] **无商标侵权** — 未经授权，不要在扩展名称、描述或截图中使用其他公司的名称、徽标或商标。
- [ ] **联系邮箱有效** — 列表中显示的邮箱必须有人监控。Google 会向此地址发送重要通知（下架、政策变更）。

## 图形素材

- [ ] **商店图标**：128×128 PNG，无透明度问题，小尺寸下可读。
- [ ] **至少 1 张截图**：1280×800 或 640×400 像素。展示扩展的实际运行效果。
- [ ] **截图是最新的** — 与扩展 UI 的当前版本一致。过时的截图可能触发拒绝。
- [ ] **无误导性截图** — 截图必须准确反映扩展功能。
- [ ] **无手机/平板样机图** — 除非扩展确实在这些设备上可用。
- [ ] **小型宣传图块**（推荐）：440×280 PNG 或 JPEG。用于精选展示位。

## 隐私与合规

- [ ] **数据披露表单与实际一致** — CWS 数据使用披露复选框必须准确反映扩展代码的实际行为。不一致 = 拒绝。
- [ ] **隐私政策 URL 可访问** — 亲自访问该 URL。确认能加载且包含实际的隐私政策，而非 404 或占位内容。
- [ ] **隐私政策与披露一致** — 政策文本必须与披露表单中声明的内容一致。
- [ ] **已披露 chrome.storage.sync** — 如果使用 `chrome.storage.sync`，数据会传输到 Google 服务器。这算作设备外传输。
- [ ] **禁止远程代码** — 扩展不能执行远程托管的代码。所有 JS 必须打包在扩展包中。运行时不得从 CDN 加载脚本（Manifest V3 强制此要求，但仍需验证）。
- [ ] **无混淆代码** — 压缩可以。混淆（故意使代码不可读）被禁止，会导致拒绝。

## 功能

- [ ] **扩展可运行** — 在 Chrome 中以未打包方式加载，测试所有功能。检查控制台是否有错误。
- [ ] **无崩溃或空白弹窗** — 测试弹窗、侧边栏、选项页、内容脚本。所有都应无错误加载。
- [ ] **在目标网站上可用** — 如果扩展针对特定网站，验证其在那些网站的当前版本上可用。
- [ ] **优雅降级** — 扩展应处理边界情况（无网络、空数据、chrome:// 等受限页面）而不崩溃。
- [ ] **卸载干净** — 扩展移除后无持久副作用。
- [ ] **无过度资源占用** — 扩展不应明显拖慢浏览。内容脚本尤其应轻量。

## 更新（针对已有扩展）

- [ ] **CHROMEWEBSTORE.md 版本历史已更新** — 新增条目包含版本、日期和变更摘要。
- [ ] **Last Updated 日期已更新** — 如果做了任何用户可见的变更。
- [ ] **描述中的功能列表已更新** — 如果添加了新功能。
- [ ] **权限理由已更新** — 如果 manifest.json 权限有变更。
- [ ] **截图已刷新** — 如果 UI 有显著变化。
- [ ] **隐私披露已更新** — 如果数据处理方式有变更。

## 打包脚本

要创建干净的提交 ZIP，使用如下脚本：

```bash
#!/bin/bash
# package-extension.sh — Creates a clean ZIP for Chrome Web Store submission

EXTENSION_NAME="my-extension"
VERSION=$(node -p "require('./manifest.json').version")
OUTPUT="${EXTENSION_NAME}-v${VERSION}.zip"

# Remove old package
rm -f "$OUTPUT"

# Create ZIP excluding dev files
zip -r "$OUTPUT" . \
  -x ".git/*" \
  -x "node_modules/*" \
  -x ".env" \
  -x "*.map" \
  -x "tests/*" \
  -x "__tests__/*" \
  -x "*.test.*" \
  -x "*.spec.*" \
  -x ".eslintrc*" \
  -x ".prettierrc*" \
  -x "tsconfig.json" \
  -x "package.json" \
  -x "package-lock.json" \
  -x "webpack.config.*" \
  -x "vite.config.*" \
  -x "rollup.config.*" \
  -x "CHROMEWEBSTORE.md" \
  -x "README.md" \
  -x "CHANGELOG.md" \
  -x ".DS_Store" \
  -x "Thumbs.db" \
  -x "*.sh" \
  -x "store-assets/*"

echo "Packaged: $OUTPUT ($(du -h "$OUTPUT" | cut -f1))"
```

根据项目自定义排除列表。关键原则：只打包 Chrome 运行扩展所需的内容。