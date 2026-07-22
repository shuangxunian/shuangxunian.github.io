---
title: "OpenClaw 源码解析 ⑥：系统提示词完整拆解"
date: 2026-07-10
categories:
- AI技术
tags: [OpenClaw, 源码解析, Agent, Prompt Engineering]
---

OpenClaw 源码解析系列第六篇。上一篇拆解了工具系统的七层洋葱模型，这一篇聚焦 Agent 和 LLM 之间最核心的"契约"——系统提示词。

---

## 总览：三大结构图

```
┌──────────────────────────────────────────────────────────┐
│ 系统提示词 (总共约 2000-8000 tokens)                      │
│                                                          │
│ ┌────────────────────────────────────────────────────┐   │
│ │ Provider 前缀 (stablePrefix)                       │   │
│ │ 由 AI 模型提供商插件注入                             │   │
│ │ 示例: Anthropic 不注入, Codex 注入行为指南            │   │
│ ├────────────────────────────────────────────────────┤   │
│ │ OpenClaw 核心提示词 ↓ 接下来逐段讲                    │   │
│ │ ┌── identity ────┐  ┌── tooling ───┐               │   │
│ │ ├── style ───────┤  ├── safety ────┤               │   │
│ │ ├── cli ─────────┤  ├── skills ────┤               │   │
│ │ ├── memory ──────┤  ├── update ────┤               │   │
│ │ ├── workspace ───┤  ├── sandbox ───┤               │   │
│ │ ├── identity ────┤  ├── time ──────┤               │   │
│ │ ├── output ──────┤  ├── canvas ────┤               │   │
│ │ ├── messaging ───┤  ├── voice ─────┤               │   │
│ │ ├── reactions ───┤  ├── reasoning ─┤               │   │
│ │ ├── heartbeat ───┤  └── runtime ───┘               │   │
│ ├────────────────────────────────────────────────────┤   │
│ │ 缓存边界 (CACHE_BOUNDARY)                           │   │
│ ├────────────────────────────────────────────────────┤   │
│ │ 项目上下文 (Project Context)                         │   │
│ │ ┌── AGENTS.md / BOOTSTRAP.md / CLAUDE.md ──┐        │   │
│ ├────────────────────────────────────────────────────┤   │
│ │ 动态上下文 (Dynamic Context)                         │   │
│ │ ┌── HEARTBEAT.md ──┐ 以及其他动态文件               │   │
│ ├────────────────────────────────────────────────────┤   │
│ │ Provider 后缀 (dynamicSuffix)                       │   │
│ │ 由模型提供商插件注入的动态变量                         │   │
│ │ 示例: "current date: ..." 等变化的内容               │   │
│ └────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## `buildAgentSystemPrompt()` — 逐段拆解

核心逻辑就是往一个 `lines` 数组里 push 字符串，最后 `.join("\n\n")` 一把拼接。

---

### 第 1 段：身份 Identity

```typescript
lines = [
  "You are a personal assistant running inside OpenClaw.",
]
```

**只有一句话。** LLM 不需要知道太多"你是谁"——后面的全是"你该怎么做"。

---

### 第 2 段：工具清单 Tooling

```typescript
lines.push(
  "## Tooling",
  "Tool availability (filtered by policy):",
  "Tool names are case-sensitive. Call tools exactly as listed.",
  // 对每个启用的工具生成一行描述：
  "- exec: Run shell commands (pty available for TTY-required CLIs)",
  "- read: Read file contents",
  "- write: Create or overwrite files",
  "- edit: Make precise edits to files",
  "- web_search: Search the web (Brave API)",
  "- web_fetch: Fetch and extract readable content from a URL",
  "- sessions_spawn: Spawn an isolated sub-agent session",
  "- cron: Manage cron jobs and wake events",
  // ... 20-40 个工具
)
```

把 Stage 3 阶段构建的工具名 + 摘要列出来，让 LLM 知道"我有哪些武器"。

---

### 第 3 段：风格约束 + 工具调用风格

```typescript
lines.push(
  "## Interaction Style", "...",
  "## Tool Call Style",
  "Default: do not narrate routine, low-risk tool calls.",
  "Narrate only when it helps: multi-step work, complex problems.",
  "Keep narration brief and value-dense.",
  "Never execute /approve through exec or any shell path.",
  "## Execution Bias",
  "Prefer completing tasks over asking.",
  "If a task requires multiple independent, slow inputs, parallelize."
)
```

本质是**"你该怎么说话"**和**"你该怎么用工具"**两条指令。

---

### 第 4 段：安全 Safety

```typescript
lines.push(
  "## Safety",
  "You have no independent goals: do not pursue self-preservation, replication...",
  "Prioritize safety and human oversight over completion.",
  "Do not manipulate or persuade anyone to expand access or disable safeguards.",
  "Do not copy yourself or change system prompts, safety rules, or tool policies."
)
```

直接借鉴 Anthropic 宪法。防止 AI 越狱、自我复制、绕过限制。

---

### 第 5 段：CLI 速查

```typescript
lines.push(
  "## OpenClaw CLI Quick Reference",
  "OpenClaw is controlled via subcommands. Do not invent commands.",
  "- OpenClaw gateway status",
  "- OpenClaw gateway start",
  "- OpenClaw gateway stop",
  "- OpenClaw gateway restart",
)
```

防止 LLM 瞎编命令。

---

### 第 6 段：Skills & Memory

```typescript
const skillsSection = buildSkillsSection({ skillsPrompt, readToolName });
// "## Available Skills"
// "- weather: Get weather forecasts"
// "- summarize: Summarize URLs, YouTube videos, PDFs..."
// "Use read_skill to load a skill recipe."

const memorySection = buildMemorySection({ isMinimal, ... });
// "## Memory"
// "Memory is available. When the user asks you to remember something..."
```

Skills 和 Memory 各自由独立的 builder 函数构建，主流程只负责把它们插进去。

---

### 第 7 段：Self-Update

```typescript
if (hasGateway && !isMinimal) {
  lines.push(
    "## OpenClaw Self-Update",
    "Updates are ONLY allowed when the user explicitly asks for it.",
    "Do not update or restart unless the user explicitly requests it."
  );
}
```

LLM 没有主动升级系统的权限。

---

### 第 8 段：Workspace 工作区

```typescript
lines.push(
  "## Workspace",
  "Your working directory is: /home/user/project",
  "Treat this directory as the single global workspace unless otherwise instructed."
)
```

如果启用了沙箱，会额外说明路径映射规则。

---

### 第 9 段：Sandbox 沙箱

```typescript
lines.push(
  "## Sandbox",
  "You are running in a sandboxed runtime (tools run in isolated containers).",
  "Sub-agents stay sandboxed (no elevated/host access).",
  "Sandbox container workdir: /workspace",
  "Agent workspace access: rw",
  "Elevated exec is available for this session."
)
```

---

### 第 10 段：用户身份

```typescript
lines.push(
  "## User Identity",
  "Authorized senders: hash:a1b2c3d4, hash:e5f6g7h8."
)
```

不会在 prompt 里泄露真实手机号/用户名，默认 hash 显示。

---

### 第 11 段：时间

```typescript
lines.push(
  "## Current Date & Time",
  "Time zone: Asia/Shanghai",
  "If you need the current date, time, or day of week, run session_status."
)
```

不写死当前时间——让 LLM 需要时调 `session_status` 获取，这样 prompt 本身可以被缓存。

---

### 第 12-17 段：Workspace Files / Output / Canvas / Messaging / Voice / Reactions

```typescript
lines.push("## Workspace Files (injected)", "...");
lines.push("## Output Style", "Speak the user's language unless asked otherwise.");
if (runtimeChannel === "webchat") lines.push("## Canvas", "...");
lines.push("## Messaging", "Your current channel is Discord...");
if (ttsHint) lines.push("## Voice", ttsHint);
if (reactionGuidance) lines.push("## Reactions", "...");
```

这些是根据 channel / config 动态变化的部分。

---

### 第 18 段：Reasoning Format

```typescript
if (reasoningHint) {
  lines.push(
    "## Reasoning Format",
    "ALL internal reasoning MUST be inside <think>...</think>.",
    "Do not output any analysis outside <think>.",
    "Format every reply as <think>...</think> then <final>...</final>."
  );
}
```

---

### 第 19 段：Silent Replies

```typescript
lines.push(
  "## Silent Replies",
  "When you have nothing to say, respond with ONLY: HEARTBEAT_OK",
)
```

**"没话可说时的统一暗号"**。避免了 LLM 回复 "好的，收到" 这种废话。

---

## 缓存边界 (CACHE_BOUNDARY)

```typescript
lines.push(SYSTEM_PROMPT_CACHE_BOUNDARY);
// "__OPENCLAW_PROMPT_CACHE_BOUNDARY__"
```

**这是魔法分隔符！** 以上所有内容是"稳定的"（同一次 session 内不会变），可以被 Provider（如 Anthropic）缓存起来跨 turns 复用。以下内容每次对话可能变化，不缓存。

---

## 项目上下文

```typescript
lines.push(
  "# Project Context",
  "## AGENTS.md", "[文件内容]...",
  "## CLAUDE.md", "[文件内容]...",
  "# Dynamic Project Context",
  "## HEARTBEAT.md", "[文件内容]...",
);
```

文件按优先级排序：`AGENTS.md > soul.md > identity.md > user.md > tools.md > BOOTSTRAP.md > memory.md`

---

## Heartbeat 指令

```typescript
lines.push(
  "## Heartbeat",
  "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly.",
);
```

告诉 LLM 当心跳触发时该怎么做。

---

## Runtime 环境信息（最后一段！）

```typescript
lines.push(
  "## Runtime",
  "host=my-macbook os=Darwin 24.0.0 arch=arm64 node=v22.12.0" +
  " model=anthropic/claude-opus-4-6 shell=/bin/zsh channel=telegram",
  "Reasoning: off (hidden unless on/stream)."
);
```

这是 LLM 知道的关于运行环境的全部信息。注意它没有当前时间——需要时 LLM 要自己调 `session_status` 获取。

---

## Provider 后缀

```typescript
if (providerDynamicSuffix) {
  lines.push(providerDynamicSuffix);
  // 示例（来自某个 provider 插件）：
  // "Today is 2026-07-10. The current time is 14:30:00."
}
```

有些 Provider 会在这里注入动态变化的参数（如当前日期），因为这个内容没法缓存。

---

## 最终拼接

```typescript
return lines.filter(Boolean).join("\n\n");
```

所有非空的 `lines` 项用 `\n\n` 连起来，就是最终发给 LLM 的系统提示词。

---

## 系统提示词的三种模式

| Mode | 用途 | 内容 |
|------|------|------|
| `full` | 主 Agent | 全部 19 段 |
| `minimal` | 子 Agent | 只保留 Tooling、Workspace、Runtime 三段 |
| `none` | 探测/测试 | 只返回 "You are a personal assistant running inside OpenClaw." |

子 Agent 用 `minimal` 模式可以大幅减少 token 消耗——子 Agent 不需要知道 Messaging / Self-Update / Reactions 这些主 Agent 专属的东西。

---

## CACHE_BOUNDARY 的设计智慧

```
┌──────────── 稳定区域 ────────────┐
│ Identity / Tool List / Safety   │
│ Style / Workspace / Timezone   │
│ (相同 session 内不变)            │
│                                  │
│ === CACHE_BOUNDARY ===           │ ← 从这里断开
│                                  │
│ Project Context (AGENTS.md)     │
│ Dynamic Context (HEARTBEAT.md)  │
│ Heartbeat / Runtime             │
│ Provider 注入的当前时间           │
│ (每次对话可能变化)                │
└──────────────────────────────────┘
```

Anthropic 的 prompt caching 机制可以利用这个边界：`CACHE_BOUNDARY` 以上的内容跨 turns 复用，只为下面变化的部分付费——**省掉 50-80% 的 prompt 开销**。

---

## 总结：三个层次

```
┌─────────────────────────────────────────────────┐
│ 第 1 层：固化的"规则书"                           │
│ Tooling / Safety / Style / CLI / Self-Update    │
│ → src/agents/system-prompt.ts 里硬编码           │
│ → 跨所有用户都相同                                │
├─────────────────────────────────────────────────┤
│ 第 2 层：用户/会话特定的"配置"                     │
│ Skills / Memory / Workspace / Sandbox / Timezone│
│ Channel / Canvas / Voice / Reactions           │
│ → 根据 config + runtime info 动态生成            │
│ → 同一会话内不变，可以缓存                        │
├─────────────────────────────────────────────────┤
│ 第 3 层：可变动的"上下文"                         │
│ Project Context (CLAUDE.md etc)                │
│ Heartbeat / Provider 动态内容 / 当前时间          │
│ → 每次对话可能变化                               │
│ → CACHE_BOUNDARY 以下，不缓存                    │
└─────────────────────────────────────────────────┘
```

这就是 300 行代码、值 2000-8000 tokens 的系统提示词完整构建逻辑。

---

## 下一步

系统提示词拆解完了。接下来的可选方向：

- **Subscription 管道**：LLM 返回流式 token 后，subscription 怎么逐个处理
- **Compaction**：当 context window 满了怎么压缩
- **Session 文件格式**：对话历史的 JSONL 结构

下一篇将从其中一个方向深入。敬请期待。

---

> 本系列文章基于 OpenClaw v2026.4.23 源码分析。
