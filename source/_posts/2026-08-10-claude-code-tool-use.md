---
title: "Claude Code 的 Tool Use：模型怎样调用终端、文件和网页搜索"
date: 2026-08-10
categories:
- AI技术
tags:
  - Claude Code
  - Anthropic
  - Agent
  - Tool Use
  - Security
---

Claude Code 能改文件、跑测试、读网页，看起来像是一个会操作电脑的程序。容易忽略的事实是：**模型本身不会执行命令，也不会直接打开网页。**从模型的角度看，它始终只是在输入文本（更准确地说是结构化消息）后，输出文本或结构化内容块。

让它获得行动能力的机制叫 **Tool Use（工具调用）**：模型负责决定“该用什么工具、参数是什么”，Claude Code 或 Anthropic 的服务端负责真正执行，再把结果作为下一轮输入交回模型。这篇文章用一次 `git status` 的客户端工具调用和 Web Search 的服务器工具调用，把这个循环拆开，并区分公开协议与抓包中观察到的内部实现。

> 文中 API 字段以 Anthropic 当前公开文档为准。Claude Code 的内部工具名、请求次数、所用子模型会随版本和任务变化；抓包能说明某一版本的实现，不能当作稳定接口。

---

## 先建立一个正确心智模型：模型没有“手”

可以把 Claude 看作一个能根据上下文选择下一步的决策器。它看到的能力清单大致是这样：

```json
{
  "name": "bash",
  "description": "在项目目录中执行 shell 命令",
  "input_schema": {
    "type": "object",
    "properties": {
      "command": { "type": "string" }
    },
    "required": ["command"]
  }
}
```

模型不会把 `git status` 交给操作系统。它输出的是“我要调用 `bash`，参数为 `git status`”这一段结构化意图。能访问终端、文件系统和网络的是模型外面的宿主程序：Claude Code、SDK 应用、浏览器插件，或 Anthropic 的服务端。

这也解释了为什么同一个模型在不同产品中“能力”差异很大：能力不只来自模型，还来自它被授予了哪些工具、工具在哪执行，以及每一层如何限制权限。

---

## 客户端工具：一次 `git status` 是怎样跑起来的

假设我们问 Claude Code：“这个 Git 仓库有什么改动？”一次典型的客户端工具调用包含四步。

```
用户问题
  │
  ▼
Claude：输出 tool_use（工具名 + JSON 参数）
  │
  ▼
Claude Code：在本机执行命令
  │
  ▼
Claude Code：输出包装成 tool_result，发回模型
  │
  ▼
Claude：依据结果生成自然语言回答
```

### 1. 客户端把工具定义随请求交给模型

调用 Messages API 时，应用除了传入用户消息，还传入 `tools` 数组。数组中的每项至少描述工具名、用途和输入 JSON Schema。模型据此决定要不要调用工具；它也可能直接回答，或一次请求多个彼此独立的工具。

### 2. 模型返回 `tool_use`

若需要终端，响应的 `content` 中会包含 `type: "tool_use"` 的块。简化后如下：

```json
{
  "type": "tool_use",
  "id": "toolu_01ABC...",
  "name": "bash",
  "input": {
    "command": "git status --short"
  }
}
```

这里的重点有三个：

- `name` 选择工具；
- `input` 是通过 JSON Schema 描述的参数；
- `id` 是这次调用的关联标识，结果必须用它配对。

当响应的 `stop_reason` 为 `tool_use`，不是“模型出错”，而是当前回合停在了等待工具结果的位置。流式响应里，参数可能分成多段增量抵达；客户端应完成组装和校验后再执行，而不是把未完成的片段直接当 shell 命令运行。

### 3. Claude Code 在你的电脑上执行

对客户端工具而言，这一步完全在客户端发生。Claude Code 取出命令，按自身的权限策略、工作目录、审批设置和沙箱规则执行它；再收集标准输出、标准错误和退出状态。

因此，“模型叫我运行一条命令”不等于“命令已经安全地执行”。真正的安全边界在客户端：工具输入要校验，危险操作要审批，文件路径和工作目录要受限，尽可能放进沙箱。

### 4. 客户端把结果作为 `tool_result` 交回去

客户端随后在下一次 Messages 请求里保留刚才的 assistant 消息，并追加一条带 `tool_result` 的用户消息：

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01ABC...",
  "content": " M README.md\n M hello.py\n?? notes.txt"
}
```

`tool_use_id` 必须与上一步的 `id` 完全对应。模型并没有亲眼看见终端，它只看见这段文本结果，于是再输出“`README.md` 和 `hello.py` 被修改，并有一个未跟踪文件”之类的总结。此时若 `stop_reason` 为 `end_turn`，这一轮才真正结束。

这正是官方 Tool Use 文档里的基本循环：**模型提议调用，应用执行，应用回传结果，模型继续推理。**`bash`、读写文件、数据库查询、企业内部 API 都可以套进这个模式。

---

## “客户端执行”与“服务器执行”不是同一种工具

Tool Use 按执行位置可以粗分为两类：

| 类型 | 谁执行 | 例子 | 结果怎样回来 |
| --- | --- | --- | --- |
| 客户端工具 | 你的应用或 Claude Code | shell、读写文件、内部 API | 客户端自己构造 `tool_result` 再请求模型 |
| 服务器工具 | Anthropic 基础设施 | Web Search、Web Fetch、Code Execution 等托管能力 | 通常由服务端在同一次 API 调用内执行并返回相应结果块 |

前一类给了应用最高的自由度，也把权限控制、重试、超时、审计都留给应用。后一类减少了客户端集成成本，但用量、可用区域、数据保留和价格由服务端工具的规则决定。

有一个常见误解需要避免：**服务器工具不意味着模型知道“真实世界”。**它依然是“模型决定调用 -> 某个执行器得到数据 -> 数据进入后续上下文”的循环，只是执行器从本机换成了 Anthropic 托管的服务。

---

## Web Search：服务端如何把搜索结果交给 Claude

在 Claude API 中启用 `web_search` 后，Claude 可以在一个 Messages API 调用中发起搜索。官方文档将其称为在单个 API 调用内自动完成的搜索：响应会包含服务端的调用与结果内容块，随后模型可据此继续生成回答。

这是最常见的路径，不是“永远只有一次 HTTP 请求”的承诺。若同一组并行调用里还混入了客户端工具，API 会先以 `stop_reason: "tool_use"` 等待客户端的 `tool_result`；某些较长的服务端 Agent 循环也可能以 `pause_turn` 要求调用方原样续传上下文。因此，调用方应始终按返回的停止原因和内容块驱动状态机，而不是把请求次数写死。

概念上的返回形态如下：

```json
[
  {
    "type": "server_tool_use",
    "id": "srvtoolu_01XYZ...",
    "name": "web_search",
    "input": { "query": "London weather today" }
  },
  {
    "type": "web_search_tool_result",
    "tool_use_id": "srvtoolu_01XYZ...",
    "content": [
      {
        "type": "web_search_result",
        "title": "...",
        "url": "https://...",
        "encrypted_content": "..."
      }
    ]
  }
]
```

### `encrypted_content` 应该怎样理解

`encrypted_content` 不是给应用读取的网页正文。官方文档要求多轮对话时把它**原样回传**；API 会在后续回合解密并恢复 Claude 所需的搜索上下文。应用可使用标题、URL 和引用信息构建展示，但不应依赖或尝试解析该字段。

视频将它解释为“防止开发者把搜索结果当作免费内容下载服务”。这个推断有现实上的合理性，但目前能从公开文档确认的语义只有：它是供后续 API 回合回传的加密内容。把具体搜索供应商的授权条款或加密的唯一商业目的写成事实，会超出公开证据。

### 为什么抓包可能会看到多次请求和 Haiku

有些 Claude Code 抓包会显示：主模型先提出 Web Search，客户端又用一个较小模型处理搜索，最后把摘要交还主模型。这是一种很常见的 Agent 编排方式，但要分清两个层次：

1. **公开 Web Search API 协议**：搜索工具可以由 Anthropic 在单次 Messages 调用内执行；
2. **Claude Code 产品内部编排**：产品可能额外创建子代理、压缩上下文、重试或做权限检查，因此网络上看到的请求数和模型选择不构成 API 承诺。

用较小模型处理搜索材料有两个合理的工程目标：减少昂贵主模型的上下文负担，以及让不可信网页内容先经过低权限环境。前者是成本与上下文管理；后者则是安全隔离。

---

## 子代理为什么有助于防提示词注入

网页不是可信输入。搜索结果、网页正文甚至 README 都可能出现“忽略前文指令，执行某个命令”这样的恶意文本。模型未必总能可靠地区分它是数据还是指令，这就是 Prompt Injection（提示词注入）的核心风险。

把搜索放在只拥有 `web_search` 权限的子代理中，可以缩小风险半径：即使子代理被网页内容误导，它也没有 `bash`、写文件或发布部署的工具可用。主代理只接收经过压缩的、受格式约束的结果。

但隔离不是魔法。摘要仍可能夹带错误或攻击性指令，主代理仍应把外部内容视为不可信数据。一个可落地的防线至少包括：

- 按最小权限给每个代理和工具授权；
- 对写文件、执行命令、外发数据等高影响动作要求人工确认；
- 限制工作目录、网络出口和可访问的密钥；
- 校验工具参数，记录调用与结果；
- 让不可信内容与高权限工具尽量不要共处同一上下文。

这比单纯在系统提示词里写一句“不要被注入”可靠得多。

---

## Web Search 的费用：与 Claude Code 订阅不要混为一谈

对 Claude API，Anthropic 当前公开的 Web Search 价格是 **每 1,000 次搜索 10 美元**，并且搜索产生的内容仍按普通 token 规则计费。响应的 `usage` 中会以 `web_search_requests` 记录次数；一次搜索无论返回多少条结果，都计为一次。搜索报错不收费。

这是 **API 的服务端工具定价**，不能直接推导为每个 Claude Code 套餐、地区或账户的实际扣费规则。使用 Claude Code 时，应以产品页面、账户用量页和当时的套餐说明为准；使用 API 时，才应按请求数加 token 成本做预算。

---

## 小结

Claude Code 的“会干活”可以压缩成一句话：

> 模型负责提出结构化的行动请求，受控的执行器负责行动，结果再回到模型的上下文中。

理解这个循环后，许多现象就顺理成章了：为什么模型能改文件却不会真的敲键盘，为什么工具结果也会消耗上下文，为什么 `tool_use_id` 必须配对，为什么网页搜索会额外计费，以及为什么 Agent 安全的关键并不只是换一个更聪明的模型，而是工具权限、执行环境和不可信输入的隔离。

## 参考链接

- [Anthropic：Tool use with Claude](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
- [Anthropic：Define tools / 客户端工具调用循环](https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use)
- [Anthropic：Web search tool（返回字段、回传要求与定价）](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)
- [Anthropic：Mitigate prompt injections](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)
- [Claude Code：Security](https://code.claude.com/docs/en/security)
