---
title: "OpenClaw 源码解析 ②：Agent 执行引擎"
excerpt: "深入 OpenClaw Agents 模块的四层架构，从 runEmbeddedPiAgent 唯一入口出发，逐层拆解事件驱动引擎、工具系统三层过滤、沙箱安全边界、上下文压缩机制、模型解析管道与子 Agent 生命周期。"
date: 2026-07-10
categories:
- AI技术
tags: [OpenClaw, 源码解析, Agent, LLM]
---

# Agent 执行引擎深度讲解

Agents 模块是 OpenClaw 中规模最大的模块（327K 行，1395 文件），承载了 AI Agent 的完整执行引擎。本文将自顶向下拆解其架构：先从全局四层模型建立整体认知，再逐层深入入口函数、事件管道、工具系统、沙箱、上下文压缩、模型解析和子 Agent 机制。

## 全局架构：四个层级

```
┌────────────────────────────────────────────────────────────────────────────┐
│ Agents 模块架构                                                             │
│                                                                            │
│ ┌──────────────────────────────────────────────────────────────────────┐   │
│ │ L0: 入口与编排                                                        │   │
│ │                                                                      │   │
│ │ runEmbeddedPiAgent() ← 外部调用的唯一入口                            │   │
│ │   ├── sessionLane enqueue ← 同一会话内串行                           │   │
│ │   └── globalLane enqueue ← 全局并发控制 (main/subagent/cron)         │   │
│ │                                                                      │   │
│ │ subagent-spawn.ts ← 主 Agent 内部调用 spawn 子 Agent                  │   │
│ │ cli-runner.ts / agent-runner.ts ← CLI 和 auto-reply 的适配层          │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                     │                                      │
│ ┌─────────────────────────────────▼────────────────────────────────────┐   │
│ │ L1: Agent 执行循环 (run.ts, compact.ts)                              │   │
│ │                                                                      │   │
│ │ while (未完成) {                                                      │   │
│ │   1. compact.ts → 检查上下文是否超限，需要压缩                        │   │
│ │      ├── 调用 LLM 生成摘要                                            │   │
│ │      └── 替换旧历史为摘要 + 关键消息                                   │   │
│ │                                                                      │   │
│ │   2. model.ts → 解析模型/提供商 → 构建 API 调用参数                   │   │
│ │      ├── resolveModel() → 静态模型注册                                │   │
│ │      ├── resolveModelAsync() → 支持插件动态模型                       │   │
│ │      └── attachProviderRequestTransport() → 连接 HTTP/WS              │   │
│ │                                                                      │   │
│ │   3. 调用 LLM API (通过 @mariozechner/pi-ai 库)                      │   │
│ │                                                                      │   │
│ │   4. subscribeEmbeddedPiSession() → 接管流式响应                      │   │
│ │      └── pi-embedded-subscribe.handlers.*                             │   │
│ │          ├── handleMessageStart/Update/End → 解析文本流               │   │
│ │          ├── handleToolExecutionStart/Update/End → 执行工具           │   │
│ │          └── handleCompactionStart/End → 上下文压缩                   │   │
│ │                                                                      │   │
│ │   5. 如果 LLM 返回 tool_calls → 执行工具 → 把结果发回 LLM → 回到 4   │   │
│ │   6. 如果 LLM 返回文本 → 完成，返回结果                               │   │
│ │ }                                                                    │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                     │                                      │
│ ┌─────────────────────────────────▼────────────────────────────────────┐   │
│ │ L2: 工具系统 (pi-tools.ts + tools/)                                  │   │
│ │                                                                      │   │
│ │ createPiTools() → 构建工具清单                                        │   │
│ │   ├── bash-tools (exec, process) ← 执行命令                           │   │
│ │   ├── file-tools (read, write, edit) ← 文件操作                       │   │
│ │   ├── sessions-* (history, list, send) ← 跨会话通信                   │   │
│ │   ├── image-tool ← 图片生成                                           │   │
│ │   ├── canvas-tool ← Canvas 画布操作                                   │   │
│ │   ├── cron-tool ← 定时任务管理                                        │   │
│ │   ├── agents-list-tool ← 查看可用 Agent                               │   │
│ │   ├── pdf-tool ← PDF 处理                                             │   │
│ │   └── channel-tools ← 渠道特定工具                                    │   │
│ │                                                                      │   │
│ │ + tool-policy-pipeline.ts → 多层策略过滤                              │   │
│ │   1. ownerOnly → 只有主人能用的工具                                   │   │
│ │   2. allowlist → 显式允许列表                                         │   │
│ │   3. profile-based → 按角色/配置文件过滤                              │   │
│ │   4. messageProvider → 按消息渠道过滤                                 │   │
│ │   5. subagent → 子 Agent 只能用的工具子集                             │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
│                                     │                                      │
│ ┌─────────────────────────────────▼────────────────────────────────────┐   │
│ │ L3: 基础设施                                                          │   │
│ │                                                                      │   │
│ │ sandbox/ ← Docker/SSH 代码执行沙箱                                    │   │
│ │ skills/ ← 技能加载系统 (本地 + ClawHub)                               │   │
│ │ auth-profiles/ ← API Key 轮换 & 多 Profile failover                   │   │
│ │ pi-hooks/ ← Agent 生命周期钩子                                        │   │
│ │ command/ ← Agent 内命令处理 & 会话管理                                │   │
│ │ harness/ ← Agent 测试 harness                                         │   │
│ │ pi-embedded-helpers/ ← 辅助工具函数                                   │   │
│ │ schema/ ← Agent 配置 schema                                           │   │
│ └──────────────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────┘
```

以上四层构成了完整的 Agent 运行时栈：L0 负责外部调用接入和并发编排，L1 驱动核心的「调用 LLM → 处理响应 → 可能再调用」循环，L2 提供可被 LLM 调用的工具能力并施加多层安全策略，L3 则是支撑这一切的底层基础设施。接下来我们逐层深入。

---

## 一核：`runEmbeddedPiAgent` — 唯一的 Agent 执行入口

整个 Agents 模块对外只暴露一个核心函数，所有 Agent 执行最终都会汇聚到这里：

```typescript
// src/agents/pi-embedded-runner/run.ts:207
export async function runEmbeddedPiAgent(params: RunEmbeddedPiAgentParams) {
  // ...
  const sessionLane = resolveSessionLane(params.sessionKey);
  const globalLane = resolveGlobalLane(params.lane);

  // 双层排队：先排 session lane（同一会话串行），再排 global lane（全局并发）
  return enqueueSession(() => {
    return enqueueGlobal(async () => {
      // ====== 执行前准备 ======
      await ensureOpenClawModelsJson(config, agentDir);  // 模型清单
      ensureRuntimePluginsLoaded(...);                    // 加载插件

      // ====== 模型解析 ======
      // → model.ts: resolveModelAsync()
      // → 通过插件系统解析 provider + model → 构建 API client

      // ====== 主循环 ======
      // → compact.ts: 每次 LLM 调用前检查是否需要压缩上下文
      // → model.ts: 构建请求参数 → 调用 LLM API
      // → pi-embedded-subscribe.ts: 处理流式响应

      // ====== Failover ======
      // 如果失败：auth-profile 轮换 → 换 key 重试
      // 如果 context overflow：自动触发 compaction 后重试
      // 如果 rate limit：退避重试
    });
  });
}
```

**关键设计在于双层排队**。`session:{key}` lane 保证**同一个用户会话**内的消息严格串行处理——用户连续发送多条指令时，后一条必须等前一条完全处理完毕。而 `main` lane 控制**全局**最多 4 个 Agent 并发执行，防止资源耗尽。两层 lane 各自独立，互不阻塞。

---

## 事件驱动引擎：Subscribe / Handlers 管道

Agent 执行中最值得细品的部分是事件处理管道。LLM 的流式响应会产生一连串事件，由一个统一的 handler 管道接管并分发：

```
LLM 流式响应
 │
 ▼
createEmbeddedPiSessionEventHandler()
 │
 ├── "message_start" → handleMessageStart()
 │   └── 初始化流式文本 buffer
 │
 ├── "message_update" → handleMessageUpdate()
 │   ├── 拼接增量的 text token
 │   ├── 检测 <thinking>...</thinking> 标签
 │   ├── 解析 {directive} 指令
 │   ├── 构建代码块索引 (code spans)
 │   ├── 实时 broadcast("agent", delta) → WebUI
 │   └── 累积 reply payload block
 │
 ├── "message_end" → handleMessageEnd()
 │   └── 最终化回复 → 写入 transcript
 │
 ├── "tool_execution_start" → handleToolExecutionStart()
 │   ├── 解析 tool_call → 找到对应工具实现
 │   ├── 检查 tool policy (是否允许)
 │   ├── broadcast("tool.start") → WebUI
 │   └── 执行工具
 │
 ├── "tool_execution_update" → handleToolExecutionUpdate()
 │   └── broadcast("tool.progress") → WebUI
 │
 ├── "tool_execution_end" → handleToolExecutionEnd()
 │   ├── 格式化 tool result → 发回 LLM
 │   └── broadcast("tool.done") → WebUI
 │
 ├── "compaction_start" → handleCompactionStart()
 │   └── 告知客户端正在压缩
 │
 └── "compaction_end" → handleCompactionEnd()
     └── 告知客户端压缩完成
```

每个事件类型由独立的 handler 处理，职责清晰。`message_update` 是最频繁触发的事件——每收到一个 token 就会调用一次，内部做了 thinking 标签检测、指令解析和代码块索引构建等工作，再通过 broadcast 实时推送到 WebUI，用户看到的逐字输出效果就来源于此。

**事件顺序保证**：LLM 的流式回调在底层可能乱序到达，但 handler 通过 `pendingEventChain` 强制串行化。实现非常简洁——利用 Promise 链式排队：

```typescript
// pi-embedded-subscribe.handlers.ts
if (!pendingEventChain) {
  pendingEventChain = run().catch(...).finally(() => { pendingEventChain = null; });
} else {
  pendingEventChain = pendingEventChain.then(() => run()); // 链式排队
}
```

当前没有正在处理的事件时，直接启动新的 Promise 链；否则将新事件追加到链尾。这保证了即使并发到达的事件也会按序执行，不会出现先到的事件因异步操作而被后到的事件插队。

---

## 工具系统：三层架构

工具系统是 Agent 与环境交互的桥梁。OpenClaw 将其拆为三层，每层解决不同维度的问题：

```
┌──────────────────────────────────────────────┐
│ L1: 工具注册 — pi-tools.ts                    │
│                                              │
│ createPiTools() 返回所有可用工具定义           │
│ 每个工具定义为 { name, description, schema,   │
│                  execute }                   │
│                                              │
│ "pi" 指 @mariozechner/pi-ai 库的             │
│ Tool 协议（类似 OpenAI function call）         │
└───────────────────┬──────────────────────────┘
                    │
┌───────────────────▼──────────────────────────┐
│ L2: 策略过滤 — tool-policy-pipeline.ts        │
│                                              │
│ 每层策略逐个过滤工具清单：                      │
│                                              │
│ 1. OwnerOnly → 限定主人的工具                 │
│    e.g. cron-tool, sessions-send             │
│ 2. Allowlist → 显式白名单                     │
│    e.g. config.tools.allow: ["read","exec"]  │
│ 3. Profile → 按角色过滤                      │
│    e.g. subagent 不能 exec                   │
│ 4. MessageProvider → 按渠道过滤               │
│    e.g. WhatsApp 不能用 file_write            │
│                                              │
│ 结果：最终生效的工具清单 → 发给 LLM             │
└───────────────────┬──────────────────────────┘
                    │
┌───────────────────▼──────────────────────────┐
│ L3: 具体实现 — tools/*.ts                     │
│                                              │
│ Bash Tools (最复杂):                          │
│   bash-tools.ts → 入口                       │
│   bash-tools.exec.ts → 命令执行               │
│   bash-tools.process.ts → 进程管理            │
│   bash-tools.schemas.ts → JSON Schema        │
│   bash-tools.exec-host-gateway.ts → 宿主机执行 │
│   bash-tools.exec-host-node.ts → 远程执行     │
│   bash-tools.exec-approval-*.ts → 审批流程    │
│                                              │
│ File Tools:                                  │
│   pi-tools.read.ts → read/write/edit         │
│   ├── 宿主机版 (wrapper)                      │
│   └── 沙箱版 (wrapper)                       │
│                                              │
│ Session Tools:                               │
│   sessions-history-tool.ts → 查看历史         │
│   sessions-list-tool.ts → 列会话             │
│   sessions-send-tool.ts → 跨会话发消息        │
└──────────────────────────────────────────────┘
```

三层职责分明：L1 负责"有什么工具可用"，L2 负责"当前上下文下允许用哪些"，L3 负责"工具具体怎么执行"。策略过滤管道是安全的关键——同一个 Agent 在不同渠道、不同角色下看到的工具清单可以完全不同，LLM 甚至不知道某些工具的存在。

### 沙箱：工具执行的安全边界

工具最终要在某处执行代码或操作文件，沙箱提供了隔离的执行环境：

```
sandbox.ts 导出的核心概念：

sandbox/config.ts → 解析 Docker/SSH 沙箱配置
sandbox/context.ts → 为会话创建沙箱上下文
sandbox/backend.ts → 注册/获取沙箱后端工厂（docker/ssh/...）
sandbox/docker.ts → Docker 容器管理
sandbox/ssh.ts → SSH 远程执行
sandbox/fs-bridge.ts → 沙箱内文件系统操作桥接
sandbox/runtime-status.ts → 运行时状态检查
sandbox/tool-policy.ts → 沙箱工具策略

文件工具会根据是否启用沙箱自动切换：
 沙箱内 → createSandboxedReadTool / createSandboxedWriteTool
 宿主机 → createOpenClawReadTool / createHostWorkspaceWriteTool
```

工具实现层通过 wrapper 模式自动适配执行环境：同一个 `read` 工具，在沙箱内通过 `fs-bridge` 操作容器文件系统，在宿主机则直接访问本地文件。上层调用者无需关心底层差异。

---

## 上下文压缩（Compaction）

Agent 对话可能持续数百轮，累积的 token 数远超 LLM 的 context window 上限。Compaction 机制会在此刻自动介入：

```
compact.ts / compact.queued.ts

触发条件：
 1. Token 用量接近 context window 上限 (overflow compaction)
 2. 配置的超时触发 (timeout-triggered compaction)
 3. 手动触发 (manual compaction)

流程：
 ┌──────────────────────────────────────────┐
 │ 1. 暂停当前 LLM 流                        │
 │                                          │
 │ 2. 调用专门的 "compaction LLM" 生成摘要    │
 │    输入：完整对话历史                      │
 │    输出：摘要 + 保留的关键消息              │
 │                                          │
 │ 3. 替换 session transcript                │
 │    [系统提示词]                            │
 │    [摘要: "用户问了A，AI回答了B..."]        │
 │    [关键消息: 最近3轮对话、重要工具调用结果]  │
 │    → 重新开始 LLM 调用                     │
 │                                          │
 │ 4. 运行 compaction hooks                  │
 │    (memory 插件可以在这里做长期记忆持久化)    │
 └──────────────────────────────────────────┘

compaction-safety-timeout.ts → 防止压缩本身超时卡死
compaction-hooks.ts → 压缩前后的钩子
context-engine-maintenance.ts → 上下文引擎维护（周期清理）
```

压缩的本质是"用一小段摘要替换一整段历史"——摘要保留了对话的语义要点，关键消息保留了最近几轮和重要工具调用的完整内容，其余历史被裁剪。对 LLM 来说，压缩后的上下文是透明的：它看到的是"之前发生了什么"的浓缩版，而非残缺不全的原始对话。

---

## 模型解析管道

OpenClaw 支持多种 LLM provider（OpenAI / Anthropic / Google / 自定义），模型解析管道的职责是：给定一个模型标识，找到对应的 provider、构建正确的 API 请求参数、建立传输通道：

```
model.ts (pi-embedded-runner/)

resolveModel() → 静态模式：根据 provider + model 字符串查注册表
resolveModelAsync() → 支持插件动态模型发现

provider-request-config.ts → 为每个 provider 构建请求配置
 ├── API endpoint URL
 ├── Headers (Authorization, etc.)
 ├── 请求参数 (temperature, max_tokens, tools, etc.)
 └── Transport (HTTP fetch / WebSocket)

model.inline-provider.ts → 支持内联定义的 provider（无需 extension）
 配置文件里直接写 endpoint + api key 就能用

model.provider-normalization.ts → 标准化不同 provider 的模型 ID
model.forward-compat.* → 模型前向兼容检测与适配
```

**Provider 抽象层**的设计值得注意：每个 LLM provider 是一个独立的 extension（openai / anthropic / google / ...），通过 `plugins/provider-runtime.js` 注册。`model.ts` 不直接调用任何具体 API——它通过插件系统间接调用。这意味着添加新的 provider 不需要修改核心代码，只需编写一个新的 extension。

---

## 子 Agent（Subagent）

当主 Agent 需要并行处理独立子任务时，可以 spawn 子 Agent。子 Agent 拥有独立的 session、context 和受限的工具集，完成后将结果返回主 Agent：

```
subagent-spawn.ts → 主 Agent 内部 spawn 子 Agent

子 Agent 是什么？
 → 主 Agent（如代码助手）可以创建子 Agent 去执行独立任务
 → 子 Agent 有自己的 session、context、tools
 → 子 Agent 完成后，结果返回给主 Agent

subagent-registry.ts → 子 Agent 的生命周期管理
 run-manager → 创建/销毁子 Agent run
 registry → 记录所有子 Agent 状态
 memory → 子 Agent 的独立记忆
 cleanup → 清理孤立/死掉的子 Agent
 steer-runtime → 父进程可以 steer（控制）子 Agent

subagent-depth.ts → 控制子 Agent 嵌套深度（默认只允 1 层）
subagent-capabilities.ts → 子 Agent 的能力集（比父 Agent 更受限）
subagent-system-prompt.ts → 子 Agent 专用的系统提示词
subagent-announce.ts → 子 Agent 结果广播
```

子 Agent 的嵌套深度默认限制为 1 层——子 Agent 不能再 spawn 孙子 Agent。能力集也是主 Agent 的严格子集，例如子 Agent 通常不能访问 cron-tool 或跨会话通信工具。这是一个典型的"最小权限"设计。

---

## 各子模块速查

| 子模块 | 行数 | 做什么 |
|--------|------|--------|
| `pi-embedded-runner/` | 48K | 核心执行循环、compaction、streaming、model 解析 |
| `tools/` | 29K | 30+ 种工具的具体实现 |
| `sandbox/` | 11K | Docker/SSH 代码执行沙箱 |
| `auth-profiles/` | 11K | API key 轮换、多 profile failover |
| `pi-hooks/` | 5K | Agent 生命周期钩子 |
| `cli-runner/` | 4K | CLI 适配层 |
| `pi-embedded-helpers/` | 4K | 辅助函数 |
| `skills/` | 4K | 技能加载/管理 |
| `command/` | 4K | Agent 命令 & 会话管理 |
| `harness/` | 1K | 测试 harness |
| `schema/` | 1K | 配置 schema |

加上 `src/agents/` 顶层那 817 个文件（200K 行），主要是 provider 特定的适配逻辑：
- `openai-*.ts` / `anthropic-*.ts` / `google-*.ts` → 各 provider 的差异兼容
- `model-*.ts` → 模型发现、选择、fallback
- `tool-*.ts` → 工具策略、显示、图片处理
- `session-*.ts` → 会话文件管理、transcript 修复
- `subagent-*.ts` → 子 Agent 完整生命周期

---

## 完整执行时序图

```
用户消息到达
 │
 ▼
auto-reply → runEmbeddedPiAgent()
 │
 ├── [排队] sessionLane.enqueue → globalLane.enqueue
 │
 ▼
┌──────────────────────────────────────────────────────┐
│ 1. 准备                                              │
│ ├── resolveModelAsync(model.ts)                      │
│ │   → 查插件注册表 → 找 provider → 构建 API client    │
│ ├── createPiTools(pi-tools.ts)                       │
│ │   → 收集所有工具 → 跑 policy pipeline → 过滤        │
│ ├── 构建系统提示词 (system-prompt.ts)                  │
│ ├── 加载会话历史 + 记忆                                │
│ └── 检查 context window → 是否需要 compact()          │
│                                                      │
│ 2. 调用 LLM                                           │
│ ├── provider-request-config.ts → 构建 HTTP 请求       │
│ ├── @mariozechner/pi-ai → 发送请求                    │
│ └── subscribeEmbeddedPiSession() → 订阅流式响应        │
│                                                      │
│ 3. 事件处理循环                                       │
│ ├── message_update → 文本 token 到达                   │
│ │   → broadcast("agent", token) → WebUI               │
│ ├── tool_execution_start → LLM 要调工具               │
│ │   → pi-tools 里找对应工具 → 执行 → 返回结果          │
│ │   → 把 tool_result 发回 LLM → 回到步骤 2            │
│ ├── message_end → 最终回复完成                        │
│ │   → 写入 transcript → broadcast("chat.done")        │
│ └── compaction_start/end → 上下文超限                 │
│     → compact.ts → 生成摘要 → 替换历史 → 重试         │
│                                                      │
│ 4. 错误处理                                           │
│ ├── auth fail → auth-profiles 轮换 key → 重试        │
│ ├── context overflow → 自动 compaction → 重试        │
│ ├── rate limit → 退避重试                             │
│ └── 不可恢复 → failover-error → 返回错误给用户         │
└──────────────────────────────────────────────────────┘
```

---

## 总结

本文从四层架构出发，逐层拆解了 OpenClaw Agents 模块的核心设计：以 `runEmbeddedPiAgent` 为唯一入口的双层排队机制、基于事件管道和 Promise 链的流式响应处理、注册-过滤-执行三层工具架构与沙箱隔离、自动上下文压缩、插件化的模型解析，以及受限的子 Agent 体系。这些组件协同工作，构成了一个完整的、可扩展的 AI Agent 运行时。

**下一集预告：消息调度系统**——从飞书 / Telegram 等渠道的消息到达开始，经过网关路由、协议适配、会话匹配，到最终送入 Agent 执行引擎的完整链路。我们将看到多渠道消息如何被统一调度和管理。
