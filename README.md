

> **[中文化] modern-web-guidance-src**
>
> 此项目是 [GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src) 的中文翻译版本。
> - 原项目 Stars: 9916
> - 主语言: TypeScript
> - 许可证: Apache-2.0
> - 翻译日期: 2026-11-21
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

Modern Web Guidance 是一套「技能（skills）」，把 Web 平台的专业知识、最佳实践和浏览器兼容性数据直接注入到你的编码 Agent（coding agent）中。它要解决的痛点是：编码 Agent 因为训练数据里充斥着大量遗留代码，常常默认生成老旧的写法，为本来已有原生高性能方案的任务写出臃肿的 JavaScript。这个项目通过 CLI 把经过专家整理、token 高效的指南返回给 Agent，引导它使用现代浏览器 API、性能与无障碍友好的平台能力，以及轻量的降级方案。本仓库是它的**源码仓库**，包含编写、校准和评估这些内容的代码与文档。

### 快速上手（最短路径）

1. 确认本机 Node.js 版本不低于 22（`package.json` 中 `engines` 要求 `>=22.18.0`，README 快速上手处写的是 Node.js 22 或更高）。
2. 运行安装向导：

   ```shell
   npx modern-web-guidance@latest install
   ```

   该命令会启动一个交互式向导来安装 Modern Web Guidance。
3. （可选）不安装、直接试用，搜索相关指南：

   ```shell
   npx modern-web-guidance@latest search "animate a dialog modal backdrop"
   ```

4. （可选）按 ID 取回某篇指南：

   ```shell
   npx modern-web-guidance@latest retrieve "animate-to-from-top-layer"
   ```

5. （可选）扫描代码库中的遗留模式：

   ```shell
   npx modern-web-guidance@latest legacy-scan .
   ```

> 注意：以上命令来自 README 的 Quickstart 与「Try it out」部分，针对的是 `modern-web-guidance` 这个 CLI 包。本仓库自身的开发脚本（如 `pnpm build`、`pnpm test`）属于源码开发流程，README 前 7000 字未展开说明，原文未说明其完整用法。

### 环境要求

- **操作系统**：原文未说明。
- **硬件**：原文未说明。
- **运行时依赖**：Node.js 22 或更高（README Quickstart 明确写「Requires Node.js 22 or later」；`package.json` 的 `engines.node` 为 `>=22.18.0`）。
- **包管理器**：仓库内含 `pnpm-lock.yaml`，说明源码开发使用 pnpm；但 README 前 7000 字未给出源码构建的具体安装命令，原文未说明。
- **版本要求**：CLI 通过 `npx modern-web-guidance@latest` 调用最新版；仓库自身 `version` 为 `1.0.0`。
- **网络提示**：`npx` 需要从 npm 源下载包，国内网络环境如遇下载缓慢，可考虑更换 npm 镜像源或使用加速下载。

### 常见坑与注意事项

- **Node 版本门槛**：必须 Node.js 22 及以上，低于此版本无法运行 CLI。
- **这是源码仓库，不是直接使用的成品**：README 明确本仓库是 `GoogleChrome/modern-web-guidance` 的 source repository，面向编写/校准/评估内容；普通用户应使用 `npx modern-web-guidance@latest` 系列命令，而非直接克隆本仓库使用。
- **安装是交互式向导**：`install` 命令会启动交互式向导，需要按提示操作，不是一条命令静默完成。
- **源码开发流程未在材料中说明**：构建、测试、发布等脚本虽在 `package.json` 中可见，但 README 前 7000 字未给出对应操作说明，需参考 `CONTRIBUTING.md` 与 README 正文。

---



---

# <img src="./.github/img/modern-web-guidance.svg" alt="Modern Web Guidance" width="30" height="30"> Modern Web Guidance（源仓库）

本仓库是 [GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance) 的源仓库，包含用于编写、校准和评估其内容的代码与文档。

[**Modern Web Guidance**](https://developer.chrome.com/docs/modern-web-guidance) 是一组技能，将 Web 平台专业知识、最佳实践和浏览器兼容性数据直接嵌入你的编码智能体。它帮助引导编码智能体远离遗留模式，转而采用能发挥现代 Web 平台能力的解决方案。

关于项目协作方式和代码库结构，请参阅 [CONTRIBUTING.md](./CONTRIBUTING.md) 和 [GOVERNANCE.md](./GOVERNANCE.md)。

---

*由 Google Chrome、Microsoft Edge、Mozilla Firefox 和 MDN 团队以及 Web 开发社区共同维护。*

[文档](https://developer.chrome.com/docs/modern-web-guidance) · [评估仪表盘](https://googlechrome.github.io/modern-web-guidance-src/) · [源代码](https://github.com/GoogleChrome/modern-web-guidance-src) · [报告问题](https://github.com/GoogleChrome/modern-web-guidance-src/issues)

## <img src="https://github.com/GoogleChrome/modern-web-guidance/raw/main/.github/img/terminal.svg" width="24" height="24" style="vertical-align: middle; margin-right: 4px;"> 快速开始

```shell
npx modern-web-guidance@latest install
```

该命令运行交互式向导来安装 Modern Web Guidance。参见下方的[其他安装方式](#-其他安装方式)。需要 Node.js 22 或更高版本。

#### 试用（无需安装）

```shell
# 搜索相关指南
npx modern-web-guidance@latest search "animate a dialog modal backdrop"

# 按 ID 获取指南
npx modern-web-guidance@latest retrieve "animate-to-from-top-layer"

# 扫描代码库中的遗留模式以进行现代化改造
npx modern-web-guidance@latest legacy-scan .
```

## <img src="https://github.com/GoogleChrome/modern-web-guidance/raw/main/.github/img/lightbulb.svg" width="24" height="24" style="vertical-align: middle; margin-right: 4px;"> 为什么需要它？

编码智能体常常默认使用旧模式，因为 LLM 训练数据中包含大量遗留代码。这导致它们为如今已有原生、高性能 Web 平台解决方案的任务生成臃肿的 JavaScript。

即使模型知道某个 API 存在，也往往缺乏生产级代码所需的、密集的真实世界现代实现模式。

**Modern Web Guidance 弥合了这一差距。** 本技能的 CLI 将针对性的、专家精选的指南直接返回至智能体的上下文窗口，重点关注：
* **现代浏览器 API**：帮助模型正确构建它们经常误用的 API。
* **性能与无障碍**：优先使用可被浏览器优化且内置无障碍能力的平台级 API。
* **负责任的回退方案**：引导模型使用合理、轻量的回退方案，而非重型 polyfill 或遗留库。

## <img src="https://github.com/GoogleChrome/modern-web-guidance/raw/main/.github/img/package.svg" width="24" height="24" style="vertical-align: middle; margin-right: 4px;"> 包含内容

我们覆盖了 Web 平台过去数年的新特性，直至最前沿。指南**设计为 token 高效**；我们运行评估，以剔除模型已知的最低共同标准内容。

### 核心领域

<table width="100%" style="border-collapse: collapse; border: none;">
  <tr style="border: none;">
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>🎨 用户体验</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">流畅的视觉状态（View Transitions、进入/退出动画、视差滚动、CSS <code>scrollbar-color</code>）。</p>
    </td>
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>📐 CSS 布局</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">现代布局系统（容器查询、<code>subgrid</code>、<code>oklch</code> 等现代色彩空间、text-wrap 调优、行高修剪）。</p>
    </td>
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>⚡ 性能</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">速度优化（即时预加载、Interaction to Next Paint (INP) 诊断、通过 <code>scheduler.yield</code> 调度任务）。</p>
    </td>
  </tr>
  <tr style="border: none;">
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>📝 表单与 UI</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">原生组件（用于工具提示的锚点定位、Popover API、对话框、<code>:user-invalid</code> 验证、自动调整大小的字段）。</p>
    </td>
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>♿ 无障碍</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">重要考量（屏幕阅读器和键盘可操作性、内容导航和可发现性）。</p>
    </td>
    <td width="33%" valign="top" style="border: none; padding: 6px;">
      <h4>🤖 内置 AI</h4>
      <p style="font-size: 0.9em; line-height: 1.4;">本地客户端模型（原生翻译、摘要和语言检测 API）。</p>
    </td>
  </tr>
</table>

_查看示例：_[`navigation-drawer` 指南](https://github.com/GoogleChrome/modern-web-guidance/blob/main/skills/modern-web-guidance/guides/ui-components/navigation-drawer.md)。

<!-- INJECT_SKILL_COVERAGE_START -->
#### 完整列表

<details>
<summary><strong>262 项现代 Web 特性</strong></summary>

### CSS 与布局（140 项特性）

| | | |
| :--- | :--- | :--- |
| [::backdrop](https://web-platform-dx.github.io/web-features-explorer/features/backdrop/) | [content-visibility](https://web-platform-dx.github.io/web-features-explorer/features/content-visibility/) | [progress()](https://web-platform-dx.github.io/web-features-explorer/features/progress-function/) |
| [::details-content](https://web-platform-dx.github.io/web-features-explorer/features/details-content/) | [contrast-color()](https://web-platform-dx.github.io/web-features-explorer/features/contrast-color/) | [reading-flow](https://web-platform-dx.github.io/web-features-explorer/features/reading-flow/) |
| [::file-selector-button](https://web-platform-dx.github.io/web-features-explorer/features/file-selector-button/) | [跨文档视图过渡](https://web-platform-dx.github.io/web-features-explorer/features/cross-document-view-transitions/) | [相对颜色](https://web-platform-dx.github.io/web-features-explorer/features/relative-color/) |
| [::placeholder](https://web-platform-dx.github.io/web-features-explorer/features/placeholder/) | [currentColor](https://web-platform-dx.github.io/web-features-explorer/features/currentcolor/) | [rem](https://web-platform-dx.github.io/web-features-explorer/features/rem/) |
| [::selection](https://web-platform-dx.github.io/web-features-explorer/features/selection/) | [自定义高亮](https://web-platform-dx.github.io/web-features-explorer/features/highlight/) | [resize（CSS 属性）](https://web-platform-dx.github.io/web-features-explorer/features/resize/) |
| [:dir()](https://web-platform-dx.github.io/web-features-explorer/features/dir-pseudo/) | [自定义属性](https://web-platform-dx.github.io/web-features-explorer/features/custom-properties/) | [revert](https://web-platform-dx.github.io/web-features-explorer/features/revert-value/) |
| [:focus-visible](https://web-platform-dx.github.io/web-features-explorer/features/focus-visible/) | [display 动画](https://web-platform-dx.github.io/web-features-explorer/features/display-animation/) | [revert-rule](https://web-platform-dx.github.io/web-features-explorer/features/revert-rule/) |
| [:focus-within](https://web-platform-dx.github.io/web-features-explorer/features/focus-within/) | [元素作用域视图过渡](https://web-platform-dx.github.io/web-features-explorer/features/view-transitions-element-scoped/) | [安全区域插入环境变量](https://web-platform-dx.github.io/web-features-explorer/features/safe-area-inset/) |
| [:has()](https://web-platform-dx.github.io/web-features-explorer/features/has/) | [em 单位](https://web-platform-dx.github.io/web-features-explorer/features/em-unit/) | [滚动标记目标伪类](https://web-platform-dx.github.io/web-features-explorer/features/scroll-marker-targets/) |
| [:host-context()](https://web-platform-dx.github.io/web-features-explorer/features/host-context/) | [field-sizing](https://web-platform-dx.github.io/web-features-explorer/features/field-sizing/) | [滚动方法 Promise](https://web-platform-dx.github.io/web-features-explorer/features/scroll-promises/) |
| [:is()](https://web-platform-dx.github.io/web-features-explorer/features/is/) | [filter](https://web-platform-dx.github.io/web-features-explorer/features/filter/) | [滚动吸附](https://web-platform-dx.github.io/web-features-explorer/features/scroll-snap/) |
| [:modal](https://web-platform-dx.github.io/web-features-explorer/features/modal/) | [flex-wrap: balance](https://web-platform-dx.github.io/web-features-explorer/features/flexbox-flex-wrap-balance/) | [滚动吸附事件](https://web-platform-dx.github.io/web-features-explorer/features/scroll-snap-events/) |
| [:not()](https://web-platform-dx.github.io/web-features-explorer/features/not/) | [Flexbox](https://web-platform-dx.github.io/web-features-explorer/features/flexbox/) | [scroll-axis-lock](https://web-platform-dx.github.io/web-features-explorer/features/scroll-axis-lock/) |
| [:nth-child() of &lt;selector>](https://web-platform-dx.github.io/web-features-explorer/features/nth-child-of/) | [font-size](https://web-platform-dx.github.io/web-features-explorer/features/font-size/) | [scroll-behavior](https://web-platform-dx.github.io/web-features-explorer/features/scroll-behavior/) |
| [:placeholder-shown](https://web-platform-dx.github.io/web-features-explorer/features/placeholder-shown/) | [font-size-adjust](https://web-platform-dx.github.io/web-features-explorer/features/font-size-adjust/) | [滚动驱动动画](https://web-platform-dx.github.io/web-features-explorer/features/scroll-driven-animations/) |
| [:user-valid 和 :user-invalid](https://web-platform-dx.github.io/web-features-explorer/features/user-pseudos/) | [强制颜色](https://web-platform-dx.github.io/web-features-explorer/features/forced-colors/) | [scroll-initial-target](https://web-platform-dx.github.io/web-features-explorer/features/scroll-initial-target/) |
| [:where()](https://web-platform-dx.github.io/web-features-explorer/features/where/) | [渐变插值](https://web-platform-dx.github.io/web-features-explorer/features/gradient-interpolation/) | [scroll-target-group](https://web-platform-dx.github.io/web-features-explorer/features/scroll-target-group/) |
| [@function](https://web-platform-dx.github.io/web-features-explorer/features/function/) | [渐变](https://web-platform-dx.github.io/web-features-explorer/features/gradients/) | [scrollbar-color](https://web-platform-dx.github.io/web-features-explorer/features/scrollbar-color/) |
| [@scope](https://web-platform-dx.github.io/web-features-explorer/features/scope/) | [Grid](https://web-platform-dx.github.io/web-features-explorer/features/grid/) | [scrollbar-gutter](https://web-platform-dx.github.io/web-features-explorer/features/scrollbar-gutter/) |
| [@starting-style](https://web-platform-dx.github.io/web-features-explorer/features/starting-style/) | [Grid lanes](https://web-platform-dx.github.io/web-features-explorer/features/grid-lanes/) | [scrollbar-width](https://web-platform-dx.github.io/web-features-explorer/features/scrollbar-width/) |
| [@supports](https://web-platform-dx.github.io/web-features-explorer/features/supports/) | [断字](https://web-platform-dx.github.io/web-features-explorer/features/hyphens/) | [scrollend](https://web-platform-dx.github.io/web-features-explorer/features/scrollend/) |
| [abs() 和 sign()](https://web-platform-dx.github.io/web-features-explorer/features/abs-sign/) | [image-set()](https://web-platform-dx.github.io/web-features-explorer/features/image-set/) | [scrollIntoView()](https://web-platform-dx.github.io/web-features-explorer/features/scroll-into-view/) |
| [accent-color](https://web-platform-dx.github.io/web-features-explorer/features/accent-color/) | [独立变换属性](https://web-platform-dx.github.io/web-features-explorer/features/individual-transforms/) | [Shadow parts](https://web-platform-dx.github.io/web-features-explorer/features/shadow-parts/) |
| [活动视图过渡](https://web-platform-dx.github.io/web-features-explorer/features/active-view-transition/) | [inherit](https://web-platform-dx.github.io/web-features-explorer/features/inherit-value/) | [sibling-count() 和 sibling-index()](https://web-platform-dx.github.io/web-features-explorer/features/sibling-count/) |
| [生成内容的替代文本](https://web-platform-dx.github.io/web-features-explorer/features/alt-text-generated-content/) | [initial](https://web-platform-dx.github.io/web-features-explorer/features/initial-value/) | [粘性定位](https://web-platform-dx.github.io/web-features-explorer/features/sticky-positioning/) |
| [锚点定位容器查询](https://web-platform-dx.github.io/web-features-explorer/features/container-anchor-position-queries/) | [交互媒体查询](https://web-platform-dx.github.io/web-features-explorer/features/interaction/) | [Subgrid](https://web-platform-dx.github.io/web-features-explorer/features/subgrid/) |
| [锚点定位](https://web-platform-dx.github.io/web-features-explorer/features/anchor-positioning/) | [interpolate-size](https://web-platform-dx.github.io/web-features-explorer/features/interpolate-size/) | [系统颜色](https://web-platform-dx.github.io/web-features-explorer/features/system-color/) |
| [动画（CSS）](https://web-platform-dx.github.io/web-features-explorer/features/animations-css/) | [isolation](https://web-platform-dx.github.io/web-features-explorer/features/isolation/) | [文本溢出](https://web-platform-dx.github.io/web-features-explorer/features/text-overflow/) |
| [appearance](https://web-platform-dx.github.io/web-features-explorer/features/appearance/) | [Lab 和 LCH](https://web-platform-dx.github.io/web-features-explorer/features/lab/) | [文本描边和填充（兼容性前缀）](https://web-platform-dx.github.io/web-features-explorer/features/text-stroke-fill/) |
| [aspect-ratio](https://web-platform-dx.github.io/web-features-explorer/features/aspect-ratio/) | [light-dark()](https://web-platform-dx.github.io/web-features-explorer/features/light-dark/) | [text-box](https://web-platform-dx.github.io/web-features-explorer/features/text-box/) |
| [at-rule()](https://web-platform-dx.github.io/web-features-explorer/features/supports-at-rule/) | [line-clamp](https://web-platform-dx.github.io/web-features-explorer/features/line-clamp/) | [text-decoration](https://web-platform-dx.github.io/web-features-explorer/features/text-decoration/) |
| [attr()](https://web-platform-dx.github.io/web-features-explorer/features/attr/) | [line-height](https://web-platform-dx.github.io/web-features-explorer/features/line-height/) | [text-decoration-skip-ink](https://web-platform-dx.github.io/web-features-explorer/features/text-decoration-skip-ink/) |
| [background-blend-mode](https://web-platform-dx.github.io/web-features-explorer/features/background-blend-mode/) | [linear() 缓动](https://web-platform-dx.github.io/web-features-explorer/features/linear-easing/) | [text-decoration-skip