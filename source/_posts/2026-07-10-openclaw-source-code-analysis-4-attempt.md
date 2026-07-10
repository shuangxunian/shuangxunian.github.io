---
title: "OpenClaw 源码解析 ④：runEmbeddedAttempt 七阶段拆解"
date: 2026-07-10
categories: [技术]
tags: [OpenClaw, 源码解析, Agent, TypeScript]
---

# runEmbeddedAttemptWithBackend 完整拆解

这是 OpenClaw 源码解析系列第四篇。上一篇逐行走读了 `runEmbeddedPiAgent` 的主循环，这一篇深入它内部最核心的函数——`runEmbeddedAttemptWithBackend`，以及它最终调用的 `runEmbeddedAttempt`（2600 行）。

---

## 调用链

```
runEmbeddedPiAgent()
 └─ runEmbeddedAttemptWithBackend()   ← 选一个"执行后端"
     └─ runAgentHarnessAttemptWithFallback()
         └─ selectAgentHarnessDecision()  ← 选 "pi" 还是插件
             └─ harness.runAttempt()
                 └─ runEmbeddedAttempt()  ← 真正干活的（2600 行）
```

**为什么要有 Harness 这层间接？**

OpenClaw 设计了一个 Agent Harness 抽象层，方便将来把 Agent 执行后端换成别的东西（比如插件提供的后端）。但目前 99% 的情况选的都是内置的 `"pi"` harness：

```typescript
// harness/builtin-pi.ts
export function createPiAgentHarness(): AgentHarness {
  return {
    id: "pi",
    label: "PI embedded agent",
    supports: () => ({ supported: true, priority: 0 }),
    runAttempt: runEmbeddedAttempt,  // ← 直接指向 attempt.ts
  };
}
```

所以 `runEmbeddedAttemptWithBackend` → 选 harness → 最终就是调 `runEmbeddedAttempt()`。

---

## `runEmbeddedAttempt` —— 七个阶段

2600 行的核心函数，按执行顺序分为七个阶段：

```
┌────────────────────────────────────────────────────────────────────┐
│ Stage 1: 准备沙箱 + 工作目录          (行 436-470)                 │
│ Stage 2: 加载技能 Skills + 环境变量    (行 470-492)                 │
│ Stage 3: 构建工具清单 tools            (行 507-735)                 │
│ Stage 4: 构建系统提示词 system prompt   (行 747-969)                 │
│ Stage 5: 创建 Agent 会话 + 加载历史    (行 972-1177)                │
│ Stage 6: 注册 Subscription 事件管道    (行 1753-1807)               │
│ Stage 7: 提交 Prompt → 发给 LLM        (行 2282-2338)               │
└────────────────────────────────────────────────────────────────────┘
```

---

### Stage 1: 准备沙箱 + 工作目录

```typescript
// 行 447-461
await fs.mkdir(resolvedWorkspace, { recursive: true });

// 解析沙箱配置
const sandbox = await resolveSandboxContext({
  config, sessionKey, workspaceDir
});

// 如果启用了沙箱：用沙箱的目录；否则用原始工作目录
const effectiveWorkspace = sandbox?.enabled
  ? sandbox.workspaceDir
  : resolvedWorkspace;
```

如果配置了 Docker 沙箱，后续的 bash/read/write 工具操作都会在沙箱容器里执行。

---

### Stage 2: 加载技能 Skills

```typescript
// 行 470-492
const { skillEntries } = resolveEmbeddedRunSkillEntries({
  workspaceDir, config, agentId
});

// 把 skill 的环境变量应用到当前进程
restoreSkillEnv = applySkillEnvOverrides({ skills, config });

// 把 skill 的描述拼成给 LLM 看的格式
const skillsPrompt = resolveSkillsPromptForRun({
  entries: skillEntries, workspaceDir
});
// 示例输出：
// "Available skills: weather, summarize, diagram-maker..."
// "Use read_skill tool to load a skill recipe."
```

---

### Stage 3: 构建工具清单（最复杂，~230 行）

```typescript
// 行 507-735（简化版）

// 第1步：创建所有内置工具
const allTools = createOpenClawCodingTools({
  agentId, sandbox, messageProvider, config,
  abortSignal, workspaceDir, ...
});
// 返回: [exec, read, write, edit, glob, grep, ...]

// 第2步：应用工具策略过滤
const tools = applyEmbeddedAttemptToolsAllow(allTools, params.toolsAllow);
// 如果 toolsAllow = ["read", "exec"] → 只保留这两个

// 第3步：规范化工具 schema（不同 provider 格式不同）
const normalizedTools = normalizeProviderToolSchemas({
  tools, provider, modelId, config
});
// OpenAI 格式 vs Anthropic 格式 vs Google 格式

// 第4步：加载 MCP 工具
const bundleMcpRuntime = await materializeBundleMcpToolsForRun({
  runtime: mcpSessionRuntime,
  reservedToolNames: tools.map(t => t.name)
});

// 第5步：加载 LSP 工具（代码智能）
const bundleLspRuntime = await createBundleLspToolRuntime({
  workspaceDir, config
});

// 第6步：合并所有工具
const effectiveTools = [...tools, ...mcpTools, ...lspTools];
// 最终可能有 30-50 个工具发给 LLM
```

**类比**：这一步就像给 AI 装工具箱——确定它这次对话能用哪些工具、不能用哪些。工具有五个来源：内置工具、MCP 服务器、LSP 服务、客户端注册、插件注册。

---

### Stage 4: 构建系统提示词

```typescript
// 行 826-968（简化版）

// 1. 收集运行时信息
const { runtimeInfo, userTimezone, userTime } = buildSystemPromptParams({
  config,
  runtime: {
    host: "my-macbook",
    os: "Darwin 24.0.0",
    arch: "arm64",
    node: "v22.12.0",
    model: "anthropic/claude-opus-4-6",
    shell: "/bin/zsh",
    channel: "telegram",
    capabilities: ["reactions", "threads"],
  }
});

// 2. 组装系统提示词（一个巨大的字符串）
const builtAppendPrompt = buildEmbeddedSystemPrompt({
  workspaceDir, reasoningTagHint, heartbeatPrompt,
  skillsPrompt, docsPath, ttsHint, sandboxInfo,
  tools: effectiveTools,   // 告诉 LLM 有哪些工具
  runtimeInfo,              // 系统环境信息
  contextFiles,             // CLAUDE.md / BOOTSTRAP.md
  userTimezone, userTime,
  ...
});

// 3. 让 Provider 插件有机会修改
const appendPrompt = transformProviderSystemPrompt({
  provider, config, systemPrompt: builtAppendPrompt
});
```

最终系统提示词大概 5000-20000 字符，包含环境信息、可用工具清单、技能列表、安全规则和 workspace 上下文文件。

---

### Stage 5: 创建 Agent 会话 + 加载历史

```typescript
// 行 972-1177（简化版）

// 1. 修复可能损坏的 session 文件
await repairSessionFileIfNeeded({ sessionFile });

// 2. 打开 Session Manager（管理 JSONL transcript 文件）
sessionManager = guardSessionManager(
  SessionManager.open(params.sessionFile),
  { contextWindowTokens, allowedToolNames }
);

// 3. 配置 compaction 设置
const settingsManager = createPreparedEmbeddedPiSettingsManager({
  contextTokenBudget  // 例如：128000 tokens
});
applyPiAutoCompactionGuard(settingsManager);

// 4. 创建 Agent 会话对象
const createdSession = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  model: params.model,
  thinkingLevel: "off",
  tools: toolAllowlist,
  customTools: allCustomTools,
  sessionManager,
});
session = createdSession.session;

// 5. 注入系统提示词
applySystemPromptOverrideToSession(session, systemPromptText);
```

`createAgentSession` 来自 `@mariozechner/pi-coding-agent` 库——底层 Agent 框架。它做了：从 transcript JSONL 文件加载之前的对话 → 构建 messages 数组（system + history + user prompt）→ 注册所有工具 schema → 准备好模型 API client。

---

### Stage 6: 注册 Subscription（事件管道）

```typescript
// 行 1753-1807

const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onAgentEvent: params.onAgentEvent,
  ...
});

const {
  assistantTexts,           // 累积的助手回复文本
  toolMetas,                // 执行过的工具元信息
  unsubscribe,              // 取消订阅
  waitForCompactionRetry,   // 等待 compaction
  isCompactionInFlight,     // 是否正在 compaction
  getUsageTotals,           // token 用量统计
  getCompactionCount,       // compaction 次数
} = subscription;
```

在发出 prompt 之前先把接收管道搭好。LLM 流式响应开始后，每个 token/tool call/compaction 事件都会通过这个管道处理。

---

### Stage 7: 提交 Prompt → 发给 LLM

```typescript
// 行 2282-2338（最关键的一步！）

if (imageResult.images.length > 0) {
  await abortable(
    activeSession.prompt(effectivePrompt, { images: imageResult.images })
  );
} else {
  await abortable(
    activeSession.prompt(effectivePrompt)
  );
}
```

**就这两行！** 整个 2600 行函数的核心动作：

```
activeSession.prompt("帮我总结这篇文章")
```

这行代码：
1. 把 `effectivePrompt`（用户消息）加到 messages 数组末尾
2. 调用 LLM API（通过 provider 插件 → HTTP/WS 请求）
3. **等待整个流式响应完成**（中间的每个 token 由 Stage 6 的 subscription 处理）
4. 如果 LLM 返回 tool calls → 框架自动执行工具 → 结果再发回 LLM → 循环直到 LLM 给出文本

`abortable()` 是一个包装器，让整个 promise 可以被 `abortSignal` 取消。

---

## 完整流程简图

```
runEmbeddedAttempt() {

  // === 准备阶段（同步，快）===
  Stage 1: mkdir workspace
  Stage 2: 加载 skills
  Stage 3: 构建 30-50 个工具定义
  Stage 4: 拼接系统提示词（~5000-20000 字符）
  Stage 5: 加载 session transcript → messages 数组
  Stage 6: 搭好事件接收管道

  // === 执行阶段（异步，几十秒到几分钟）===
  Stage 7: await session.prompt(userMessage)
  │
  ├── HTTP POST 到 LLM API
  ├── 流式接收 token → onPartialReply()
  ├── 如果收到 tool_call:
  │   ├── 找到对应工具实现
  │   ├── 执行工具（e.g. exec("ls -la")）
  │   ├── 把 tool_result 发回 LLM
  │   └── 继续等待下一批 tokens...
  └── 流式结束 → 返回

  // === 收尾 ===
  等 compaction 完成（如果有）
  收集 usage 统计
  返回 result
}
```

---

## 关键"厚度"在哪

| 阶段 | 行数 | 为什么这么"厚" |
|------|------|---------------|
| Stage 3: 工具构建 | ~230 行 | 5 种来源（内置/MCP/LSP/客户端/插件）+ policy 过滤 + provider schema 规范化 |
| Stage 4: 系统提示词 | ~200 行 | 20+ 信息源综合：环境、channel、skills、config、hooks、context files |
| Stage 5: 会话加载 | ~200 行 | transcript 修复、compaction guard、context engine 初始化 |
| Stage 7 后续 | ~300 行 | compaction wait、yield abort、timeout、各种边界处理 |

---

## 总结

`runEmbeddedAttempt` 的核心逻辑其实很简单——**"准备一个充分的上下文 → 把一切塞给 LLM → 处理结果"**。只不过"准备上下文"这一步要考虑的因素实在太多了，所以撑出了 2600 行。

而最核心的那个动作，只有一行：

```typescript
await activeSession.prompt("帮我总结这篇文章")
```

---

## 下一步

这一篇拆解了 `runEmbeddedAttempt` 的七个阶段。接下来的可选方向：

- **Stage 6 深入**：subscription 管道怎么处理流式 token 和 tool call
- **Stage 3 深入**：`createOpenClawCodingTools` 具体创建了哪些工具，每个工具长什么样
- **Stage 4 深入**：系统提示词的完整内容是什么样的

下一篇将从其中一个方向深入。敬请期待。

---

> 本系列文章基于 OpenClaw v2026.4.23 源码分析，代码引用以实际文件行号为准。
