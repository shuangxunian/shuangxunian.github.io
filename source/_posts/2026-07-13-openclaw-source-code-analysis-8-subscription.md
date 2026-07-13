---
title: OpenClaw 源码解析 ⑧：Subscription 管道完整拆解
date: 2026-07-13 11:08:00
categories:
  - OpenClaw
  - 源码解析
tags:
  - OpenClaw
  - Subscription
  - Event Pipeline
  - WebSocket
  - LLM
excerpt: 从 LLM SSE 到 WebUI 屏幕上的每一个字——Subscription 管道如何通过事件链实现打字机效果、工具调用追踪、思考过程流式推送的完整拆解。
---

# Subscription 管道：完整拆解

## 全局数据流

```
LLM API (DeepSeek/OpenAI/Anthropic)
 │
 │ SSE stream / WebSocket frames
 ▼
@mariozechner/pi-ai library (transport层)
 │
 │ 解析 SSE → 生成标准化事件
 ▼
pi-coding-agent library (session层)
 │
 │ 管理 messages 数组状态 → 发出 AgentEvent
 ▼
session.subscribe(handler) ← 注册回调
 │
 │ AgentEvent { type, message, ... }
 ▼
createEmbeddedPiSessionEventHandler() ← 事件调度器
 │
 ├── message_start → handleMessageStart()
 ├── message_update → handleMessageUpdate() ← 每个 token
 ├── message_end → handleMessageEnd()
 ├── tool_execution_start → handleToolExecutionStart()
 ├── tool_execution_update → handleToolExecutionUpdate()
 ├── tool_execution_end → handleToolExecutionEnd()
 ├── compaction_start → handleCompactionStart()
 ├── compaction_end → handleCompactionEnd()
 ├── agent_start → handleAgentStart()
 └── agent_end → handleAgentEnd()
 │
 │ handler 内部调用:
 │ emitAgentEvent({ runId, stream, data })
 │ ctx.params.onPartialReply(text)
 │ ctx.params.onBlockReply(payload)
 │ ctx.params.onToolResult(result)
 ▼
infra/agent-events.ts → notifyListeners()
 │
 │ 全局事件总线
 ▼
gateway/server-runtime-subscriptions.ts
 │
 │ onAgentEvent → createAgentEventHandler()
 ▼
gateway/server-broadcast.ts → broadcastInternal()
 │
 │ for (client of connectedClients)
 │ JSON.stringify(frame) → client.socket.send(frame)
 ▼
WebUI / 其他 WebSocket 客户端
```

---

## 核心设计：`pendingEventChain` — 保序执行

```typescript
// pi-embedded-subscribe.handlers.ts:23-73

export function createEmbeddedPiSessionEventHandler(ctx) {
 let pendingEventChain: Promise<void> | null = null;

 const scheduleEvent = (evt, handler, options?) => {
 const run = () => handler();

 // 情况 1：当前没有排队的事件 → 直接执行
 if (!pendingEventChain) {
 const result = run();
 if (!isPromiseLike(result)) {
 return; // 同步完成 → 结束
 }
 // 异步操作 → 变成链的起始点
 pendingEventChain = result.finally(() => { pendingEventChain = null; });
 return;
 }

 // 情况 2：有事件在排队 → 链到后面
 pendingEventChain = pendingEventChain
 .then(() => run())
 .finally(() => { pendingEventChain = null; });
 };
}
```

**为什么需要保序？**

```
如果不保序：
 LLM 先返回 text_delta "你好"
 LLM 紧接着返回 tool_call exec("ls")
 如果两者并发处理 → 可能 exec 先跑完再显示 "你好" → 输出乱序

保序后：
 pendingEventChain:
 → handleMessageUpdate("你好") // 先完成
 → handleToolExecutionStart() // 再执行
 → handleToolExecutionEnd() // 等工具完成
 → handleMessageUpdate("结果...") // 最后继续
```

**但有一个例外**：`tool_execution_end` 用了 `{ detach: true }`

```typescript
case "tool_execution_end":
 scheduleEvent(evt, () => handleToolExecutionEnd(ctx, evt), { detach: true });
 // ^^^^^^^^^^^^^
 // detach=true → 不阻塞后续事件
 // 因为 tool_execution_end 里可能有 hook 回调（如 after_tool_call），
 // 它们的结果不影响后续 LLM 响应的处理
```

---

## 事件类型详解

### message_start — 助手开始说话

```typescript
export function handleMessageStart(ctx, evt) {
 // 重置累积状态
 ctx.resetAssistantMessageState(ctx.state.assistantTexts.length);

 // 通知 "typing..." 状态
 ctx.params.onAssistantMessageStart?.();
}
```

**触发频率**：每轮 LLM 回复开始时一次。

---

### message_update — 逐 token 流式处理（最核心！）

```typescript
export function handleMessageUpdate(ctx, evt) {
 const msg = evt.message;
 // 过滤非 assistant 消息
 if (msg.role !== "assistant") return;

 // ========================
 // 分支 1: 思考模式 (<thinking>)
 // ========================
 if (isThinkingEvent(evt)) {
 // 累积 <thinking> 内容
 // 如果 reasoningMode="stream" → 实时推送思考过程
 ctx.emitReasoningStream(thinkingContent);
 return;
 }

 // ========================
 // 分支 2: 正常文本 (text_delta)
 // ========================
 const delta = assistantRecord.delta; // "你"
 const content = assistantRecord.content; // "你好，我是" (累积)

 // 检测 <think>...</think> 标签（某些模型的推理格式）
 const chunk = resolveAssistantChunk(delta, content, ctx.state);

 if (chunk.cleaned) {
 // 有清理后的新文本要显示

 // A) 检测指令 (如 {directive})
 const directive = parseDirective(chunk.cleaned);
 if (directive) {
 ctx.handleDirective(directive);
 return;
 }

 // B) 发送 "partial reply" 给调用者
 // (auto-reply 用这个做 streaming 到 Telegram)
 ctx.params.onPartialReply?.(chunk.cleaned, {
 runId, assistantIndex, fullText
 });

 // C) 发 agent event → gateway → WebUI
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "text",
 data: { delta: chunk.cleaned, content: fullText }
 });

 // D) 检查是否构成一个完整的 "block"
 // (一个段落结束 / 一个代码块结束)
 if (shouldFlushBlock(ctx)) {
 ctx.params.onBlockReply?.(buildBlockPayload());
 }
 }
}
```

**触发频率**：每个 token 一次。Claude 4 打字速度 ~100 tokens/s = 每秒 100 次。

---

### message_end — 助手说完了

```typescript
export function handleMessageEnd(ctx, evt) {
 const msg = evt.message;

 // 最终化文本
 const finalText = coerceChatContentText(msg);

 // 处理 "静默回复"
 if (isSilentReplyText(finalText)) {
 // LLM 回了 "HEARTBEAT_OK" → 不需要向用户发任何东西
 ctx.markSilent();
 return;
 }

 // 解析最终指令
 const directives = parseReplyDirectives(finalText);
 // 例如: {model: "gpt-4o"} 切换模型
 // {tts: true} 语音合成
 // {reaction: "👍"} 发表情

 // 记录最终文本
 ctx.state.assistantTexts.push(finalText);

 // 发送最终 block reply
 ctx.params.onBlockReply?.(buildFinalBlockPayload());

 // 发 agent event
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "text",
 data: { done: true, content: finalText }
 });
}
```

---

### tool_execution_start — LLM 要调工具

```typescript
export function handleToolExecutionStart(ctx, evt) {
 const toolName = evt.toolName; // "exec"
 const toolCallId = evt.toolCallId; // "call_abc123"
 const args = evt.args; // { command: "npm test" }

 // 1. 记录开始时间（用于后续统计耗时）
 toolStartData.set(toolCallId, { startTime: Date.now(), args });

 // 2. 构建 meta 信息（给 WebUI 显示的摘要）
 // 例如: "exec: npm test"
 // "read: src/index.ts"
 const meta = buildToolCallMeta(toolName, args);
 ctx.state.toolMetaById.set(toolCallId, { meta });

 // 3. 发 agent event → WebUI 显示 "正在执行 exec..."
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "item",
 data: {
 type: "tool_start",
 name: toolName,
 args: summarizeToolArgs(args),
 meta,
 }
 });

 // 注意：工具的实际执行不在这里！
 // 工具执行发生在 pi-coding-agent 库内部：
 // session.prompt() → LLM 返回 tool_call
 // → 库自动查找 tool.execute 函数 → 调用
 // → 产生 tool_execution_update 事件 (进度)
 // → 产生 tool_execution_end 事件 (结果)
}
```

---

### tool_execution_update — 工具执行中间状态

```typescript
export function handleToolExecutionUpdate(ctx, evt) {
 // 只有少数工具会发 update（如 exec 的实时 stdout 输出）

 // 发 agent event → WebUI 显示工具的中间输出
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "item",
 data: {
 type: "tool_progress",
 name: evt.toolName,
 partial: evt.partial, // 部分 stdout
 }
 });
}
```

---

### tool_execution_end — 工具执行完毕

```typescript
export async function handleToolExecutionEnd(ctx, evt) {
 const toolName = evt.toolName;
 const result = evt.result; // tool 的返回值 (stdout/stderr/exitCode...)
 const isError = evt.isError;
 const duration = Date.now() - startData.startTime;

 // 1. 运行 after_tool_call hook
 // → 让插件有机会审计/记录工具调用
 await runAfterToolCallHook({ toolName, args, result, duration });

 // 2. 发送 tool result 回调
 // → auto-reply 用这个决定是否需要额外处理
 ctx.params.onToolResult?.({
 toolName, toolCallId, result, isError, duration
 });

 // 3. 发 agent event → WebUI 显示工具结果
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "item",
 data: {
 type: "tool_end",
 name: toolName,
 result: summarizeResult(result),
 isError,
 duration,
 }
 });

 // 工具结果发回 LLM 是自动的：
 // pi-coding-agent 库在 tool.execute() 返回后，
 // 会自动把 tool_result 放入 messages，然后
 // 重新调用 LLM API → 产生新的 message_start/update/end
}
```

---

### compaction_start / compaction_end — 上下文压缩

```typescript
function handleCompactionStart(ctx) {
 ctx.state.isCompacting = true;
 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "lifecycle",
 data: { type: "compaction_start" }
 });
}

function handleCompactionEnd(ctx, evt) {
 ctx.state.isCompacting = false;
 ctx.incrementCompactionCount();

 // 通知等待 compaction 完成的代码
 ctx.resolveCompactionRetry();

 emitAgentEvent({
 runId: ctx.params.runId,
 stream: "lifecycle",
 data: { type: "compaction_end", reason: evt.reason }
 });
}
```

---

## 从 handler 到 WebUI 的完整链路

以"用户在 WebUI 上看到 AI 打字"为例：

```
DeepSeek API → SSE: data: {"choices":[{"delta":{"content":"你"}}]}
 │
 ▼
pi-ai openai-completions transport
 → 解析 SSE → 更新 output.content = "你"
 → 推送事件到 eventStream
 │
 ▼
pi-coding-agent session
 → 更新 messages[last].content = "你"
 → 调用 subscribeCallback({ type: "message_update", message: {...} })
 │
 ▼
createEmbeddedPiSessionEventHandler
 → scheduleEvent("message_update", () => handleMessageUpdate(ctx, evt))
 │
 ▼
handleMessageUpdate()
 → 解析 delta = "你"
 → 清理 <think> 标签（如果有）
 → 调 ctx.params.onPartialReply("你", {...})
 → 调 emitAgentEvent({ runId, stream: "text", data: { delta: "你" } })
 │
 ▼
infra/agent-events.ts → notifyListeners()
 → 遍历所有注册的 listener → callback(enrichedEvent)
 │
 ▼
gateway/server-runtime-subscriptions.ts
 → onAgentEvent(createAgentEventHandler({...}))
 → agentEventHandler(event) → 识别 stream="text"
 → broadcast("agent", event, { dropIfSlow: true })
 │
 ▼
gateway/server-broadcast.ts → broadcastInternal("agent", event)
 → for (client of params.clients)
 │ → 检查 scope (需要 READ_SCOPE)
 │ → 检查 bufferedAmount < MAX_BUFFERED_BYTES
 │ → JSON.stringify({ type: "event", event: "agent", payload: {...} })
 │ → client.socket.send(frame)
 │
 ▼
WebSocket → 网络 → 浏览器 → WebUI React 组件
 → onMessage → dispatch({ type: "AGENT_TEXT_DELTA", delta: "你" })
 → UI re-render → 用户看到 "你" 出现在屏幕上
```

**整条链路延迟**：

- LLM SSE 到事件循环处理：~0ms（已在内存中）
- handler + emitAgentEvent：~0.1ms
- broadcast JSON.stringify + send：~0.5ms（取决于连接数）
- 网络 → 浏览器渲染：~5-50ms
- **总计：~5-50ms（这就是为什么你能感受到"实时打字"）**

---

## 状态管理：`EmbeddedPiSubscribeState`

subscription 内部维护一组可变状态：

```typescript
const state: EmbeddedPiSubscribeState = {
 // === 累积的回复文本 ===
 assistantTexts: [], // 最终完成的助手文本列表

 // === 工具追踪 ===
 toolMetas: [], // 所有执行过的工具 meta
 toolMetaById: new Map(), // 当前进行中的工具 (by callId)
 toolSummaryById: new Set(), // 已发送摘要的工具
 itemActiveIds: new Set(), // 当前活跃的 item
 itemStartedCount: 0, // 启动的 item 数量
 itemCompletedCount: 0, // 完成的 item 数量
 lastToolError: undefined, // 最近一次工具错误

 // === 流式文本处理 ===
 deltaBuffer: "", // 未完成的 delta 缓冲
 blockBuffer: "", // 未完成的 block 缓冲
 blockState: { // <think>/<final> 标签解析状态
 thinking: false,
 final: false,
 inlineCode: {...} // 内联代码块追踪（防误解析标签）
 },

 // === reasoning ===
 reasoningMode: "off", // off | on | stream
 streamReasoning: false, // 是否实时推送思考过程
 reasoningStreamOpen: false, // 是否打开了 reasoning 流

 // === compaction 状态 ===
 isCompacting: false,
 compactionRetryPromise: null,

 // === usage 统计 ===
 usageAccumulator: {
 input: 0,
 output: 0,
 cacheRead: 0,
 cacheWrite: 0,
 totalTokens: 0,
 cost: { input: 0, output: 0, total: 0 }
 },

 // === 生命周期 ===
 unsubscribed: false,
};
```

---

## 返回给调用者的接口

`subscribeEmbeddedPiSession()` 返回一组控制句柄：

```typescript
// attempt.ts:1753
const subscription = subscribeEmbeddedPiSession({
 session: activeSession,
 onPartialReply, // 每收到一个 token 调一次
 onBlockReply, // 每完成一个段落/块调一次
 onToolResult, // 每完成一次工具执行调一次
 onReasoningStream, // 思考过程实时回调
 onAgentEvent, // 所有 agent event 回调
 ...
});

// 返回值：
const {
 assistantTexts, // () => string[] — 已完成的回复列表
 toolMetas, // () => ToolMeta[] — 工具调用记录
 unsubscribe, // () => void — 停止订阅
 waitForCompactionRetry, // () => Promise — 等 compaction 完成
 isCompactionInFlight, // () => boolean — 是否正在 compaction
 getUsageTotals, // () => Usage — token 用量统计
 getCompactionCount, // () => number — compaction 次数
} = subscription;
```

attempt 在 `session.prompt()` 完成后用这些句柄：

```typescript
// prompt 完成后
await session.prompt(userMessage);

// 取出结果
const texts = subscription.assistantTexts; // LLM 最终说了什么
const usage = subscription.getUsageTotals(); // 花了多少 token
const tools = subscription.toolMetas; // 调了哪些工具

// 清理
subscription.unsubscribe();
```

---

## 完整的事件时序（一次带 tool call 的对话）

```
时间轴：

t=0ms 用户发 "帮我看看这个文件"
 └─ session.prompt("帮我看看这个文件")
 └─ HTTP POST → DeepSeek API

t=200ms DeepSeek 开始回复
 ├─ agent_start
 └─ message_start
 └─ handleMessageStart() → onAssistantMessageStart → typing...

t=210ms 收到 delta "好"
 └─ message_update { delta: "好" }
 └─ handleMessageUpdate() → emitAgentEvent → broadcast → WebUI 显示 "好"

t=220ms 收到 delta "的，"
 └─ message_update { delta: "的，" }
 └─ ... → WebUI 显示 "好的，"

t=250ms 收到 delta "让我看看。"
 └─ message_update { delta: "让我看看。" }
 └─ ... → WebUI 显示 "好的，让我看看。"

t=260ms LLM 决定调用 read 工具
 └─ tool_execution_start { toolName: "read", args: { path: "src/main.ts" } }
 └─ handleToolExecutionStart()
 → emitAgentEvent("item", { type: "tool_start", name: "read" })
 → WebUI 显示 "📖 Reading src/main.ts..."

 pi-coding-agent 自动执行 read tool:
 → readTool.execute("src/main.ts")
 → 读取文件内容

t=265ms read 工具完成
 └─ tool_execution_end { toolName: "read", result: "import React from..." }
 └─ handleToolExecutionEnd()
 → emitAgentEvent("item", { type: "tool_end", name: "read" })
 → WebUI 显示 "✅ Read src/main.ts (1.2KB)"
 → onToolResult({ toolName: "read", result: "..." })

 pi-coding-agent 自动把 tool_result 放入 messages
 → 重新调 DeepSeek API

t=500ms DeepSeek 收到 tool result → 开始新回复
 └─ message_start
 └─ handleMessageStart()

t=510ms 收到 delta "这个文件是一个 React 组件..."
 └─ message_update → ... → WebUI 流式显示

t=800ms LLM 回复完成
 └─ message_end { content: "这个文件是一个 React 组件，它..." }
 └─ handleMessageEnd()
 → ctx.state.assistantTexts.push("...")
 → emitAgentEvent("text", { done: true })
 → onBlockReply(finalPayload)

t=800ms agent_end
 └─ handleAgentEnd()

t=800ms session.prompt() 的 Promise resolve
 └─ attempt 收集结果 → 返回给 run.ts 主循环
```

---

## 关键设计总结

| 设计 | 为什么 |
|------|--------|
| **pendingEventChain 保序** | 保证用户看到的输出顺序和 LLM 产生的一致 |
| **tool_execution_end 用 detach** | hook 回调不阻塞后续 LLM 响应处理 |
| **emitAgentEvent 全局总线** | 解耦 Agent 执行和 WebUI 广播 |
| **state 对象可变** | 高频更新（100次/秒），不能每次 immutable copy |
| **dropIfSlow 背压** | 慢客户端不拖累快客户端 |
| **onPartialReply vs onBlockReply** | partial：逐 token，block：段落级 → 不同渠道用不同粒度 |
