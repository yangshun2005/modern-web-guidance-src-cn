# Interest Invokers

Interest Invokers API 提供声明式方式，基于用户"兴趣"（悬停、聚焦或长按）在源元素（如按钮或链接）与目标弹出框元素（如工具提示或悬停卡片）之间建立关系。

## 无障碍

`interestfor` 的规范要求根据目标内容，为辅助技术本身连接提示弹出框（`popover="hint"`）。**纯提示**（仅文本和图片）成为触发器的无障碍描述，若触发器无名称则成为其名称。**富提示**（标题、链接或其他交互内容）至少获得 `tooltip` 角色，提示打开时触发器获得 `aria-expanded` 和 `aria-details` 关系，且打开的提示在 Tab 顺序中紧跟触发器，使键盘用户能访问其内容。

浏览器和屏幕阅读器尚未一致暴露此行为，因此还需显式添加 ARIA：

- **提示重复触发器的无障碍名称**（如仅图标按钮 `aria-label="Bold"` 配"Bold"工具提示）：不添加任何内容。与名称相同的描述会被读两次。
- **触发器自身无可见标签的提示**（如无 `aria-label` 的仅图标按钮）：在触发器上设 `aria-labelledby` 指向提示，使提示成为名称。
- **触发器在激活时也会打开提示**（切换提示，或唯一职责就是提示的按钮：为其设置 `popovertarget` 和 `popovertargetaction="show"`）：提示设 `role="tooltip"`，触发器设 `aria-expanded`，并与弹出框的 `toggle` 事件保持同步。紧接"按钮"后读出的描述暗示有进一步操作，会混淆角色；展开状态告知用户有内容可揭示及是否已打开，按下按钮即可呈现。将提示放在 DOM 中紧接触发器之后，使阅读键在打开后能到达其内容。
- **触发器在激活时执行其他操作的提示**（链接、有自身操作的按钮）：提示设 `role="tooltip"`，触发器设 `aria-describedby`，绝不用 `aria-expanded`。被宣布为"已折叠"的触发器暗示 `Enter` 会展开它，但 `Enter` 会导航或执行操作。提示必须是用户可承受错过的内容：悬停卡片预览目标，操作按钮上的工具提示描述操作。
- **不要**将激活时打开的提示附加到已用 `aria-expanded` 表示其他含义的控件（如菜单按钮）。为该提示提供独立触发器。
- **不要**在显示兴趣时将焦点移入目标（无 `autofocus`，不在 `interest` 监听器中调用 `focus()`）。兴趣仅由悬停或聚焦触发，移动焦点会违反 WCAG 3.2.1（聚焦时）。

并非所有屏幕阅读器在页面导航时都读取描述（NVDA 在浏览模式下按设计跳过），因此作为描述暴露的提示只能携带用户可承受错过的内容。链接预览和操作控件上的工具提示符合条件；用户需要的内容应放在切换提示或内联中。

用简短脚本连接这些属性，而非手写，使标记保持声明式且相同规则适用于每个提示。脚本应：

1. 在所有浏览器中运行，在加载 `interestfor` polyfill 之前，DOM 解析后执行，并暴露可再次调用的函数以处理后续插入的内容。必须预先扫描：浏览模式屏幕阅读器用户在任何交互前就需要属性就位，因此在 `focus` 或 `interest` 时惰性连接太晚，等待 polyfill 也太晚——polyfill 仅在 `load` 时连接其 ARIA，且不为纯提示添加 `role="tooltip"`。
2. 对每个目标为 `popover="hint"` 的 `[interestfor]` 触发器，规范化空白并比较提示文本与触发器的无障碍名称（`aria-labelledby` 目标，否则 `aria-label`，否则含图片 `alt` 的文本内容，否则 `title`）。相等：不操作。否则若提示无角色，设 `role="tooltip"`。
3. 触发器无名称时，将 `aria-labelledby` 指向提示（除非已有）。否则若触发器在激活时也打开提示（`popovertarget` 或 `commandfor` 指向该提示），将 `aria-expanded` 设为弹出框当前状态（`:popover-open`）（除非已有 `aria-expanded`），并记录脚本拥有的触发器；若否，将 `aria-describedby` 指向提示（除非已有）。
4. 在所有浏览器中通过 `document` 上单个捕获监听器，从弹出框的 `toggle` 事件同步 `aria-expanded`，更新脚本拥有的、`interestfor` 匹配的每个触发器（`interestfor` polyfill 不为纯文本提示同步 `aria-expanded`，也不在通过 `popovertarget` 打开切换提示时同步）。

容易出错的地方：

- `toggle` 不冒泡。`document` 上的冒泡监听器永不触发；使用捕获阶段（或每个弹出框一个监听器）。
- 从 `toggle` 同步 `aria-expanded`，而非 `interest`/`loseinterest`。弹出框是真相来源：它可被 `Escape`、轻触关闭或其他调用器关闭，且多个触发器可共享一个提示。
- 隐藏的弹出框是有效的 `aria-describedby` 或 `aria-labelledby` 目标。不要强制打开提示或复制其文本使其"可见"。
- 不要为任何提示添加 `aria-haspopup` 或实时区域。
- 不要覆盖作者 ARIA：触发器上已有的 `aria-describedby`、`aria-labelledby` 或 `aria-expanded` 优先。

## 时序

默认情况下，键盘和鼠标用户的兴趣显示与丢失延迟为半秒。这防止在页面中 Tab 切换或移动鼠标时目标弹出框打开，也防止鼠标从触发器移到弹出框时关闭。要调整时序，使用 `interest-delay` CSS 属性。

```css
[interestfor]{
  /* interest-delay-start 和 interest-delay-end 的简写 */
  interest-delay: 0.2s 0.1s;
}
```

若用户可能快速连续经过多个触发器，且兴趣触发的弹出框已打开，将初始延迟设为 `0s`。为避免 `:has()` 的性能问题，选择子节点较少的附近容器，并尽可能直接定位触发器。

```css
.container:has( > li > a:interest-source) [interestfor] {
  interest-delay-start: 0s;
}
```

## 回退

Interest invokers 必须使用 NPM 的 `interestfor` polyfill 包进行条件性 polyfill。优先打包 polyfill 而非使用 CDN。

**强制：** 通过检查 `HTMLButtonElement.prototype` 上的 `interestForElement` 属性进行特性检测，**仅**在缺少原生支持时加载 polyfill。

```html
<script type="module">
  if (!("interestForElement" in HTMLButtonElement.prototype)) {
    // CDN 链接仅作示例，优先打包。
    await import("https://unpkg.com/interestfor@latest");
  }
</script>
```

### CSS Polyfill 注意事项

使用 polyfill 时，CSS API 略有变化，以弥补缺少原生伪类和属性支持：

- **伪类：** polyfill 无法定义真正的 `:interest-source` 和 `:interest-target` 伪类，因此改用 `.interest-source` 和 `.interest-target` 类。**强制：** 用 `:is()` 组合它们以同时支持原生和 polyfill 状态。

```css
/* 效果预览时应用的样式 */
:is(:interest-source, .interest-source) { /* ... */ }
:is(:interest-target, .interest-target) { /* ... */ }
```

- **CSS 属性：** 对于 `interest-delay`、`interest-delay-start` 和 `interest-delay-end`，polyfill 要求使用 CSS 变量，因为它无法在所有浏览器中直接读取自定义属性。

```css
/* 调整 interest invokers 的开始和结束延迟 */
[interestfor] {
  --interest-delay-start: 0.2s;
  interest-delay-start: var(--interest-delay-start);
  --interest-delay-end: 0.1s;
  interest-delay-end: var(--interest-delay-end);
}
```

若兴趣触发的弹出框已打开且用户可能快速连续经过多个触发器，将初始延迟设为 `0s`。

```css
.container:has([interestfor]:is(:interest-source, .interest-source)) [interestfor] {
  --interest-delay-start: 0s;
}
```