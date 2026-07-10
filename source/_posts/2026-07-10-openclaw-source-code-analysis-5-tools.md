---
title: "OpenClaw 源码解析 ⑤：工具系统深度拆解"
date: 2026-07-10
categories:
  - 技术
tags:
  - OpenClaw
  - 源码解析
  - Agent
  - Tools
  - Security
---

# Stage 3 深入：工具系统 — `createOpenClawCodingTools` 完整拆解

OpenClaw 源码解析系列第五篇。上一篇拆解了 `runEmbeddedAttempt` 的七个执行阶段，这一篇聚焦其中最复杂的 Stage 3——工具是如何构建、过滤、包装，最终发给 LLM 的。

---

## 工具从哪里来？一张洋葱图

```
 ┌──── 最终发给 LLM 的工具清单 ────┐
 │     (30-50 个工具定义)           │
 │                                  │
 │  ┌─ 层 7: abortSignal 包装 ───┐  │
 │  │                             │  │
 │  │  ┌─ 层 6: hook 包装 ─────┐  │  │
 │  │  │  wrapToolWithBefore    │  │  │
 │  │  │  ToolCallHook          │  │  │
 │  │  │                        │  │  │
 │  │  │  ┌─ 层 5: 归一化 ────┐ │  │  │
 │  │  │  │ normalizeToolParams │ │  │  │
 │  │  │  │ (不同provider格式)  │ │  │  │
 │  │  │  │                    │ │  │  │
 │  │  │  │  ┌─ 层 4: 策略管道┐│ │  │  │
 │  │  │  │  │ Pipeline filter ││ │  │  │
 │  │  │  │  │ 8 层 policy    ││ │  │  │
 │  │  │  │  │                ││ │  │  │
 │  │  │  │  │  ┌─ 层 3 ────┐ ││ │  │  │
 │  │  │  │  │  │OwnerOnly  │ ││ │  │  │
 │  │  │  │  │  │MessageProv│ ││ │  │  │
 │  │  │  │  │  │ModelProv  │ ││ │  │  │
 │  │  │  │  │  │           │ ││ │  │  │
 │  │  │  │  │  │  ┌─ 层2 ─┐│ ││ │  │  │
 │  │  │  │  │  │  │OpenClaw   ││ ││ │  │  │
 │  │  │  │  │  │  │Tools  ││ ││ │  │  │
 │  │  │  │  │  │  │       ││ ││ │  │  │
 │  │  │  │  │  │  │┌层 1 ┐││ ││ │  │  │
 │  │  │  │  │  │  ││Base │││ ││ │  │  │
 │  │  │  │  │  │  ││Tools│││ ││ │  │  │
 │  │  │  │  │  │  │└─────┘││ ││ │  │  │
```

不用被这张图吓到，我们一层一层拆开看。

---

## 层 1：Base Tools（来自 pi-coding-agent 库）

```typescript
const base = createCodingTools(workspaceRoot);
// 来自外部库，提供最基础的文件操作工具模板
// → read, write, edit, bash
```

OpenClaw **并不直接使用**这些模板。每个工具都被替换或重新包装了：

| 库给的模板 | OpenClaw 做了什么 |
|-----------|------------------------------|
| `read` | 替换为支持图片和 PDF 的增强版本 |
| `write` | 替换为带 workspace 边界检查的版本 |
| `edit` | 替换为带 workspace 边界检查的版本 |
| `bash` | **完全丢弃**，换成自己的 `exec` 工具 |

如果启用了沙箱，文件工具还会被进一步替换：

```typescript
if (sandboxRoot) {
  // read  → 沙箱版本（通过容器命令读文件）
  // write → 沙箱版本（通过容器命令写文件）
  // edit  → 沙箱版本（通过容器命令编辑文件）
}
```

---

## 层 2：OpenClaw Tools（业务层）

OpenClaw 自己的工具清单，跟"代码编辑"无关，是 AI 助手的通用能力：

- **代码执行**：exec（shell 命令）、process（后台进程管理）、apply_patch（diff patch）
- **媒体生成**：图片生成、音乐生成、视频生成、图片分析、PDF 处理
- **通信**：消息发送、远程设备调用、Canvas 画布
- **会话管理**：列出会话、查看历史、跨会话通信、创建新会话、子 Agent 管理
- **AI/搜索**：web_search、web_fetch
- **调度**：定时任务、网关控制、Agent 列表
- **TTS**：语音合成
- **规划**：更新执行计划
- **插件**：MCP 服务器注册的工具 + 第三方插件

---

## 层 3-4：策略过滤管道（安全核心）

所有工具构建完成后，经过**多层策略过滤**，把不该暴露的工具剔除：

```
第 1 步: filterToolsByMessageProvider()
 │  示例：WhatsApp 消息来的 → 移除 file_write
 │
第 2 步: applyModelProviderToolPolicy()
 │  示例：本地小模型 → 移除 browser, cron, message（它处理不了太复杂的工具）
 │
第 3 步: applyOwnerOnlyToolPolicy()
 │  非 owner 发来的消息 → 移除所有 ownerOnly 工具（如 cron, gateway）
 │
第 4 步: applyToolPolicyPipeline() — 8 层嵌套策略
 │
 ├── profilePolicy        ← 配置文件里的 profile 定义
 ├── providerProfilePolicy ← 按 provider 特定的 profile
 ├── globalPolicy         ← 全局 tools.allow / tools.deny
 ├── globalProviderPolicy ← 全局按 provider 的策略
 ├── agentPolicy          ← Agent 级别策略
 ├── agentProviderPolicy  ← Agent + Provider 组合策略
 ├── sandboxToolPolicy    ← 沙箱策略
 └── subagentPolicy       ← 子 Agent 策略
```

每一层都可以 allow 或 deny 工具，效果可叠加：

```yaml
# 配置示例：
tools:
  profile: safe             # 只允许安全工具
  allow:
    - read
    - exec
  deny:
    - gateway               # 禁止控制网关
    - cron                  # 禁止定时任务

agents:
  list:
    - id: code-review
      tools:
        allow: [read, exec, web_search]  # 只给这3个
```

---

## 重点深入：`exec` 工具（最复杂的一个）

`exec` 是 Agent 执行 shell 命令的入口。**一个工具，1800 行代码**。

### 执行流程

```
LLM 返回 tool_call: exec({ command: "npm test", timeout: 30 })
 │
 ▼
┌─ exec.execute() ──────────────────────────────────────────────┐
│                                                                │
│ 1. 解析参数                                                    │
│    command = "npm test"                                        │
│    timeout = 30s                                               │
│    host = "auto"                                               │
│    security = "full"                                           │
│                                                                │
│ 2. 解析执行目标 (host)                                          │
│    resolveExecTarget() → 决定在哪里跑：                         │
│    ┌──────────────────────────────────────┐                   │
│    │ host="sandbox"  → Docker 容器内       │                   │
│    │ host="gateway"  → 本地宿主机          │                   │
│    │ host="node"     → 远程 Node 设备      │                   │
│    │ host="auto"     → 有沙箱用沙箱，否则本地│                  │
│    └──────────────────────────────────────┘                   │
│                                                                │
│ 3. 安全审查                                                    │
│    ┌──────────────────────────────────────────────────────┐   │
│    │ security="deny"      → 拒绝所有命令                   │   │
│    │ security="allowlist" → 只允许白名单里的命令            │   │
│    │ security="full"      → 允许任何命令                   │   │
│    │                                                      │   │
│    │ ask="off"      → 直接执行                             │   │
│    │ ask="on-miss"  → 白名单外的才问用户                    │   │
│    │ ask="always"   → 每次都问用户确认                      │   │
│    └──────────────────────────────────────────────────────┘   │
│    ↓                                                           │
│    processGatewayAllowlist() →                                  │
│    分析命令 (analyzeShellCommand)                               │
│    查 exec-approvals.json → 是否在白名单中？                    │
│    如果需要审批 → 发 approval request → 等用户确认              │
│                                                                │
│ 4. 环境变量构建                                                 │
│    ┌──────────────────────────────────────────────────────┐   │
│    │ 沙箱 → buildSandboxEnv()                              │   │
│    │ 宿主机 → sanitizeHostExecEnv()                        │   │
│    │   ├── 阻止覆盖 PATH（安全）                            │   │
│    │   ├── 阻止注入危险变量                                 │   │
│    │   ├── 合并 shell 登录环境                              │   │
│    │   └── 应用 pathPrepend                                │   │
│    └──────────────────────────────────────────────────────┘   │
│                                                                │
│ 5. 执行命令                                                    │
│    runExecProcess({                                            │
│      command: "npm test",                                     │
│      workdir: "/home/user/project",                           │
│      env: {...},                                              │
│      timeoutSec: 30,                                          │
│    })                                                         │
│    → 内部用 child_process.spawn() 或容器命令                   │
│    → 收集 stdout + stderr                                      │
│                                                                │
│ 6. 处理结果                                                    │
│    ┌──────────────────────────────────────────────────────┐   │
│    │ 命令完成 → 返回 stdout / stderr / exitCode             │   │
│    │ 超时     → 终止进程 → 返回已有输出 + timeout 标记       │   │
│    │ yield    → 后台化 → 返回 session ID                   │   │
│    │   "Command still running (session abc, pid 1234)"    │   │
│    │   后续用 process 工具管理                               │   │
│    └──────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────┘
```

### 核心设计：yield / background

这是 exec 工具特有的精巧设计：

```
普通模式：
 LLM: exec("npm test") → 等待完成 → 拿到结果

yield 模式（默认 10 秒超时自动 yield）：
 LLM: exec("npm run dev", { yieldMs: 10000 })
 │
 ├── 命令在 10 秒内完成 → 直接返回结果
 │
 └── 命令超过 10 秒还在跑 → 自动后台化
      → 返回 "Command still running (session X)"
      → LLM 之后可以用 process 来查状态

background 模式：
 LLM: exec("npm run dev", { background: true })
 → 立即后台化 → 返回 session ID
```

---

## 一个工具定义长什么样（发给 LLM 的格式）

```json
{
  "name": "exec",
  "description": "Execute a shell command. Prefer this over all other...",
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
        "description": "Timeout in seconds"
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
```

---

## 完整工具清单速查表

| 工具名 | 来源 | 用途 | 需Owner? |
|--------|------|------|---------|
| `read` | base→包装 | 读取文件（支持图片/PDF/目录） | |
| `write` | base→包装 | 创建/覆写文件 | |
| `edit` | base→包装 | 精确替换文件内容 | |
| `glob` | base | 匹配文件路径 | |
| `grep` | base | 搜索文件内容 | |
| `exec` | 自建 | 执行 shell 命令 | |
| `apply_patch` | 自建 | OpenAI diff patch 格式 | |
| `web_search` | 自建 | 搜索互联网 | |
| `web_fetch` | 自建 | 抓取网页内容 | |
| `image` | 自建 | 分析图片 | |
| `image_generate` | 自建 | 生成图片 | |
| `video_generate` | 自建 | 生成视频 | |
| `music_generate` | 自建 | 生成音乐 | |
| `pdf` | 自建 | 读取 PDF | |
| `tts` | 自建 | 文字转语音 | |
| `message` | 自建 | 向渠道发送消息 | |
| `canvas` | 自建 | Canvas 画布操作 | ✅ |
| `cron` | 自建 | 定时任务管理 | ✅ |
| `gateway` | 自建 | 网关控制 | ✅ |
| `nodes` | 自建 | 远程设备控制 | ✅ |
| `agents_list` | 自建 | 列出可用 Agent | |
| `sessions_list` | 自建 | 列出会话 | |
| `sessions_history` | 自建 | 查看会话历史 | |
| `sessions_send` | 自建 | 跨会话发消息 | ✅ |
| `sessions_spawn` | 自建 | 创建新会话 | ✅ |
| `sessions_yield` | 自建 | 暂停当前执行 | |
| `session_status` | 自建 | 当前会话状态 | |
| `subagents` | 自建 | 管理子 Agent | |
| `update_plan` | 自建 | 更新执行计划 | |
| `...pluginTools` | 插件提供 | MCP / 第三方工具 | 看配置 |

---

## 安全设计总结

整个工具系统的安全理念是**多层纵深防御**：

```
用户消息 → Agent → LLM 决定调用工具
 │
 ▼
┌─────────── 第 1 层：工具清单过滤 ──────────────┐
│ 你根本看不到被禁止的工具（LLM 不知道它存在）     │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│ 第 2 层：参数验证（JSON Schema）                │
│ 参数类型/格式不对 → 直接拒绝                    │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│ 第 3 层：workspace 边界检查                     │
│ read/write/edit 只能操作 workspace 内的文件      │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│ 第 4 层：exec 审批系统                          │
│ 高危命令 → 需要用户确认才能执行                  │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│ 第 5 层：沙箱隔离                              │
│ 命令在容器内运行，即使出事也不影响宿主机          │
└───────────────────────┬───────────────────────┘
                        │
┌───────────────────────▼───────────────────────┐
│ 第 6 层：环境变量净化                            │
│ 禁止覆盖 PATH、注入敏感变量                      │
└───────────────────────────────────────────────┘
```

---

## 总结

工具系统是 Agent 和 LLM 之间的"能力接口"。它的设计精髓在于三个关键词：

- **洋葱模型**——从 base tools 到最终发给 LLM 的清单，经过 7 层包装和过滤
- **策略管道**——8 层可叠加的 policy，让每个 Agent 只看到该看的工具
- **纵深防御**——从清单过滤到沙箱隔离，6 层安全防线层层递进

理解了工具系统，你就理解了 Agent 的"手"是怎么长出来的。

---

## 下一步

工具系统拆解完了。接下来的可选方向：

- **订阅管道**：LLM 返回流式响应后，subscription 怎么逐个处理 token 和 tool call
- **系统提示词**：完整内容到底塞了什么给 LLM
- **Failover / 压缩**：异常怎么恢复，上下文满了怎么压缩

下一篇将从其中一个方向深入。

---

> 本系列文章基于 OpenClaw v2026.4.23 源码分析。
