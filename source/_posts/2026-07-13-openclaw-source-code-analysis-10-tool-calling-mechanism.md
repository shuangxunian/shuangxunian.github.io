---
title: OpenClaw 源码解析 ⑩：工具调用的底层机制——不是靠 Prompt
date: 2026-07-13 11:28:00
categories:
- AI技术
tags:
  - OpenClaw
  - Tool Calling
  - Function Calling
  - LLM
  - API Protocol
excerpt: LLM 调用工具真的是靠 prompt 格式约定吗？不是——Function Calling 是 API 协议层面的硬约束，从模型训练到推理阶段的全链路拆解。
---

# 工具调用：不是靠 prompt，是靠模型训练 + API 协议

## 三种实现方式（历史演进）

```
┌──────────────────────────────────────────────────────────────────────┐
│ 方式 1：纯 Prompt 模拟（2023年以前的做法，已过时）                    │
│                                                                        │
│ System Prompt 里写：                                                  │
│   "当你需要调用工具时，用以下格式输出：                                │
│    <tool_call>{"name":"exec","arguments":{"command":"ls"}}</tool_call> │
│    然后等待结果..."                                                   │
│                                                                        │
│ 问题：                                                                │
│   - LLM 可能格式错误（漏引号、多余文本）                              │
│   - 不确定什么时候该停止生成                                          │
│   - 不可靠，需要大量解析+重试                                         │
│                                                                        │
│ OpenClaw 不用这种方式                                                 │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ 方式 2：API 原生 Function Calling（当前主流，OpenClaw 使用这种）      │
│                                                                        │
│ 在请求的 tools 字段里传入工具定义                                     │
│ 模型在训练阶段就学会了：                                              │
│   "当我判断需要调用工具时，                                            │
│    输出特殊的结构化 token 序列，                                      │
│    由 API 服务端解析成 tool_calls 对象"                              │
│                                                                        │
│ DeepSeek/OpenAI/Anthropic 都支持                                     │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ 方式 3：模型内嵌格式（如 DeepSeek / Qwen 的某些开源版本）            │
│                                                                        │
│ 模型在 fine-tune 时被训练成：                                         │
│   看到 tools 定义 → 决定调用时 → 生成特定格式的文本                   │
│   然后 API 服务端把文本解析成 tool_calls                              │
│                                                                        │
│ 本质上和方式 2 一样，只是底层实现细节不同                            │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 方式 2 的底层机制（DeepSeek/OpenAI 真正在做的事）

### 训练阶段：模型学会了「什么时候该调工具」

```
训练数据里有大量这样的样本：

样本 1：
 输入: [system prompt] + [tools: read, exec] + [user: "看看这个文件"]
 标注: → 输出 tool_call: read(file_path="xxx")

样本 2：
 输入: [system prompt] + [tools: read, exec] + [user: "今天天气怎么样"]
 标注: → 输出纯文本 "我没有天气查询工具..."

样本 3：
 输入: [system prompt] + [tools: read, exec] + [user: "运行测试"]
 标注: → 输出 tool_call: exec(command="npm test")

经过百万级样本训练后，模型学会了：
 ✓ 根据用户意图判断是否需要调用工具
 ✓ 选择正确的工具
 ✓ 按照 JSON Schema 构造正确的参数
 ✓ 在需要时调用多个工具
```

### 推理阶段：API 服务端做的事

```
┌──────────────────────────────────────────────────────────────────┐
│ DeepSeek API 服务端                                                │
│                                                                     │
│ 1. 收到请求：                                                      │
│    { messages: [...], tools: [...] }                              │
│                                                                     │
│ 2. 把 tools 定义「编码」成模型能理解的 token 序列                   │
│    （不是直接塞进 prompt，而是用特殊的编码方式）                     │
│    每个 provider 的内部编码不同，但效果类似                         │
│                                                                     │
│ 3. 模型开始 token-by-token 生成：                                  │
│                                                                     │
│ 情况 A：模型决定调工具                                              │
│ ┌──────────────────────────────────────────────────────┐          │
│ │ 模型输出特殊 token 序列（内部格式，对外不可见）：    │          │
│ │                                                      │          │
│ │   <|tool_call_start|>                                │          │
│ │   {"name":"exec","arguments":{"command":"npm test"}} │          │
│ │   <|tool_call_end|>                                  │          │
│ │                                                      │          │
│ │ ↓ API 服务端拦截这些特殊 token                       │          │
│ │ ↓ 解析成结构化 JSON                                  │          │
│ │ ↓ 返回给客户端：                                     │          │
│ │                                                      │          │
│ │   finish_reason: "tool_calls"                        │          │
│ │   message.tool_calls: [{                             │          │
│ │     id: "call_abc",                                  │          │
│ │     function: { name: "exec", arguments: "..." }     │          │
│ │   }]                                                 │          │
│ └──────────────────────────────────────────────────────┘          │
│                                                                     │
│ 情况 B：模型决定直接回复文本                                        │
│ ┌──────────────────────────────────────────────────────┐          │
│ │ 模型正常输出文本 token：                             │          │
│ │   "这个文件是一个 React 组件..."                     │          │
│ │                                                      │          │
│ │ ↓ API 服务端正常返回文本                             │          │
│ │                                                      │          │
│ │   finish_reason: "stop"                              │          │
│ │   message.content: "这个文件是一个 React 组件..."    │          │
│ └──────────────────────────────────────────────────────┘          │
│                                                                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## 实际的 API 响应结构

当 LLM **决定调工具**时，返回的 SSE stream 长这样：

```json
data: {
 "choices": [{
 "delta": {
 "role": "assistant",
 "content": null,
 "tool_calls": [{
 "index": 0,
 "id": "call_abc123",
 "type": "function",
 "function": {
 "name": "read",
 "arguments": "{\"file_path\":\"src/main.ts\"}"
 }
 }]
 },
 "finish_reason": "tool_calls"
 }]
}
```

当 LLM **决定说话**时，返回的是：

```json
data: {
 "choices": [{
 "delta": {
 "role": "assistant",
 "content": "这个文件是一个React组件...",
 "tool_calls": null
 },
 "finish_reason": "stop"
 }]
}
```

**这两种输出是在 API 协议层面分开的，不是靠 prompt 解析出来的。**

---

## 关键区别：prompt 控制 vs 协议控制

| | Prompt 控制 | API 协议控制 (Function Calling) |
|--|--|--|
| **格式保证** | LLM 可能输出错误格式 | API 层面保证结构正确 |
| **停止时机** | LLM 可能继续生成垃圾 | 遇到 tool_call token → 立即停止文本生成 |
| **参数验证** | 需要应用层校验 | 部分 provider 支持 `strict: true`（保证参数符合 schema） |
| **训练支持** | 通用能力，碰运气 | 专门针对 function calling 做了 RLHF |
| **性能** | 浪费 token 在格式化上 | 特殊 token 编码，更高效 |

---

## 对照 OpenClaw 代码里的体现

```typescript
// openai-transport-stream.ts:1707-1714
// 发请求时把工具定义放在 tools 字段（协议层）
if (context.tools) {
 params.tools = convertTools(context.tools, compat, model);
 // → [{ type: "function", function: { name, description, parameters } }, ...]
}

// openai-transport-stream.ts:1142-1200
// 收到 SSE 流时处理 tool_calls
async function processOpenAICompletionsStream(responseStream, output, model, stream) {
 for await (const chunk of responseStream) {
 const choice = chunk.choices?.[0];

 if (choice.delta?.tool_calls) {
 // API 返回了结构化的 tool_calls → 不是文本！
 // 直接解析成 { name, arguments } 对象
 handleToolCallChunk(choice.delta.tool_calls);
 }

 if (choice.delta?.content) {
 // 普通文本 token
 handleTextDelta(choice.delta.content);
 }

 if (choice.finish_reason === "tool_calls") {
 // 模型决定调用工具 → 停止生成
 output.stopReason = "tool_use";
 }
 }
}
```

---

## 所以 OpenClaw 里的执行逻辑

```
pi-coding-agent 库内部：

1. 调 LLM API
2. 解析响应：
 if (response.finish_reason === "tool_calls") {
   // 模型决定调工具 —— 这是 API 层面确定的，不需要猜
   for (const toolCall of response.tool_calls) {
     const tool = registeredTools.find(t => t.name === toolCall.function.name);
     const args = JSON.parse(toolCall.function.arguments);
     const result = await tool.execute(toolCall.id, args);
     // 把结果放回 messages
     messages.push({ role: "tool", tool_call_id: toolCall.id, content: result });
   }
   // 带上 tool_result 再调一次 LLM
   goto step 1;
 } else {
   // 模型决定说话 —— 直接返回文本
   return response.content;
 }
```

**没有任何「解析文本里是否包含工具调用」的步骤。** 工具调用和文本回复是 API 响应里完全分离的两个字段。

---

## 补充说明

某些开源模型（如早期 Hermes、部分 Qwen 版本）确实用 prompt 模拟工具调用，靠特定文本格式让应用层解析——也就是前面说的方式 1。但 DeepSeek API / OpenAI / Anthropic 这些商业 API 都已经把 function calling 内置到了协议层，不需要也不应该用 prompt 来模拟。

## 总结

> LLM 不是靠「请按以下格式输出」的 prompt 来调用工具的。它是在训练阶段就学会了「看到 tools 定义 → 判断需要时生成特殊 token → API 服务端解析成结构化输出」这个机制。

System Prompt 里列工具只是给 LLM 额外的自然语言上下文（什么时候该用什么工具），但真正的「能不能调」是由 API 请求体里的 `tools` 字段决定的——这是协议层面的硬约束，不是 prompt 工程。
