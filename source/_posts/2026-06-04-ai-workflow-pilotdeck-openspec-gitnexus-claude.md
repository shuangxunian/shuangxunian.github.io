---
title: PilotDeck + OpenSpec + GitNexus + Claude：我的 AI 辅助开发工作流
excerpt: 从「对话玩具」到真正干活的生产力工具——记录我用四个工具搭建的 AI 编程工作流。
date: 2026-06-04
categories:
  - 工程实践
tags:
  - AI
  - Claude
  - 多智能体
  - 开发工具
  - 工作流
---

最近把 AI 辅助开发这件事想明白了一些，用上了四个工具的组合，记录一下。

## 背景：为什么单用 Claude 不够

直接用 Claude 写代码，最大的问题不是能力，是**上下文管理**。

- 代码库大了，Claude 不知道调用链在哪
- 需求模糊，AI 猜着写，常常遗漏要求或加了多余的东西
- 多个项目并行，上下文混在一起
- 每次对话都要重新喂背景，效率极低

要解决这些问题，需要在 Claude 外面搭一层基础设施。

---

## 工具一：PilotDeck — 多 Agent 调度平台

[PilotDeck](https://github.com/OpenBMB/PilotDeck) 是清华 OpenBMB 团队开源的 Agent 平台，核心理念是 **WorkSpace 工作舱**：

- 每个项目一个独立的舱
- 舱内有专属文件、专属记忆、专属技能
- 多项目并行互不干扰

除了工作舱，它还有几个关键能力：

**Smart Router（省钱路由）**：TokenSaver 智能降档 + 多 Provider Fallback。复杂任务用强模型，简单任务自动降档，账单直接少一截。

**双层记忆**：Project Memory + Feedback Memory。AI 犯过的错、你纠正过的决策，都会沉淀下来，下次不再踩坑。白盒可追溯，不是黑盒魔法。

**Always-on 后台常驻**：部署完你去干别的，它自己推进任务。Cron 定时任务 + Discovery 主动发现，不用你盯着对话框。

PilotDeck 解决的是**多项目、多任务的调度和记忆问题**，是整套工作流的骨架。

---

## 工具二：OpenSpec — 规范驱动开发

[OpenSpec](https://github.com/Fission-AI/OpenSpec) 是一个为 AI 编码助手设计的规范驱动开发工具。一句话：**在写代码之前，先让人和 AI 对需求达成明确共识**。

### 为什么需要它？

AI 编码助手的典型问题：

- 根据模糊提示生成不符合需求的代码
- 遗漏重要功能要求，或添加不必要的功能
- 代码行为不可预测

OpenSpec 的思路是**规范先行**：把需求写成结构化的 spec 文件，AI 根据 spec 写代码，而不是根据你一句话描述猜。

### 工作流

OpenSpec 的核心是四步循环：

```
1. 起草提案（proposal）
   → openspec init / /openspec:proposal <功能描述>
   → AI 生成 proposal.md + tasks.md + 规范差异文件

2. 审查对齐
   → 与 AI 讨论，补充验收标准，直到双方达成一致

3. 实施
   → /openspec:apply <change-name>
   → AI 按 tasks.md 逐步实施，打勾标记

4. 归档
   → openspec archive <change-name>
   → 变更合并回主规范，留存完整历史
```

### 目录结构

```
openspec/
├── specs/          # 当前规范（真理源）
├── changes/        # 进行中的变更提案
│   └── feature-x/
│       ├── proposal.md
│       ├── tasks.md
│       └── specs/  # 规范差异（delta）
└── project.md      # 项目级约定（技术栈、命名规范等）
```

### 安装

```bash
npm install -g @fission-ai/openspec@latest
cd your-project
openspec init
```

支持 Claude Code、Cursor、Copilot 等主流工具，以及所有兼容 AGENTS.md 的工具。

---

## 工具三：GitNexus — 代码库知识图谱

[GitNexus](https://github.com/abhigyanpatwari/GitNexus) 自称「Zero-Server Code Intelligence Engine」，一句话概括：**把你的代码库索引成知识图谱，然后通过 MCP 喂给 AI Agent**。

它做的事情：

1. 用 Tree-sitter 解析代码，建立依赖关系、调用链、模块聚类
2. 把这张图存进本地的 LadybugDB（零服务器，完全本地）
3. 通过 MCP（Model Context Protocol）暴露给 Claude Code、Cursor、Codex 等工具

类比：DeepWiki 帮你**理解**代码，GitNexus 帮 AI **分析**代码——因为知识图谱追踪的是每一条调用关系，不只是文字描述。

实际效果：AI Agent 不再「盲改」。它知道你改这个函数会影响哪些调用方，不会改完一处漏掉三处依赖。

```bash
npm install -g gitnexus
gitnexus index .    # 索引当前项目
gitnexus serve      # 启动 MCP Server
```

然后在 Claude Code / Cursor 里接入这个 MCP，Agent 就有了完整的代码图谱视角。

---

## 组合起来：完整工作流

```
用户需求
    ↓
PilotDeck（WorkSpace 路由 + 跨会话记忆）
    ↓
OpenSpec（起草规范 → 对齐需求 → 生成任务清单）
    ↓
GitNexus MCP（注入代码图谱，理解调用链）
    ↓
Claude（在完整上下文下实施变更）
    ↓
OpenSpec 归档（规范更新，变更留存历史）
    ↓
PilotDeck 记忆更新（下次不再踩同样的坑）
```

每一层解决一个具体问题：

| 工具 | 解决的问题 |
|------|-----------|
| PilotDeck | 多项目调度、跨会话记忆、后台常驻 |
| OpenSpec | 需求对齐，避免「能跑但不对」 |
| GitNexus | 代码库结构感知，避免盲改漏改 |
| Claude | 核心推理和代码生成能力 |

---

## 使用感受

最大的变化是：**我开始信任 AI 的输出了**。

以前用 Claude 写代码，每次都要仔细 review，因为它对项目一无所知，对需求的理解也可能和我偏差很大。现在三层上下文全部到位——它知道调用链、知道规范、知道需求细节、知道上次犯过什么错，输出质量稳定多了。

第二个变化是：**任务可以真的交出去**。OpenSpec 的任务清单 + PilotDeck 的 Always-on，让我可以把一个任务完整描述清楚、扔出去，去做别的事，过一会儿回来看结果。不用守着对话框一步步引导。

当然也有不成熟的地方——GitNexus 在超大型 monorepo 上索引速度还需要优化，OpenSpec 的规范维护本身也是工作量（不过这个工作量在后期会通过减少返工来赚回来）。整体来说这套组合已经在日常开发中省了我不少时间。

---

## 总结

AI 辅助开发真正变得可靠，需要在「模型能力」之外补三层：

1. **调度层**（PilotDeck）：管任务、管记忆、管多项目
2. **规范层**（OpenSpec）：管「人和 AI 对需求的共识」
3. **代码理解层**（GitNexus）：管「代码库里有什么、怎么连着的」

光有 Claude 不够，光有上下文注入也不够。这三层加在一起，才让 AI 真正进入你的开发流程，而不只是偶尔问一下的搜索引擎。

感兴趣的可以去看看：
- PilotDeck: https://github.com/OpenBMB/PilotDeck
- OpenSpec: https://github.com/Fission-AI/OpenSpec
- GitNexus: https://github.com/abhigyanpatwari/GitNexus
