---
name: agentic-javascript-tools
description: 使用 WebMCP 命令式 API，将客户端 JavaScript 函数以编程方式注册为 AI 智能体的工具。
web-feature-ids:
  - document-modelcontext
---

# 智能体 JavaScript 工具

命令式 API 使用 `document.modelContext.registerTool()` 以编程方式定义 JavaScript 工具。这非常适合单页应用（SPA），因为这类应用需要根据当前路由或用户状态添加或移除工具。

WebMCP 采用生产者/消费者架构：
* **网页是工具生产者：** 你使用 `document.modelContext.registerTool()` 定义并注册工具。网页只需实现这些。**你无需在页面上调用 `getTools()` 或 `executeTool()` 来向智能体暴露工具。**
* **浏览器和扩展智能体是主要消费者：** 由浏览器提供或通过浏览器提供的智能体——包括浏览器内置智能体、浏览器扩展、DevTools 和系统级助手——通过内部平台机制带外发现并调用已注册的工具。
* **页内智能体和测试工具：** `getTools()` 和 `executeTool()` 专门用于页内 JavaScript 助手（例如嵌入式聊天组件或 `<iframe>`）以及需要在页面上下文中直接发现和调用工具的自动化测试套件。

## 注册与生命周期

注册工具时需传入工具定义对象和可选的选项对象（`ModelContextRegisterToolOptions`），后者支持两个可选成员：
* `signal`：一个 `AbortSignal`，用于在工具不再需要时注销它。
* `exposedTo`：一个源字符串数组，控制文档树中哪些文档可以跨框架边界发现并执行该工具。

### 使用 `AbortController` 处理生命周期

WebMCP 不提供 `unregisterTool()` 方法。要注销工具，你必须在注册时传入 `AbortSignal`，并在工具不再需要时中止该信号。

```javascript
const controller = new AbortController();

await document.modelContext.registerTool({
  name: "get_user_preferences",
  description: "Retrieves the user's saved preferences.",
  inputSchema: { type: "object", properties: {} },
  execute() {
    const prefs = localStorage.getItem("user_prefs");
    return prefs ? JSON.parse(prefs) : { theme: "light" };
  },
  annotations: { readOnlyHint: true }
}, { signal: controller.signal });

// To unregister the tool (e.g., on component unmount):
controller.abort();
```

中止注册信号只会注销工具——不会取消已在运行的执行。启动长时间运行工作的工具还必须遵守下文所述的每次执行信号，否则注销后会留下孤立请求和过时的 UI 更新。

### 使用 `exposedTo` 控制跨源工具暴露

默认情况下，已注册的工具仅对注册文档和同源框架私有。当你的应用嵌入在 `<iframe>` 中（或嵌入第三方组件）且需要祖先或兄弟框架发现并调用该工具时，使用 `exposedTo` 指定允许的源。

```javascript
const controller = new AbortController();

await document.modelContext.registerTool({
  name: "get_cart_items",
  description: "Retrieves items currently in the user's shopping cart.",
  inputSchema: { type: "object", properties: {} },
  async execute(input, { signal }) {
    return await cartService.getItems({ signal });
  },
  annotations: { readOnlyHint: true },
}, {
  signal: controller.signal,
  exposedTo: ["https://checkout.example.com"],
});
```

* **双向选择握手**：跨源工具发现是双向选择。工具提供方框架通过 `exposedTo` 授权消费者源，消费者框架必须通过 `getTools({ fromOrigins: [...] })` 显式请求该提供方源。没有 `fromOrigins` 时，`getTools()` 仅查询同源文档。
* **源匹配**：注册文档自身的源始终有访问权限。`exposedTo` 中指定的源被授予权限，可通过 `document.modelContext.getTools({ fromOrigins: [...] })` 发现工具，并通过 `document.modelContext.executeTool()` 执行它。
* **安全要求**：`exposedTo` 中的每个源必须潜在可信（HTTPS 或 localhost）。传入无效或非安全的源会导致 `registerTool()` 以 `SecurityError` 拒绝。
* **范围**：`exposedTo` 控制文档树中跨框架的页内跨文档发现。

## 定义参数

参数（params）使用 `inputSchema` 属性定义。它必须是一个 **JSON Schema** 对象，描述工具期望的结构化数据。

```javascript
await document.modelContext.registerTool({
  name: "calculate_area",
  description: "Calculates the area of a rectangle.",
  inputSchema: {
    type: "object",
    properties: {
      width: { type: "number", description: "The width of the rectangle." },
      height: { type: "number", description: "The height of the rectangle." }
    },
    required: ["width", "height"]
  },
  execute(input) {
    // input is { width: 10, height: 20 }
    return input.width * input.height;
  },
  annotations: { readOnlyHint: true }
});
```

## 编写工具和参数描述

智能体已收到完整的 `inputSchema`，因此 `description` 只需说明工具做什么以及何时使用。**不要**在其中重复参数名、类型、`enum` 值、默认值或限制；改为在 Schema 中声明（`enum`、`default`、`minimum`/`maximum`）。参数描述遵循同样规则：说明值的含义，而非其类型或允许值。

浏览器不会根据 `inputSchema` 校验参数，也不会应用 `default` 值，因此这两件事都要在 `execute` 中完成。

```javascript
const SORT_ORDERS = ["newest", "price_low_to_high"];

await document.modelContext.registerTool({
  name: "search_products",
  // Don't list the sort orders here, even via SORT_ORDERS: `enum` already does.
  description: "Searches the product catalog by keyword. Use when the user wants to find or compare products.",
  inputSchema: {
    type: "object",
    properties: {
      query: { type: "string", description: "Words to search for." },
      sort: { type: "string", enum: SORT_ORDERS, default: "newest", description: "Order of the results." },
    },
    required: ["query"],
  },
  async execute({ query, sort = "newest" }, { signal }) {
    if (typeof query !== "string" || !SORT_ORDERS.includes(sort)) {
      return { error: "Pass a search query and a supported sort order.", sortOrders: SORT_ORDERS };
    }
    return await productService.search({ query, sort }, { signal });
  },
  annotations: { readOnlyHint: true },
});
```

## 使用 `outputSchema` 声明返回值

自 Chrome 159 起，工具定义接受可选的 `outputSchema`：一个描述 `execute` 返回值的 JSON Schema 对象。

* **意图与结构分离**：让 `description` 专注于工具做什么以及何时调用。**不要**在 `description` 中列出返回的属性名、类型或链式 ID；改为在 `outputSchema` 中声明。
* **有选择地使用 `outputSchema`**：为返回多个字段的整合读取或搜索工具、返回更新状态的变更操作（例如购物车更新后的 `{ items, subtotal, total }`），以及返回供其他工具使用的 ID 的工具添加它。对于输出显而易见的窄读取（例如单个数字）、仅返回状态或确认的操作以及 UI 导航工具，省略它，因为每个 Schema 都会消耗提示词 token。
* **保持 `execute()` 与 `outputSchema` 一致**：返回与 Schema 匹配的 JSON 可序列化值。智能体可以根据 `outputSchema` 校验结果，不匹配时向模型发送工具错误而非结果。这包括结构化错误载荷：如果 `execute` 可能返回 `{ error: "..." }`，请在 `outputSchema` 中允许该形状（例如使用 `anyOf`），否则智能体可能永远看不到你的错误消息。

```javascript
await document.modelContext.registerTool({
  name: "search_flights",
  // Say what the tool does and when to use it; the result shape lives in `outputSchema`.
  description: "Searches available flights between two airports. Use before booking a flight.",
  inputSchema: {
    type: "object",
    properties: {
      from: { type: "string", description: "Departure airport IATA code." },
      to: { type: "string", description: "Arrival airport IATA code." },
    },
    required: ["from", "to"],
  },
  outputSchema: {
    anyOf: [
      {
        type: "object",
        properties: {
          flights: {
            type: "array",
            items: {
              type: "object",
              properties: {
                // Declared here so the agent knows to pass it to `book_flight`.
                flightId: { type: "string", description: "Pass to book_flight." },
                departureTime: { type: "string", format: "date-time" },
                price: { type: "number" },
              },
              required: ["flightId", "departureTime", "price"],
            },
          },
        },
        required: ["flights"],
      },
      // Allow the structured error payload so validation doesn't swallow it.
      {
        type: "object",
        properties: { error: { type: "string" } },
        required: ["error"],
      },
    ],
  },
  async execute({ from, to }, { signal }) {
    if (typeof from !== "string" || typeof to !== "string") {
      return { error: "Pass departure and arrival airport codes." };
    }
    try {
      const flights = await flightService.search({ from, to }, { signal });
      // Return only the declared fields, matching `outputSchema`.
      return {
        flights: flights.map(({ id, departureTime, price }) => ({ flightId: id, departureTime, price })),
      };
    } catch (err) {
      // A cancelled execution is re-thrown, not returned as an error value.
      if (signal.aborted) throw err;
      // Matches the `{ error }` branch of `outputSchema`, so the agent sees the message.
      return { error: err instanceof Error ? err.message : String(err) };
    }
  },
  annotations: { readOnlyHint: true },
});
```

## 执行模式

`execute` 回调始终接收两个参数：输入对象和携带 `AbortSignal` 的选项对象——`execute(inputObject, { signal })`。

### 何时使用 `async execute`
当工具涉及返回 Promise 或需要时间完成的操作时，使用 `async`：
- **网络调用**：从 API 获取数据。
- **异步存储**：访问 IndexedDB。
- **外部事件**：等待特定状态变化或动画完成。

```javascript
async execute(input, { signal }) {
  const response = await fetch(`/api/data/${input.id}`, { signal });
  return await response.json();
}
```

### 何时使用 `execute`（同步）
对于即时操作，使用标准同步函数。同步工具会运行至完成，期间无法观察到取消，因此可以忽略信号：
- **纯逻辑**：对已在内存中的数据进行数学运算、过滤或排序。
- **同步状态**：从 `localStorage` 或同步状态管理器读取。

```javascript
execute(input) {
  return input.items.filter(item => item.active);
}
```

### 在 `execute` 中处理错误

当 `execute` 抛出异常（或返回被拒绝的 Promise）时，智能体只会收到通用的执行失败。错误消息不会转发给智能体。如果你的工具遇到可操作或可恢复的错误（例如无效输入、缺少项目或 HTTP 状态失败），**不要** `throw new Error(...)`。改为通过工具的正常返回值返回结构化错误载荷，让 AI 智能体收到诊断细节并自行纠正或重试：

```javascript
execute({ itemName }) {
  const item = menuItems.find((entry) => entry.name.toLowerCase() === itemName.toLowerCase());
  if (!item) {
    // Return the error as a value instead of throwing, so the agent receives the message:
    return {
      error: `Menu item "${itemName}" not found.`,
      availableItems: menuItems.map((entry) => entry.name),
    };
  }
  return item;
}
```

仅在执行被取消（`signal.aborted` 为 `true`）时重新抛出。

### 取消进行中的执行

取消由用户或智能体发起。**强制要求：** 任何执行网络请求或其他长时间运行异步工作的工具都必须遵守 `signal`，否则被取消的执行会继续消耗资源，并可能在智能体已继续前进后向 UI 写入过时数据。

* **接受信号的 API**（`fetch()`、`addEventListener()`、`ReadableStream` 读取器）：直接传入 `signal`。
* **不接受信号的 API**（`setTimeout`、`setInterval`、轮询循环、大量 `await` 的流水线）：在步骤之间调用 `signal.throwIfAborted()`，和/或在 `signal.addEventListener('abort', ...)` 时拒绝。定时器不接受信号，因此你必须在中止监听器中自行清除它们。

```javascript
await document.modelContext.registerTool({
  name: "fetch_tool",
  description: "Fetch the text content of a URL and show it on the page.",
  inputSchema: {
    type: "object",
    properties: {
      url: { type: "string", description: "The URL to fetch" },
      priority: {
        type: "string",
        enum: ["high", "low", "auto"],
        description: "Relative network priority for the request",
      },
    },
    required: ["url"],
  },
  execute: async ({ url, priority }, { signal }) => {
    const output = document.querySelector("pre");
    output.textContent = "Loading…";
    try {
      // Passing `signal` aborts the network request the moment execution is cancelled.
      const response = await fetch(url, { priority, signal });
      if (!response.ok) {
        output.textContent = "";
        return { error: `Request failed with status ${response.status}` };
      }
      const text = await response.text();
      output.textContent = text;
      return text;
    } catch (err) {
      // Roll back the UI state this execution set, so a cancelled run leaves no trace.
      output.textContent = "";
      if (signal.aborted) {
        // The execution was cancelled: re-throw instead of returning an error value.
        throw err;
      }
      // Return other errors as a value instead of throwing, so the agent receives the message.
      return { error: err instanceof Error ? err.message : String(err) };
    }
  },
  annotations: { readOnlyHint: true }
});
```

## 标记重大影响工具

自 Chrome 154 起，`annotations` 支持 `consequentialHint`。对于执行高风险、不可逆或现实世界操作的工具——预订旅行、转移资金、发送消息或删除数据——将其设为 `true`。它提示智能体在执行前请求用户明确确认。

```javascript
await document.modelContext.registerTool({
  name: 'book_flight',
  description: 'Book a flight for the user with confirmed flight details.',
  inputSchema: {
    type: 'object',
    properties: {
      flightId: { type: 'string', description: 'ID of the flight to book' },
      passengers: { type: 'number', description: 'Number of tickets to purchase' },
    },
    required: ['flightId', 'passengers'],
  },
  execute: async ({ flightId, passengers }, { signal }) => {
    // Forward `signal` to the transaction API to honor cancellation:
    return await bookFlightReservation({ flightId, passengers }, { signal });
  },
  annotations: {
    readOnlyHint: false,      // This tool mutates state (it makes a booking).
    consequentialHint: true,  // Spends money and is not trivially reversible.
    untrustedContentHint: false, // Output is first-party data, not third-party content.
  }
});
```

**强制要求：** 将 `consequentialHint` 视为提示，而非强制机制。它本身不会阻止执行，且不支持该提示的旧版浏览器完全不会弹出提示。保留该操作原本应有的应用内确认。

## 标记调试工具

自 Chrome 156 起，`annotations` 支持 `debugging`。对于为调试和开发者工具而非最终用户任务构建的工具，将 `annotations.debugging` 设为 `true`。例如：转储内部应用状态、切换功能标志或填充测试数据。这让智能体能区分仅开发者工具和代表用户操作的工具。默认为 `false`。页内智能体和测试工具可通过 `getTools()` 从 `RegisteredTool.annotations.debugging` 读取它。

```javascript
await document.modelContext.registerTool({
  name: "dump_app_state",
  description: "Returns a snapshot of the internal application store for debugging.",
  execute() {
    return appStore.getState();
  },
  annotations: {
    readOnlyHint: true, // Only reads state.
    debugging: true,    // Developer tooling, not an end-user action.
  }
});
```

## 页内智能体与测试（`getTools` 和 `executeTool`）

**网页无需 `getTools()` 或 `executeTool()` 即可向 AI 智能体暴露工具。** 只需调用 `document.modelContext.registerTool()`，浏览器智能体、浏览器扩展、DevTools 和系统级助手即可带外发现并执行你的工具。

`getTools()` 和 `executeTool()` 方法专门供在 JavaScript 环境中运行的消费者使用：
1. **页内 JavaScript 智能体：** 直接嵌入页面（或在 `<iframe>` 内运行）的 AI 助手、copilot 组件或聊天界面，需要发现周围页面或文档树中可访问框架提供的工具，并代表用户调用它们。
2. **自动化测试与调试：** 测试套件（如 Playwright、Vitest 或浏览器内单元测试）检查已注册工具的元数据并模拟智能体工具调用，无需实时外部 LLM 工具链。

### 使用 `getTools()` 发现可用工具

页内智能体调用 `document.modelContext.getTools()`，从当前文档和文档树中任何可访问框架获取 `RegisteredTool` 对象数组。

自 Chrome 155 起，`RegisteredTool.inputSchema` 直接作为 JavaScript 对象返回（注册时提供的 Schema 的深拷贝）。修改此对象不会改变已注册工具的实际 Schema。

```javascript
// Example: An in-page AI assistant discovering tools
```