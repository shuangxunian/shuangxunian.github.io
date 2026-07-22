---
title: OpenClaw 源码解析 ⑨：Agent 如何感知工具——System Prompt vs API tools
date: 2026-07-13 11:25:00
categories:
- AI技术
tags:
  - OpenClaw
  - Tools
  - System Prompt
  - LLM
  - Function Calling
excerpt: LLM 怎么"知道"你有什么工具？答案不在一个地方——System Prompt 里的自然语言提示和 API tools 字段的结构化 Schema 各司其职，缺一不可。
---

好问题！这确实容易搞混。LLM 知道工具的信息来自**两个不同的地方**，不是一个。

## 工具信息的两个渠道

```
发给 DeepSeek 的 HTTP 请求：

POST https://api.deepseek.com/chat/completions
{
 "model": "deepseek-chat",

 "messages": [ ← 渠道 1️⃣：自然语言描述
 {
 "role": "system",
 "content": "...## Tooling
- exec: Run shell commands
- read: Read file contents
..."
 },
 { "role": "user", "content": "帮我看看文件" }
 ],

 "tools": [ ← 渠道 2️⃣：结构化定义（真正让 LLM 能调用的）
 {
 "type": "function",
 "function": {
 "name": "exec",
 "description": "Run shell commands...",
 "parameters": {
 "type": "object",
 "properties": {
 "command": { "type": "string", "description": "Shell command to execute" },
 "timeout": { "type": "number", "description": "Timeout in seconds" },
 "background": { "type": "boolean" }
 },
 "required": ["command"]
 }
 }
 },
 {
 "type": "function",
 "function": {
 "name": "read",
 "description": "Read file contents...",
 "parameters": {
 "type": "object",
 "properties": {
 "file_path": { "type": "string", "description": "Path to the file" }
 },
 "required": ["file_path"]
 }
 }
 },
 ... // 30-50 个工具的完整 JSON Schema
 ],

 "stream": true
}
```

---

## 渠道 1️⃣：System Prompt 里的工具摘要（给 LLM 的「导航提示」）

```
## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- exec: Run shell commands (pty available for TTY-required CLIs)
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
- web_search: Search the web
- cron: Manage cron jobs and wake events
- message: Send messages to channels
...
```

**这只是一个「提示/索引」**，帮 LLM 理解整体有什么工具可用，在**自然语言层面**给 LLM 一些额外上下文（比如「exec 支持 PTY」、「cron 用于定时任务和提醒」这些信息可能不会出现在结构化 schema 里）。

---

## 渠道 2️⃣：`tools` 字段（这才是 LLM 真正用来调用工具的）

```json
{
 "type": "function",
 "function": {
 "name": "exec",
 "description": "Execute a shell command. Prefer this for running scripts, installing packages, compiling code, and managing processes. Use 'background: true' or 'yieldMs' for long-running commands.",
 "parameters": {
 "type": "object",
 "properties": {
 "command": {
 "type": "string",
 "description": "Shell command to execute"
 },
 "workdir": {
 "type": "string",
 "description": "Working directory (defaults to cwd)"
 },
 "timeout": {
 "type": "number",
 "description": "Timeout in seconds (optional, kills process on expiry)"
 },
 "background": {
 "type": "boolean",
 "description": "Run in background immediately"
 },
 "host": {
 "type": "string",
 "description": "Exec host/target (auto|sandbox|gateway|node)"
 }
 },
 "required": ["command"]
 }
 }
}
```

**这才是 LLM 能「触发」工具调用的机制**。LLM 看到这些结构化定义后，如果决定要调用工具，会返回：

```json
{
 "role": "assistant",
 "tool_calls": [{
 "id": "call_abc123",
 "type": "function",
 "function": {
 "name": "exec",
 "arguments": "{\"command\": \"cat src/main.ts\"}"
 }
 }]
}
```

---

## 代码里两者是怎么分开构建的

```
 createOpenClawCodingTools()
 │
 ▼
 effectiveTools = [
 { name: "exec",
 description: "Execute a shell command...",
 parameters: { type: "object", properties: {...} },
 execute: async (toolCallId, args) => {...} ← 执行逻辑
 },
 { name: "read", ... },
 ...
 ]
 │
 ┌───────────────┼────────────────────┐
 │ │ │
 ▼ ▼ ▼
 System Prompt createAgentSession() Transport 层
 buildAgentSystemPrompt() │ │
 │ │ │
 │ 只取 name + summary │ 注册工具到 session │ 构建 API 请求
 │ 生成: │ tools = [...] │
 │ "- exec: Run shell..." │ customTools = [...] │ params.tools =
 │ "- read: Read file..." │ │ convertTools(context.tools)
 │ │ │ → { type: "function",
 │ │ │ function: { name, desc,
 │ 塞进 messages[0] │ │ parameters } }
 │ .content 里 │ │
 │ │ │ 塞进请求体的 tools 字段
 ▼ ▼ ▼

 messages[0].content session 内部保存 request.tools = [...]
 (自然语言提示) execute 函数引用 (结构化 JSON Schema)
```

---

## 三个东西的对应关系

| | System Prompt 里 | API `tools` 字段 | 代码里 |
|--|--|--|--|
| **信息** | `- exec: Run shell commands` | `{ name: "exec", description: "...", parameters: {...} }` | `{ name, description, parameters, execute }` |
| **用途** | 让 LLM 大致了解「有什么武器」 | 让 LLM 知道「怎么调用」（参数结构） | 让代码知道「怎么执行」 |
| **给谁看** | LLM（自然语言理解） | LLM（结构化解析） | 代码运行时 |
| **是否必须** | 不是必须（辅助） | **必须**（没有这个 LLM 无法调用） | **必须**（没有这个工具无法执行） |

---

## 为什么 System Prompt 里还要重复列一遍？

你可能会问：`tools` 字段已经有完整的工具定义了，为什么 system prompt 里还要列？

**三个原因**：

### 1. 补充 `tools` schema 没法表达的上下文

```
System Prompt 里：
 "- cron: Manage cron jobs and wake events (use for reminders; ...)"
 "- exec: Run shell commands (pty available for TTY-required CLIs)"
 "- sessions_spawn: Spawn an isolated sub-agent session; prefer this for parallel work"

API tools schema 里的 description 通常更短更技术性：
 "description": "Execute a shell command"

System Prompt 给了额外的「何时该用这个工具」的指导。
```

### 2. 稳定 prompt 缓存

工具列表是稳定的（同一 session 内不变），放在 system prompt 里可以被 prefix cache 覆盖。

### 3. 指导工具优先级和使用规范

```
System Prompt 里有类似这些规则：
 "When a first-class tool exists for an action, use the tool directly
 instead of asking the user to run equivalent CLI commands."
 "Prefer exec over asking the user to run commands."
 "Use sessions_spawn for parallel independent work."
```

这些「策略性指导」没法通过 `tools` 的 JSON Schema 表达。

---

## 完整流程：LLM 怎么「决定」调工具

```
1. LLM 收到请求，看到：
 - messages[0] (system): "你有这些工具: exec, read, write..."
 - messages[-1] (user): "帮我看看 src/main.ts"
 - tools: [{ name: "read", parameters: { file_path: "string" } }, ...]

2. LLM 内部推理：
 "用户想看文件 → 我有 read 工具 → 参数需要 file_path"

3. LLM 输出：
 { "tool_calls": [{ "function": { "name": "read", "arguments": "{\"file_path\":\"src/main.ts\"}" } }] }

4. pi-coding-agent 收到 tool_calls：
 → 查 customTools 里 name="read" 的工具
 → 调用 tool.execute("call_123", { file_path: "src/main.ts" })
 → 工具返回文件内容

5. 库自动把结果塞回 messages：
 messages.push({ role: "tool", tool_call_id: "call_123", content: "import React..." })
 → 重新调 LLM API（带上 tool_result）

6. LLM 看到文件内容 → 生成自然语言回复：
 "这个文件是一个 React 组件，它导入了..."
```

---

## 总结

> Agent 是怎么知道我有什么工具、它们之间是做什么的？

**答案是两层**：

1. **`tools` 请求字段**（结构化）—— 这是 **LLM 知道自己「能调什么」** 的根本来源。没有这个字段，LLM 不会返回 `tool_calls`。
2. **System Prompt 里的工具列表**（自然语言）—— 这是 **补充性的上下文指导**，帮 LLM 在**「什么时候该用什么工具」** 上做更好的决策。

**缺一不可**：

- 只有 `tools` 没有 system prompt 指导 → LLM 知道怎么调但不知道**何时该调**
- 只有 system prompt 没有 `tools` → LLM 知道有什么工具但**无法真正调用**（API 不允许返回 tool_calls）
