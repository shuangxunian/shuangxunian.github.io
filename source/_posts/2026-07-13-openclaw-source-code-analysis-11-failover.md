---
title: OpenClaw 源码解析 ⑪：Failover 重试逻辑完整拆解
date: 2026-07-13 15:43:00
categories:
- AI技术
tags:
  - OpenClaw
  - Failover
  - Retry Logic
  - LLM
  - Error Handling
excerpt: 当 LLM API 返回 401、429、503 或 context overflow 时，OpenClaw 如何优雅降级——rotate_profile、fallback_model、compaction 的完整决策矩阵。
---

# 🔄 Failover 重试逻辑：完整拆解

## 全局视角：主循环里的两条分支

`while(true)` 主循环里的每一次 LLM 调用，结果只有两种可能导致重试：

```
while (true) {
 runLoopIterations++

 // ── 发 prompt 给 LLM ──
 const attempt = await runEmbeddedAttemptWithBackend({...})

 // 错误发生在什么时候？

 // ← 分支 A：Prompt 阶段错误
 // 请求根本没发出去 / 发出去但 HTTP 直接报错
 // (网络断开、502、API key 无效、rate limit...)
 // attempt.error 有值

 // ← 分支 B：Assistant 阶段错误
 // 请求发出去了，LLM 返回了 assistant message
 // 但内容是错误（stop_reason=error、回复了空文本、连续 tool call 无文本...）
 // attempt.error 为空，但 assistantTexts 有异常
}
```

---

## 分支 A：Prompt 阶段 Failover（在 run.ts 里原地处理）

Prompt 阶段指：`session.prompt()` 抛出异常。即**HTTP 请求直接就失败了**。

```typescript
// run.ts:1335-1455 (简化)

const promptFailoverReason = classifyFailoverReason(errorText, { provider });
// 分类错误 → 返回一个 FailoverReason:
// "auth" → 401, 403
// "rate_limit" → 429
// "overloaded" → 503, 529
// "billing" → 402
// "timeout" → 请求超时
// "model_not_found" → 404
// "format" → 未知的 HTTP 状态码
// "unknown" → 无法分类的错误

// 然后根据是否有 fallback 配置，做不同的决策：
const promptFailoverDecision = resolveRunFailoverDecision({
 stage: "prompt",
 aborted,
 fallbackConfigured, // ← 是否配置了 "如果模型 A 挂了用模型 B"
 failoverReason: promptFailoverReason,
});
```

### Prompt 阶段的三个决策分支

```
没有 fallback 配置          有 fallback 配置
┌──────────────────────┐    ┌──────────────────────────┐
│                      │    │                          │
│ rotate_profile（换 key）│    │ 1. rotate_profile (先换 key)│
│       ↓              │    │       ↓                  │
│   又有新 key?         │    │ 2. 如果换 key 没用:       │
│   ├─ 有 → continue   │    │    fallback_model(换模型) │
│   └─ 没有 →          │    │       ↓                  │
│     surface_error     │    │ 3. 如果换模型也没用:      │
│     → 返回错误给用户    │    │    surface_error         │
│                      │    │    → 返回错误给用户        │
└──────────────────────┘    └──────────────────────────┘
```

### 具体代码

```typescript
// 情况 1: rotate_profile — 换下一个 API key
if (promptFailoverDecision.action === "rotate_profile") {
 const advanced = await advanceAuthProfile();
 // advanced = true → 继续 while 循环，用新 key 重试
 // advanced = false → 降级为 surface_error

 if (advanced) {
 continue; // ← 回到 while(true) 顶部，用新 key
 }
 // 所有 key 都失败 → surface_error
}

// 情况 2: fallback_model — 换模型
if (promptFailoverDecision.action === "fallback_model") {
 // 应用到配置中定义的 "fallback 模型"
 provider = fallbackProvider;
 modelId = fallbackModel;
 continue; // ← 回到 while(true) 顶部，用新模型
}

// 情况 3: surface_error — 放弃重试，返回错误
if (promptFailoverDecision.action === "surface_error") {
 return {
 result: "surface_error",
 error: "无法连接到 AI 服务，请稍后重试..."
 };
}
```

---

## 分支 B：Assistant 阶段 Failover（由 `handleAssistantFailover` 处理）

Assistant 阶段指：`session.prompt()` 返回了，但 LLM 的输出**本身有问题**。

### B1: Context Overflow（上下文溢出）

```typescript
// 优先级最高！在 check assist failure 之前处理

// 溢出恢复步骤 1: 截断 oversized tool results
if (truncateOversizedToolResults) {
 // "这个 tool result 的 stdout 有 500KB，截到 50KB"
 continue; // 重试
}

// 溢出恢复步骤 2: Compaction
if (overflowCompactionAttempts < 3) {
 overflowCompactionAttempts++;
 // 调 compact() → 生成摘要 → 截掉历史
 continue; // 用 compact 后的上下文重试
}

// 溢出恢复步骤 3: 如果 compact 也没用
if (overflowCompactionAttempts >= 3) {
 // 降级到 fallback_model
 // 这说明当前模型 "不够聪明" 或 context window 太小
}
```

### B2: 8 种 Assistant Failover 分类

`handleAssistantFailover()` 根据 LLM 的输出状态做分类：

```typescript
// assistant-failover.ts

const authFailure = isAuthAssistantError(msg); // 401/403 在响应体里
const rateLimitFailure = isRateLimitAssistantError(msg); // 429 在响应体里
const billingFailure = isBillingAssistantError(msg); // 402
const failoverFailure = isFailoverAssistantError(msg); // 503/529
const timedOut = failoverReason === "timeout"; // 请求超时
const idleTimedOut = false; // 模型返回 0 token 的文本
const formatError = cloudCodeAssistFormatError; // 工具调用格式错误
const imageError = imageDimensionError !== null; // 图片太大
```

每种错误的处理逻辑：

```
问题                           决策
─────────────────────────────────────────────────────────────
auth (401/403)
 → rotate_profile (换 API key)
 → 新 key 有效 → continue 重试
 → 所有 key 都 failed → fallback_model 或 surface_error

rate_limit (429)
 → rotate_profile (换 key)
 → 如果换 key 也没用 → fallback_model
 → backoff 等待一段时间再重试

overloaded (503/529)
 → rotate_profile (最多 5 次)
 → 超过 5 次 → fallback_model
 → 换模型前 backoff

billing (402)
 → rotate_profile (换 card / key)
 → 或者 surface_error (告诉用户欠费了)

timeout
 → same_model_idle_timeout_retry (最多 3 次)
 → → 用同一个模型重试（某些模型偶尔会"发呆"）
 → 超过 3 次 → surface_error

idle_timeout (LLM 返回 0 token)
 → 检查是否是 ack_execution_fast_path
 → 如果是 → 用 "你上次没有回复，请给出完整回复" 指令重试
 → 如果不是 → surface_error

thinking_unsupported (模型不支持推理模式)
 → thinkLevel = 降级 → continue

format_error (工具调用格式错误)
 → 清理工具参数 → 用 sanitized tool calls 重试

image_dimension_error (图片太大)
 → 不重试 → surface_error（不回到 while 循环）
```

---

## 完整的 Failover 矩阵

```
┌────────────────┬──────────────────┬──────────────────┬────────────────────┐
│ 错误类型        │ 识别方式          │ 策略 1            │ 策略 2 (fallback)   │
├────────────────┼──────────────────┼──────────────────┼────────────────────┤
│ auth           │ HTTP 401/403    │ rotate_profile   │ fallback_model     │
│ auth_permanent │ 多次 auth 失败   │ surface_error    │ —                  │
│ rate_limit     │ HTTP 429        │ rotate_profile   │ backoff + retry    │
│ overloaded     │ HTTP 503/529    │ rotate_profile   │ fallback_model     │
│ billing        │ HTTP 402        │ rotate_profile   │ surface_error      │
│ timeout        │ 请求超时         │ same_model重试   │ surface_error      │
│ context_overflow│ token 超限      │ compact ×3      │ fallback_model     │
│ thinking       │ 模型不支持推理    │ 降级thinkLevel   │ —                  │
│ format         │ tool_call格式错误 │ sanitize后重试   │ —                  │
│ idle           │ LLM 输出 0 token │ ack快路径重试     │ surface_error      │
│ empty_response │ 空回复           │ 追加指令重试      │ surface_error      │
│ image_size     │ 图片超尺寸        │ surface_error    │ —                  │
│ model_not_found│ HTTP 404        │ surface_error    │ fallback_model     │
└────────────────┴──────────────────┴──────────────────┴────────────────────┘
```

---

## rotate_profile 机制

```typescript
// run.ts:384-405
const { advanceAuthProfile } = createEmbeddedRunAuthController({
 config, authStorage, profileCandidates
});

// advanceAuthProfile() 做了：
// 1. 标记当前 profile 失败（原因: auth/rate_limit/timeout）
// 2. 取下一个 profile
// 3. 重新拿 API key
// 4. 返回 true（有新 key）或 false（所有 key 都失败了）

while (true) {
 const attempt = await runEmbeddedAttemptWithBackend({...});

 if (failoverReason === "auth") {
 const advanced = await advanceAuthProfile();
 if (advanced) {
 lastProfileId = newProfile;
 apiKeyInfo = newKey;
 continue; // ← 用新 key 重试
 }
 // 没有更多 key → surface_error
 }
}
```

---

## fallback_model 机制

在配置里可以指定备选模型：

```yaml
# openclaw.yml
agents:
 defaults:
 model: deepseek/deepseek-chat
 fallbacks:
 - model: openai/gpt-4o # 如果 deepseek 挂了
 - model: anthropic/claude-3-5 # 如果 gpt-4o 也挂了
```

```typescript
// 当 fallback_model 被触发：
provider = "openai";
modelId = "gpt-4o";
lastRetryFailoverReason = failoverReason;
continue; // ← 回到 while(true) 顶部，用备选模型从头开始
```

---

## 函数签名对比

```
resolveRunFailoverDecision({ stage, fallbackConfigured, failoverReason })
 │
 ├── 无 fallback 配置:
 │    auth/rate_limit/overloaded/timeout → rotate_profile
 │    billing → surface_error
 │    unknown → surface_error
 │
 └── 有 fallback 配置:
      rotate_profile 成功后 → continue_normal
      rotate_profile 失败后 → fallback_model
      fallback_model 也失败 → surface_error
```

---

## 关键限制

```
限制项                                    代码位置
─────────────────────────────────────────────────────────
MAX_RUN_LOOP_ITERATIONS = ~20            run.ts:478
 → 无论什么原因，最多重试 20 次

MAX_OVERFLOW_COMPACTION_ATTEMPTS = 3    run.ts:470
 → context overflow 最多 compact 3 次

MAX_TIMEOUT_COMPACTION_ATTEMPTS = 2     run.ts:494
 → timeout 最多 compact 2 次

sameModelIdleTimeoutRetries = 3         run.ts:482
 → 同模型 idle timeout 最多重试 3 次

overloadProfileRotationLimit = 5        assistant-failover.ts:119
 → overload 最多轮换 5 个 profile

rateLimitProfileFallbackLimit           run.ts:516
 → rate limit 最多轮换到指定上限
```

---

## 完整时序图

```
┌─────────────────────────────────────────────────────────────┐
│ while(true) 主循环                                            │
│                                                               │
│ t0 resolveFailedPromptMode() → 添加重试指令到 prompt          │
│ t1 advanceAuthProfile() → 换下一个 API key (如果需要)         │
│ t2 runEmbeddedAttemptWithBackend() → 发请求                  │
│    │                                                          │
│    ├── 成功 → 检查 assistantTexts                             │
│    │    ├── 有正常文本 → 退出循环，返回结果                     │
│    │    ├── 只有 tool calls → 加 "请回复文字" 指令 → continue  │
│    │    ├── 空回复 → idle_timeout → 重试 ×3 → surface_error   │
│    │    └── context overflow → compact ×3 → fallback_model    │
│    │                                                          │
│    └── 失败（抛出异常）                                        │
│         ├── classifyFailoverReason() → auth/rate_limit/...   │
│         ├── resolveRunFailoverDecision() → 决策              │
│         │    ├── rotate_profile → 换 key → continue          │
│         │    ├── fallback_model → 换模型 → continue           │
│         │    ├── surface_error → 返回错误给调用者              │
│         │    └── backoff → 等待 → continue                    │
│         └── 如果 runLoopIterations > 20 → 强制退出            │
└─────────────────────────────────────────────────────────────┘
```

---

## 总结

| 设计原则 | 体现 |
|---------|------|
| **优雅降级** | 先换 key → 不行换模型 → 不行告诉用户 |
| **防止无限重试** | 最多 20 次总重试，每种策略有单独上限 |
| **智能判断** | auth 错误不重试同 key，timeout 可以同模型重试 3 次 |
| **上下文保护** | context overflow 走 compact 而不是换模型（语义不同） |
| **用户可见** | surface_error 时返回人类可读的错误信息 |
