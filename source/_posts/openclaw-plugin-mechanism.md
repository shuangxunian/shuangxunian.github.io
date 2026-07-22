---
title: OpenClaw 插件机制科普：一个 Guard 插件是怎么"活"起来的
excerpt: 从插件目录结构、加载流程、钩子订阅到权限模型，拆解 OpenClaw 插件系统的完整运作链路。
date: 2026-07-02
categories:
- AI技术
tags:
  - OpenClaw
  - 插件
  - 架构
---

## 1. 插件目录里到底放了什么？

以 `exec-guard` / `node-guard` 为例，`/data/openclaw/state/extensions/<插件名>/` 目录下固定两个文件：

- **`openclaw.plugin.json`**：插件的"身份证"，声明 `id`、`name`、`description`，OpenClaw 靠 `id` 字段认出这是哪个插件。
- **`index.ts`**：插件真正的逻辑代码，导出一个默认函数。

```json
{
  "id": "exec-guard",
  "name": "Exec Guard",
  "description": "Block dangerous exec commands",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

`configSchema` 是空的，意味着这个插件不接受外部可配置参数——规则都硬编码在 `index.ts` 里。

## 2. 光写文件不会自动加载

很多人以为把文件夹丢进 `extensions/` 目录 OpenClaw 就会自动扫描加载，其实不会。必须再做三件事：

1. 把插件 `id` 写进 `openclaw.json` 的 `plugins.allow` 数组（白名单）
2. 在 `plugins.entries` 里加一条 `{"enabled": true}`
3. 执行 `openclaw plugins enable <id>` 显式激活

光靠 `enabled: true` 不够，必须跑这条命令。

如果插件需要往系统提示词里塞内容（比如 `node-guard`），还要多加一条权限声明 `hooks.allowPromptInjection: true`，并把插件目录路径额外登记进 `plugins.load.paths`，否则加载器根本不会去扫这个目录。

改完配置、enable 完，代码也不会热加载进正在跑的进程，最后还要**重启一次 Gateway** 才真正生效。

## 3. 钩子类型不是靠文件名判断的

这是最容易误解的一点：`index.ts` 这个文件名只是加载器约定的入口文件名（类似 npm 包的 `main` 字段），跟这个插件是什么类型的钩子**完全无关**。

真正决定"这是 `before_tool_call` 钩子还是 `before_prompt_build` 钩子"的，是代码运行时这一行：

```javascript
export default function (api) {
  api.on("before_tool_call", function (event, ctx) {
    // 这个字符串参数决定钩子类型
    // ...
  });
}
```

`api.on(eventName, handler)` 就像 Node.js 里的 `EventEmitter.on()`：OpenClaw 内部维护一张"事件名到监听器列表"的表，插件加载时调用一次你的默认导出函数，你在函数体里调用 `api.on()` 完成"订阅"。之后每次真的发生对应事件（比如 Agent 要调用 `exec` 工具），OpenClaw 就把这个事件 emit 出去，依次执行所有订阅了这个事件名的 handler。

一个插件文件里完全可以同时订阅多个事件（`api.on("before_tool_call", ...)` 又 `api.on("before_prompt_build", ...)`），互不冲突。

## 4. 两种钩子，两种权限等级

| 钩子 | 时机 | 能做什么 | 权限要求 |
|------|------|----------|----------|
| `before_tool_call` | Agent 即将调用一个工具（如 `exec`）时 | 返回 `{block: true}` 可以直接拦截这次调用 | 无额外权限 |
| `before_prompt_build` | 每次组装发给模型的系统提示词时 | 返回 `{prependSystemContext: "..."}` 可以往提示词开头插内容 | 必须在 `openclaw.json` 里显式声明 `allowPromptInjection: true` |

`before_tool_call` 只是"拦截/放行"的判断，风险较低；`before_prompt_build` 能直接改模型看到的系统指令，属于更敏感的能力，所以 OpenClaw 单独加了一道权限门禁。

## 5. 小结：一条命令背后发生了什么

当你执行 `openclaw plugins enable exec-guard` 并重启 Gateway 后：

1. 加载器读 `openclaw.plugin.json` 确认 `id` 合法且在白名单里
2. `import` `index.ts` 的默认导出函数，传入 `api` 对象并调用一次
3. 你的代码调用 `api.on("before_tool_call", handler)`，把 handler 挂进事件表
4. 以后每次 Gateway Agent 发起 `exec` 调用，OpenClaw 都会先 `emit("before_tool_call", ...)`，跑一遍所有订阅者，任何一个返回 `block: true` 就直接拦下这次调用
