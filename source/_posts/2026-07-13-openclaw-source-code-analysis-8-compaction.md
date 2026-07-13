---
title: OpenClaw 源码解析 ⑧：Compaction 完整拆解
date: 2026-07-13 09:31:00
categories:
  - OpenClaw
  - 源码解析
tags:
  - OpenClaw
  - Compaction
  - LLM
  - Context Window
  - AI Agent
excerpt: 当对话上下文超出 LLM context window 时，Compaction 如何通过生成摘要来压缩历史、腾出空间——从触发条件到完整流程的深入拆解。
---

# Compaction：完整拆解

## 一句话解释

**Compaction = 当上下文超出 LLM 的 context window 时，用一个独立的 LLM 调用生成「对话摘要」，替换掉大部分历史消息，腾出空间继续对话。**

---

## 为什么需要 Compaction？

```
正常的 Agent 执行：
 问 → LLM 理解 → 调用工具 → LLM 获得结果 → 回复

 每一轮对话 = 多则几十个 messages (tool_use + tool_result + assistant)
 20 轮下来 = 500-1000 条 messages
 如果每条 tool_result 有几千字符的 stdout → 上下文轻松超过 100K tokens

LLM 的 context window 是有限的：
 GPT-4o = 128K tokens
 Claude 4 = 200K tokens
 本地小模型 = 8K-32K tokens

当 messages 总 token 逼近甚至超过 context window 时：
 → LLM API 返回 "context_length_exceeded" 错误
 → 或者最前面的消息被截断，LLM 丢失关键上下文
```

Compaction 就是**在超出之前主动压缩历史，防止爆窗口**。

---

## Compaction 的触发时机

```
┌─────────────────────────────────────────────────────────────────┐
│ 触发条件                                                         │
│                                                                   │
│ 1. Overflow Compaction（溢出触发）                                │
│    触发位置：run.ts 主循环，每次 LLM 返回之后                     │
│    条件：LLM 返回 "context_length_exceeded" 错误                  │
│    代码：run.ts:1029-1140                                         │
│                                                                   │
│ 2. Timeout-triggered Compaction（超时触发）                       │
│    触发位置：run.ts 主循环，LLM 超时之后                          │
│    条件：LLM 超时 + prompt token 用量 > 70% context window       │
│    代码：run.ts:890-977                                           │
│                                                                   │
│ 3. Auto Compaction（SDK 自动触发）                                │
│    触发位置：pi-coding-agent 库内部                               │
│    条件：SessionManager 检测到 token 接近上限                     │
│    在 session.prompt() 内部自动完成，对 OpenClaw 透明             │
│                                                                   │
│ 4. Manual Compaction（手动触发）                                  │
│    触发位置：用户通过 CLI/Gateway 命令手动发起                    │
│    条件：用户主动调用 /compact                                    │
│    trigger = "manual"                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 两种 Compact 入口

```
compactEmbeddedPiSession()        compactEmbeddedPiSessionDirect()
(compact.queued.ts)               (compact.ts)

 有 Lane 排队                       没有 Lane 排队
 从外部调用                         从 Agent 内部调用
     ↓                                    ↓
 检查 harness 是否接管              直接执行
     ↓                                    ↓
 排 sessionLane + globalLane    ... 一样的逻辑 ...
     ↓
 → compactEmbeddedPiSessionDirect()
```

---

## `compactEmbeddedPiSessionDirect` 完整流程

```
┌─────────────────────────────────────────────────────────────────────┐
│ Step 1: 选择 Compaction 模型                                         │
│                                                                       │
│ compaction 不一定用主 Agent 的模型！                                  │
│ resolveEmbeddedCompactionTarget() →                                  │
│   可以配置一个专门的小模型做摘要（省 token）                         │
│   默认：和主 Agent 用同一个模型                                      │
│                                                                       │
│ Step 2: 解析模型 + 获取 API key                                      │
│   和主 Agent 一样的流程：resolveModelAsync() → getApiKeyForModel()  │
│                                                                       │
│ Step 3: 准备沙箱 + workspace + skills                                │
│   和 runEmbeddedAttempt 的前几步完全一样                             │
│                                                                       │
│ Step 4: 构建「compaction 专用的系统提示词」                          │
│   和主 Agent 用同一套 buildEmbeddedSystemPrompt()                    │
│   只是 promptMode = "minimal"（更精简）                              │
│                                                                       │
│ Step 5: 打开 SessionManager + 修复 session 文件                      │
│   SessionManager.open(sessionFile)                                   │
│   → 读入 JSONL transcript                                            │
│                                                                       │
│ Step 6: 对历史做清理（不清理就让 LLM 做摘要）                        │
│   sanitizeSessionHistory() → 清理损坏的 tool_result                  │
│   validateReplayTurns() → 验证每轮对话的完整性                       │
│   limitHistoryTurns() → 限制 DM channel 的历史轮次                   │
│                                                                       │
│ Step 7: 运行 before_compaction hooks                                 │
│   → 让 memory 插件在 compaction 前做长期记忆持久化                   │
│   这是关键的一步！不持久化的话 compaction 可能丢失重要记忆           │
│                                                                       │
│ Step 8: 创建新的 Agent 会话 + 调用 compact()                         │
│                                                                       │
│   const createdSession = await createAgentSession({ ... });           │
│   注入 compact 专用的系统提示词                                      │
│                                                                       │
│   activeSession.messages 现在有：                                    │
│     [系统提示词, 完整的对话历史 (可能很多条)]                        │
│                                                                       │
│   await compactWithSafetyTimeout(                                    │
│     () => activeSession.compact(customInstructions),                 │
│     900_000 // 默认 15 分钟超时                                      │
│   );                                                                  │
│                                                                       │
│   ↑ 这就是核心！一个独立的 LLM 调用，目的是：                        │
│     「阅读全部对话历史，生成一个摘要，告诉我保留哪些关键消息」       │
│                                                                       │
│ Step 9: activeSession.compact() 内部做了什么？                       │
│                                                                       │
│   ┌──────────────────────────────────────────────────────┐          │
│   │ compact() 调用 LLM，输入是：                         │          │
│   │                                                      │          │
│   │   System: "You are a summarizer. Read the            │          │
│   │            conversation and produce:                 │          │
│   │            1. A structured summary of key decisions  │          │
│   │               and outcomes                           │          │
│   │            2. A list of message IDs to KEEP          │          │
│   │               (messages critical for future turns)   │          │
│   │            ..."                                      │          │
│   │                                                      │          │
│   │   Messages: [全部对话历史]                           │          │
│   │                                                      │          │
│   │   LLM 输出:                                          │          │
│   │   ┌─ summary_text ──────────────────────────┐      │          │
│   │   │ 用户要求开发一个 React 组件，Agent 创建   │      │          │
│   │   │ 了... 遇到了 ESLint 报错，通过修改配置    │      │          │
│   │   │ 解决... 最终完成了功能 A 和 B，C 尚未     │      │          │
│   │   │ 完成...                                   │      │          │
│   │   └──────────────────────────────────────────┘      │          │
│   │   ┌─ kept ids ─────────────────────────────┐       │          │
│   │   │ [msg_045, msg_078, msg_120]             │       │          │
│   │   │ (最近几轮完整的 tool_call +             │       │          │
│   │   │  tool_result + assistant 回复)          │       │          │
│   │   └─────────────────────────────────────────┘       │          │
│   │                                                      │          │
│   │   compact() 然后替换 messages:                       │          │
│   │                                                      │          │
│   │   Before: [sys, msg1, msg2, ..., msg500]             │          │
│   │            100K tokens                               │          │
│   │                                                      │          │
│   │   After:  [sys, summary_msg, msg_045, ..., msg_120]  │          │
│   │            15K tokens                                │          │
│   │                                                      │          │
│   │   注意：summary_msg 是一条特殊的 assistant message，  │          │
│   │   内容就是 LLM 生成的摘要文本                        │          │
│   └──────────────────────────────────────────────────────┘          │
│                                                                       │
│ Step 10: 持久化 + 验证                                                │
│   hardenManualCompactionBoundary() → 写入 JSONL 文件                 │
│   estimateTokensAfterCompaction() → 检查是否仍然超标                 │
│   persistSessionCompactionCheckpoint() → 保存检查点（可回溯）        │
│                                                                       │
│ Step 11: 运行 after_compaction hooks                                  │
│   → memory 插件读取压缩后的状态                                      │
│                                                                       │
│ Step 12: 返回结果给 run.ts 主循环                                     │
│   { ok: true, compacted: true, tokensBefore: 100000,                 │
│     tokensAfter: 15000 }                                             │
│                                                                       │
│   主循环收到后：                                                      │
│   → 更新 activeSession.messages = 压缩后的 messages                  │
│   → 更新系统提示词（因为 tokensBefore/tokensAfter 变了）            │
│   → continue; → 重新发 prompt 给 LLM                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 安全防护

### 1. 防止无限循环：最大重试次数

```typescript
// run.ts:470
const MAX_OVERFLOW_COMPACTION_ATTEMPTS = 3;
const MAX_TIMEOUT_COMPACTION_ATTEMPTS = 2;

// 如果 compact 了 3 次还 overflow → 放弃 compact，走 failover（换模型）
```

### 2. 防止 compaction 干跑：跳过无内容会话

```typescript
// compact.ts:1003-1011
if (!containsRealConversationMessages(session.messages)) {
 // 如果对话历史里没有真正的人机对话（比如全是系统消息）
 → return { compacted: false, reason: "no real conversation messages" };
}
```

### 3. 超时保护

```typescript
// compaction-safety-timeout.ts
const EMBEDDED_COMPACTION_TIMEOUT_MS = 900_000; // 默认 15 分钟超时
// 可配置: agents.defaults.compaction.timeoutSeconds

compactWithSafetyTimeout(compactFn, timeoutMs, {
 abortSignal, // 用户取消
 onCancel: () => activeSession.abortCompaction() // 超时/取消时停掉 LLM 调用
})
```

### 4. 文件锁

```typescript
// compact.ts:782-787
const sessionLock = await acquireSessionWriteLock({
 sessionFile,
 maxHoldMs: resolveSessionLockMaxHoldFromTimeout({ timeoutMs })
});
// 防止多个 Agent 同时 compact 同一个 session 文件，造成数据损坏
```

### 5. 检查点机制

```typescript
// 在 compact 前拍快照，compact 后保存检查点
checkpointSnapshot = captureCompactionCheckpointSnapshot({
 sessionManager, sessionFile
});

// 如果 compact 失败 → 可以通过检查点恢复到 compact 前的状态
// 如果 compact 成功 → 检查点记录完整的前后对比（供审计/调试）
```

---

## Compaction 失败分类

```typescript
// compact-reasons.ts
function classifyCompactionReason(reason) {
 "no real conversation messages" → "no_compactable_entries"
 "token count is below compact threshold" → "below_threshold"
 "already compacted recently" → "already_compacted_recently"
 "context still exceeds target" → "live_context_still_exceeds_target"
 "guard blocked compaction" → "guard_blocked"
 "summary generation failed" → "summary_failed"
 "compaction timed out" → "timeout"
 "HTTP 400/401/403/429" → "provider_error_4xx"
 "HTTP 500/502/503/504" → "provider_error_5xx"
}
```

---

## context engine compaction vs direct compaction

项目里有两套 compaction 机制：

```
SDK Auto Compaction                  OpenClaw Direct Compaction
(pi-coding-agent 库内置)            (compact.ts)
│                                      │
│ 在 session.prompt() 内部            │ 在 prompt() 返回后才触发
│ 自动检测 token 接近上限             │ 处理 "context overflow" 错误
│ 透明地做压缩，OpenClaw 无感         │ 或者用户手动触发
│                                      │
│ context engine 可以接管传统          │ run.ts 根据 attempt.result
│ compaction 流程 →                   │ 判断是否需要 compact
│ runOwnsContextEngine() →            │
│ contextEngine.compact()             │
```

---

## 完整时序总结

```
runEmbeddedPiAgent() while(true) 循环
 │
 ├── runEmbeddedAttemptWithBackend()
 │   └── session.prompt()
 │       │
 │       ├── (SDK 内部: 检测到 token 接近上限)
 │       │   → 自动 compact → 继续 prompt
 │       │
 │       └── 返回结果
 │
 ├── 检查结果
 │   ├── status = "context_overflow"
 │   │   ├── 已经自动 compact 过了? → 再 compact 一次
 │   │   ├── overflowCompactionAttempts < 3? → compact + retry prompt
 │   │   └── overflowCompactionAttempts >= 3? → failover (换模型)
 │   │
 │   └── status = "timeout" + high token usage
 │       ├── timeoutCompactionAttempts < 2? → compact + retry prompt
 │       └── timeoutCompactionAttempts >= 2? → failover
 │
 └── compact 成功 → update session → continue 回到循环顶部
```

**核心智慧**：compaction 就是把「一段很长的对话」变成「一段摘要 + 最近几轮关键消息」，用一个小型 LLM 调用省下未来所有调用的 token 开销。但它本身也有风险——调用 LLM 本身要花钱、花时间，所以有重试上限防护。

---

想接着看什么？

1. 📨 **Subscription 管道** — `subscribeEmbeddedPiSession` 怎么处理流式响应和 tool call
2. 🔄 **Failover 重试逻辑** — timeout/rate limit/auth fail/context overflow 的完整重试矩阵
3. 📁 **Session 文件格式** — JSONL transcript 的读写 / repair / 修复
