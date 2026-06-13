---
title: "PilotDeck：OpenBMB 推出的 AI Agent 任务调度框架"
excerpt: "PilotDeck 是由清华大学 THUNLP、面壁智能与 OpenBMB 联合开源的以 WorkSpace 为核心的智能体操作系统，通过白盒记忆、智能路由与 Always-on 三大能力，为多项目并行的 AI Agent 生产力场景提供了全新解决方案。"
date: 2026-05-29
categories:
- AI技术
tags: [AI Agent, OpenBMB, PilotDeck, 任务调度, LLM]
---

## 引言

2026 年，AI Agent 领域进入了「从能用到好用」的关键转折期。Claude Code、Cursor、Codex 等工具已经证明大模型可以深度参与编程与知识工作，但当我们把场景从「单次对话」拉长到「多项目并行的长周期生产力创作」时，一系列核心问题浮出水面：记忆不透明、成本不可控、离线后任务停滞、多项目间上下文互相污染。

2026 年 5 月 28 日，由清华大学 THUNLP 实验室、面壁智能（ModelBest）、OpenBMB 与 AI9Stars 联合研发的开源项目 **PilotDeck** 正式发布。它以「WorkSpace（工作舱）」为核心抽象，提出了一套面向生产力场景的智能体操作系统方案，试图从架构层面回答上述问题。

## 项目背景：AI Agent 的下一个瓶颈在哪里？

当前主流 AI Agent 工具各有侧重：Claude Code / Cursor / Trae Solo 将模型推理能力深度集成进编程 IDE；Claude Cowork 引入了项目隔离概念，把 Agent 带到桌面知识工作场景；WorkBuddy 打通 IM 生态，让 AI 在企业微信、飞书中触手可及。

然而，当开发者需要同时推进多个项目——一边写游戏逻辑、一边做数据报告、一边运营自媒体内容——现有工具暴露出四个结构性短板：

1. **记忆黑盒**：AI 记住了什么、为什么记错，用户无从得知，更无法修改。
2. **成本失控**：所有任务一律调用最贵的旗舰模型，简单的排版润色也在烧 Opus 级别的 Token。
3. **上下文污染**：多项目共享同一个记忆池，A 项目的文风偏好可能「串台」到 B 项目。
4. **人走即停**：用户关闭电脑后，Agent 随之停止，无法自主推进长周期任务。

PilotDeck 的设计正是围绕这四个痛点展开的。

## 核心架构：以 WorkSpace 为原子单位

PilotDeck 的架构哲学可以用一句话概括：**一切以 WorkSpace 为边界**。每个 WorkSpace 就像一个独立的「工作舱」，拥有自己专属的文件系统、记忆库与技能集（Skills）。多个 WorkSpace 可以并行运行，彼此完全隔离。

在此基础上，PilotDeck 构建了三大支柱能力：

### 1. 白盒记忆（White-box Memory）

与传统 Agent 的黑盒记忆不同，PilotDeck 的记忆系统全链路可见——从记忆的生成、抽取、存储到使用，每个环节都可以查看和修改。当 AI 「记错」时，用户可以直接定位到出错的记忆条目并手动修正，而不必重开会话从头来过。

更值得一提的是内置的 **Dream 模式**：系统会利用空闲时间自动归纳整理记忆（类似人类的「做梦」过程），并且支持一键回滚到整理前的状态，避免「越整理越乱」的问题。

与黑盒 Agent 相比，PilotDeck 在可见性、可控性、可追溯、隔离性和可回滚五个维度上都有本质提升。例如，黑盒 Agent 的记忆写入后无法修改删除，上下文压缩后原始内容丢失；而 PilotDeck 按 WorkSpace 隔离记忆，支持随时编辑、删除和回滚。

### 2. 智能路由（Smart Routing）

PilotDeck 内置了任务难度识别机制。复杂的规划、推理任务自动分配给旗舰模型（如 Claude Sonnet 4.6、GPT-4o），而简单的文本润色、排版等任务则降级到轻量模型（如 MiniMax-M2.7），通过端云协同与精准匹配大幅降低 Token 消耗。

实测数据非常亮眼。在小红书等社媒运营场景中，开启智能路由后，主 Agent 使用 Opus 4.5、子 Agent 使用 Sonnet 4.5 的方案，费用仅 $2.83，而全部使用 Opus 4.5 的方案费用高达 $12.58——**节省约 70% 的成本**。

在更复杂的测试中（包括多语言播客推送、多源数据报告、领域论文综述、代码库架构文档等 7 个任务），采用「主 Sonnet 4.6 + 子 MiniMax-M2.7」的路由编排，以仅 $3.15 的成本取得了 70.6 分的综合评分，超越了单独使用 Claude Sonnet 4.6（$18.36，69.1 分）的表现——**1/6 的成本，更好的效果**。

### 3. Always-on 常驻执行

PilotDeck 突破了传统「你问我答」的交互模式。用户离开后，Agent 仍能在后台持续运行：主动发现潜在任务、执行长周期监控、将成果落地为本地文件，并生成摘要汇报等待用户回来查阅。这使得将 Agent 作为后台常驻助手成为真正可行的方案。

## 主要功能一览

除了三大支柱能力，PilotDeck 还提供了完整的工程化支撑：

- **原生 MCP 支持**：全面兼容 Model Context Protocol，可以无缝集成任何 MCP 服务器。
- **多前端一致性**：Web UI、CLI 和 IM（企业微信/飞书）三端行为一致，团队成员可以各取所需。
- **开放插件架构**：通过 `plugin.json` 即可扩展系统能力，支持自定义 Tools & Skills、生命周期钩子（如 `PreToolUse`、`UserPromptSubmit`）、自定义记忆存储 Provider 等。
- **多模型协议支持**：兼容 OpenAI、Anthropic、DeepSeek、Qwen、Kimi、MiniMax 等多种模型提供商。
- **开箱即用的 Web UI**：支持完整的 WorkSpace 管理、白盒记忆编辑与多智能体协作过程可视化。

## 快速上手

PilotDeck 提供了三种安装方式，适配不同场景：

**一键安装（推荐，macOS / Linux）：**

```bash
curl -fsSL https://raw.githubusercontent.com/OpenBMB/PilotDeck/main/install.sh | bash
```

脚本会自动配置 Node.js 22、克隆代码、安装依赖并编译前端。安装完成后：

```bash
pilotdeck            # 在 http://localhost:3001 启动服务
pilotdeck status     # 查看运行状态
```

**源码安装（适合开发者）：**

```bash
git clone https://github.com/OpenBMB/PilotDeck.git
cd PilotDeck
npm install
cd ui && npm install && cd ..
cd ui && npm run dev     # 开发模式，访问 http://localhost:5173
```

然后在 `~/.pilotdeck/pilotdeck.yaml` 中配置模型 Provider，或直接在 Web UI 设置界面中可视化配置。

**Docker Compose：**

```bash
docker compose up -d
```

## 应用场景

PilotDeck 团队展示了多个完整的端到端案例，且所有演示均通过智能路由在端侧模型上完成，无需调用云端旗舰模型：

- **工作文档生成**：输入「调研中国大模型应用市场，整理成正式 HTML 白皮书」，Agent 自动完成调研、整理、排版全流程。
- **小游戏开发**：用 Vibe Coding 模式完成 iOS AR 小游戏的开发。
- **AI 工程平台**：从零构建 Embedding 低代码调优平台。
- **多语言内容运营**：将英文播客自动推送为中、日、法、韩、西、阿六种语言版本。

这些场景覆盖了文档写作、软件开发、AI 工程与内容运营四大方向，体现了 PilotDeck 作为通用生产力工具的潜力。

## 总结

PilotDeck 提出了一个清晰的设计主张：**AI Agent 的生产力瓶颈不在模型能力，而在系统架构**。通过 WorkSpace 级隔离解决上下文污染，通过白盒记忆解决记忆不可控，通过智能路由解决成本失控，通过 Always-on 解决人走即停——这四个问题的系统性回答，构成了 PilotDeck 区别于现有 Agent 工具的核心差异。

项目采用 AGPL 3.0 协议开源，由清华大学 THUNLP、面壁智能、OpenBMB 和 AI9Stars 联合维护，社区活跃度正在快速增长。对于正在构建多 Agent 系统或寻求降低 Agent 运行成本的开发者来说，PilotDeck 值得深入研究。

- 项目地址：[https://github.com/OpenBMB/PilotDeck](https://github.com/OpenBMB/PilotDeck)
- 官方网站：[https://pilotdeck.openbmb.cn](https://pilotdeck.openbmb.cn)
- 在线演示：[Live Demo](https://pilotdeck.openbmb.cn/pilotdeck.github.io/demo/p/pilotdeck-demo)
