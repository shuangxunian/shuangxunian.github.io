---
title: 一句 Hello，Claude Code 为什么可能要处理 3 万 Token？
excerpt: 用 claude-trace 拆开一次 Claude Code 请求：隐藏上下文、系统提示词、工具定义和提示词缓存，分别怎样构成一次看似简单的对话。
date: 2026-08-10
categories:
- AI 编程
tags:
- Claude Code
- Token
- Prompt Cache
- 调试
---

在 Claude Code 里输入一句 `hello`，它回了一句“有什么可以帮你的？”。从人的视角看，这几乎是一轮没有成本的对话。

但从 API 请求的视角看，事情完全不同。在一次实际抓包中，本轮的 `input_tokens` 只有 **6**，而服务端同时记录了约 **1.4 万个缓存写入 token** 和 **1.6 万个缓存读取 token**。把三者相加，模型在这次请求中关联的上下文接近 **3.1 万 token**。

这不表示你输入的五个字母突然变成了三万个 token，而是 Claude Code 为了完成一个看似简单的请求，附带了大量运行上下文。本文用 [Mario Zechner](https://github.com/badlogic) 的 [claude-trace](https://www.npmjs.com/package/@mariozechner/claude-trace) 把一次请求拆开，看看这些 token 到底花在了哪里。

> 本文展示的是某次环境与版本下的抓包结构。Claude Code 的系统提示词、可用工具、MCP、Skills、Hooks 和缓存命中情况都会随版本、项目和配置变化，数字不应被理解为固定值。

## 先抓一份自己的请求

安装并启动 `claude-trace`：

```bash
npm install -g @mariozechner/claude-trace

# 以 Claude Code 启动一次会话，并记录所有 API 请求
claude-trace --include-all-requests
```

之后像平常一样使用 Claude Code，结束会话即可。工具会在当前工作目录生成：

```text
.claude-trace/
  log-2026-08-10-12-34-56.jsonl
  log-2026-08-10-12-34-56.html
```

HTML 是自包含的报告，直接用浏览器打开即可查看请求、响应、工具调用和 token 用量。默认情况下，`claude-trace` 只记录消息数超过两条的 `/v1/messages` 请求；`--include-all-requests` 很重要，因为单轮 `hello` 正是容易被过滤掉的请求。

**注意：报告不是可以随手上传的日志。** 它可能包含项目路径、源代码片段、文件内容、工具输出、MCP 返回值和会话上下文。调试或分享前，应先脱敏并确认其中没有私密信息。

## 一次请求长什么样

在报告的原始请求视图中，重点看四个字段：

```json
{
  "messages": [],
  "system": [],
  "tools": [],
  "thinking": { "type": "adaptive" }
}
```

这不是完整原文，只是便于理解的轮廓。`messages`、`system` 和 `tools` 承载了大部分上下文；`thinking` 与输出配置则影响模型在本轮可使用的推理策略。

## `messages`：你说的 Hello 排在最后

最反直觉的地方是：用户真正输入的 `hello`，通常不是 `messages[0]` 里的唯一内容。

Claude Code 会将一些运行时上下文包装为多个 content block，与用户输入一起发送。一次典型抓包中，可能依次出现：

1. **Hooks 相关配置**：只有启用了对应 Hook 或插件时才会出现。
2. **延迟加载的工具目录**：先告诉模型有哪些工具可用，而不是把每个工具的完整说明都塞进这里。需要时，模型可通过工具搜索机制取得具体定义。
3. **MCP 使用说明**：告诉模型如何发现和调用当前配置的 MCP 服务。
4. **可用 Skills 列表或说明**：项目所启用的工作流、技能会影响这里的内容。
5. **项目指令**：例如仓库里的 `CLAUDE.md`，其中的目录约定、编码规则和工具偏好会被带入上下文。
6. **实际输入**：最后才是用户的 `hello`。

这解释了一个常见现象：Claude Code 之所以“知道”项目规范，并不是因为规范永久存放在模型里，而是因为每次相关请求都把规范作为上下文提供给模型。

从抓包还可以看到，实际用户输入附近常带有 `cache_control`。它相当于给上下文标出一个缓存边界：边界之前相对稳定的内容可以复用；本次输入及其后的动态内容仍需要重新处理。具体 block 的位置和缓存策略会随版本改变，但这个思路是理解成本的关键。

## `system`：Claude Code 的行为说明书

`system` 字段不是一整段普通字符串，而常常是多个文本块组成的数组。它通常包括以下几类信息：

- **请求元信息**：例如客户端版本、入口类型等。
- **身份说明**：模型正在以 Claude Code 的身份工作。
- **行为规则**：如何完成任务、哪些高风险动作需要确认、怎样使用工具、输出风格等。
- **能力与环境说明**：记忆策略、当前 shell、工作目录、操作系统以及其他运行环境信息。

这些内容决定了 Claude Code 与普通聊天窗口的区别。模型不仅要生成一句自然语言回复，还要理解自己在一个什么环境中运行、能调用哪些能力、执行命令时要遵守什么边界。

如果想查看不同 Claude Code 版本的系统提示词和工具定义变化，可以使用同一作者的 [cchistory](https://github.com/badlogic/cchistory)：

```bash
npm install -g @mariozechner/cchistory
cchistory 1.0.0
```

它会提取指定版本发送的用户消息格式、系统提示词与工具定义。对于“上周还好好的，这周为什么行为变了”这类问题，版本对比比凭感觉猜测更可靠。

## `tools`：工具定义本身也是上下文

`tools` 中是 Claude Code 直接提供给模型的工具定义。常见工具覆盖文件读取、内容搜索、文件编辑、Shell 命令执行、子代理任务分派、计划唤醒和工具搜索等能力。

每个工具都不只是一个名字。它还会有：

- 何时适合或不适合使用的说明；
- 参数的 JSON Schema；
- 参数含义和约束；
- 与安全操作相关的规则。

因此，一个功能强大的命令行工具并不会只消耗“用户提示词的 token”。为了让模型正确、安全地调用工具，工具说明本身也必须进入上下文。特别是 Shell 工具的说明往往很长，因为它需要描述工作目录、超时、命令风险和使用约束。

## `thinking` 与 `effort`：决定可用的推理强度

抓包中还可能看到类似：

```json
{
  "thinking": { "type": "adaptive" },
  "output_config": { "effort": "xhigh" }
}
```

`adaptive` 表示模型可自行决定本轮是否、以及如何使用更深入的推理；`effort` 则是客户端提供的推理强度上限或偏好。字段取值与具体模型、套餐和客户端版本有关，不能简单把某个自然语言提示词等同于某个固定的 API 参数。

## 响应不是一整块 JSON，而是 SSE 事件流

Claude Code 收到的响应通常以 SSE（Server-Sent Events）流的形式抵达。浏览器或终端里“逐字出现”的效果，来自服务器持续推送的增量事件。

一轮简单文本回复常见的顺序如下：

```text
message_start
content_block_start
ping
content_block_delta
content_block_delta
content_block_stop
message_delta（携带 stop_reason: end_turn）
message_stop
```

其中，`content_block_delta` 才是“Hello”“How can I help you today?” 这类实际文本片段。`ping` 用于保持连接；`end_turn` 表示模型主动结束了这一轮回答。

## Token 账单应该怎样读

最值得看的字段通常在 `message_start` 的 `usage` 中：

```json
{
  "input_tokens": 6,
  "cache_creation_input_tokens": 14000,
  "cache_read_input_tokens": 16000
}
```

可以这样理解：

| 字段 | 含义 | 示例 |
| --- | --- | --- |
| `input_tokens` | 本轮常规输入，包含用户输入及未命中缓存的部分 | 6 |
| `cache_creation_input_tokens` | 本轮新写入提示词缓存的内容 | 14,000 |
| `cache_read_input_tokens` | 从已有提示词缓存复用的内容 | 16,000 |

在这个例子中，相关上下文总量约为：

```text
6 + 14,000 + 16,000 = 30,006 tokens
```

视频中“接近 3 万 token”的说法，指的就是这个量级。它描述的是本次请求涉及的上下文规模，而不是说 `hello` 本身有三万个 token。

缓存尤其值得区分。缓存读取通常比正常输入便宜得多，因此连续对话的成本并不等于每次都从零开始处理所有系统提示词和工具定义；但缓存写入、缓存有效期、模型定价和缓存命中条件会影响实际费用。要判断“贵不贵”，应以当次响应的 usage 字段和当期官方价格为准，而不是只看上下文 token 相加的总数。

## 用抓包回答，而不是猜测

当 Claude Code 出现以下情况时，`claude-trace` 很适合作为第一手证据：

- 明明输入很短，token 使用却很高；
- 同一提示词在不同项目或不同日期表现不同；
- 想确认 `CLAUDE.md`、MCP、Skills 或 Hooks 是否真的进入了上下文；
- 想区分“缓存没有命中”和“模型输出过长”；
- 想定位某个工具调用、系统提示词或上下文注入带来的行为变化。

结论很简单：一句 `hello` 背后并不只有一句 `hello`。Claude Code 还需要携带项目规则、环境信息、工具契约和会话状态，才能作为编码代理工作。看不见的上下文正是能力的一部分，也正是 token 账单的一部分。下次遇到异常行为或成本波动，先看它实际发了什么、收到了什么，再决定该删配置、缩短指令，还是调整工作流。
