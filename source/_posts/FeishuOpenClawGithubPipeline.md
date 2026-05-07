---
title: 飞书 + OpenClaw + GitHub：打造全自动 AI 博客发布流水线
excerpt: 从飞书发一张截图，到博客文章自动上线，整个链路只需要一句话。本文记录这套全链路的搭建过程。
date: 2026-05-07
categories:
  - 技术实践
tags:
  - OpenClaw
  - 飞书
  - GitHub
  - Hexo
  - AI Agent
---

# 飞书 + OpenClaw + GitHub：打造全自动 AI 博客发布流水线

> 从飞书发一张截图，到博客文章自动上线，整个链路只需要一句话。

## 背景

我平时会在飞书里和 AI 聊技术话题，有时候聊着聊着就产出了一些值得记录的内容。但把对话整理成博客文章是个麻烦事——截图、OCR、排版、推送，每一步都要手动操作。

于是我搭了一套全自动的流水线：**飞书 → OpenClaw → GitHub → Hexo 博客**，现在只需要把截图发给 AI，说一句"帮我发布成博客"，几分钟后文章就上线了。

## 整体架构

```
用户（飞书）
    │
    │ 发送截图/文字
    ▼
OpenClaw（AI Agent 运行时）
    │
    ├── 飞书 API 下载图片
    ├── Claude Code OCR 识别文字
    ├── AI 整理排版成 Markdown
    │
    ▼
GitHub Contents API
    │
    │ PUT /repos/{owner}/{repo}/contents/{path}
    ▼
GitHub 仓库（server 分支）
    │
    │ CI/CD 触发
    ▼
Hexo 构建 → GitHub Pages 部署
```

## 各环节详解

### 1. 飞书端：发消息触发流程

OpenClaw 接入飞书机器人后，用户可以直接在飞书私聊中发送：
- 纯文字内容
- 截图（图片消息）
- 图片 + 文字说明

飞书消息会携带 `message_id`，OpenClaw 通过飞书 API 拿到图片的 `image_key`，再下载到本地处理。

```python
# 下载飞书图片到本地
resp = requests.get(
    f"https://open.feishu.cn/open-apis/im/v1/messages/{message_id}/resources/{image_key}?type=image",
    headers={"Authorization": f"Bearer {tenant_access_token}"},
    stream=True
)
```

### 2. OpenClaw：AI Agent 运行时

[OpenClaw](https://github.com/openclaw/openclaw) 是一个 AI Agent 运行时，支持接入飞书、Telegram、Discord 等多个 IM 平台。它负责：

- **消息路由**：接收飞书消息，分发给 AI 处理
- **工具调用**：执行文件下载、代码运行、API 调用等操作
- **上下文管理**：维护对话历史和工作区文件

在这套流水线里，OpenClaw 是整个链路的"大脑"，协调各个环节的执行。

### 3. OCR：用 Claude Code 识别截图

Node 环境没有 PIL/Tesseract，但可以用 Claude Code 的视觉能力来做 OCR：

```bash
npx @anthropic-ai/claude-code \
  --print \
  --permission-mode bypassPermissions \
  "请识别这张图片中的所有文字内容，保持原有格式，输出 Markdown。图片路径：/path/to/image.jpg"
```

Claude Code 支持读取本地图片文件，识别效果远好于传统 OCR，尤其是对中文、代码块、表格的处理。

**注意**：这个代理只允许 Claude Code 客户端调用，不能直接用 `anthropic` Python SDK 调用。

### 4. 内容整理：AI 排版成博客文章

OCR 拿到原始文字后，让 AI 整理成符合 Hexo 格式的 Markdown：

```markdown
---
title: 文章标题
excerpt: 摘要
date: 2026-05-07
categories:
  - 技术实践
tags:
  - AI
  - Agent
---

# 正文内容...
```

### 5. GitHub 推送：Contents API

不需要 git clone，直接用 GitHub Contents API 推送文件：

```python
import requests, base64, json

token = open(".secrets/github_blog_token").read().strip()
content = open("article.md", "rb").read()

resp = requests.put(
    "https://api.github.com/repos/shuangxunian/shuangxunian.github.io/contents/source/_posts/article.md",
    headers={
        "Authorization": f"token {token}",
        "Accept": "application/vnd.github.v3+json"
    },
    json={
        "message": "feat: add new article via OpenClaw",
        "content": base64.b64encode(content).decode(),
        "branch": "server"
    }
)
```

**权限要求**：GitHub PAT 需要 `Contents: Read and write` 权限。

### 6. CI/CD：Hexo 自动构建部署

推送到 `server` 分支后，GitHub Actions 自动触发：

```yaml
# .github/workflows/deploy.yml（示意）
on:
  push:
    branches: [server]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install && npx hexo generate
      - run: npx hexo deploy  # 部署到 GitHub Pages
```

几分钟后，文章就出现在博客上了。

## 实际效果

整个流程的用户体验是这样的：

1. 在飞书里把截图发给 AI
2. 说一句："帮我整理成博客文章发布一下"
3. AI 自动完成 OCR → 排版 → 推送
4. 收到确认消息，博客已更新

从发消息到文章上线，大概 2-3 分钟。

## 踩过的坑

### 坑 1：飞书图片消息的两种形式

- **纯图片消息**（直接拖图片）：消息体里有 `image_key`，可以直接下载
- **图片 + 文字消息**（图片拖到输入框后附带文字）：消息体里只有 `![image]` 占位符，需要先调用 `messages.read` API 拿到原始 post 富文本，再提取 `image_key`

### 坑 2：Anthropic 代理只允许 Claude Code 客户端

直接用 Python `anthropic` SDK 调用代理会报错：
```
{"error": {"message": "No available accounts: this group only allows Claude Code clients"}}
```

解决方案：用 `npx @anthropic-ai/claude-code --print` 来调用，它会被识别为合法的 Claude Code 客户端。

### 坑 3：gateway 和 node 是两个文件系统

OpenClaw 的 `read`/`write` 工具操作的是 gateway 文件系统，`exec` 命令操作的是 node 文件系统。要发送文件给飞书，必须先用 `exec` 在 node 上创建文件，不能用 `write` 工具。

### 坑 4：GitHub PAT 权限

初始创建的 PAT 默认只有 Read 权限，推送时会报 403。需要在 GitHub 设置里把 Contents 权限改为 Read and write。

## 总结

这套流水线的核心思路是：**让 AI 成为各个工具之间的胶水层**。飞书、GitHub、Hexo 本身都有完善的 API，OpenClaw 负责把它们串联起来，用自然语言作为操作界面。

对于有博客但懒得维护的工程师来说，这套方案可以大幅降低发布门槛——有想法就发，不用再纠结排版和流程。

---

*本文由 OpenClaw AI Agent 辅助生成并自动发布。*
