---
title: 用三个 AI Agent 替代一个：Harness 架构实践
excerpt: 把 Planner、Generator、Evaluator 拆成三个独立 Agent，通过文件通信，让评估真正独立于实现。
date: 2026-05-27
categories:
- AI技术
tags:
  - AI Agent
  - 软件工程
  - 架构设计
---

> 本文记录了一次把 Anthropic Harness 架构落地到实际项目的过程。核心思路：把"规划、实现、评估"三件事拆给三个独立的 Agent，通过文件通信，让评估真正独立于实现。

## 为什么要拆成三个 Agent

单 Agent 做完整流程有一个根本问题：**它既是运动员，又是裁判**。

Generator 写完代码，自评"没问题"，然后用户发现一堆 bug。这不是模型能力的问题，而是结构问题——同一个上下文里既有实现细节，又要做客观评估，两者天然冲突。

Anthropic 在他们的 Harness 论文里给出了一个解法：把 Generator 和 Evaluator 的上下文完全隔离。Evaluator 不知道代码怎么写的，不知道 Generator 遇到了什么困难——它只看到最终产品，像真实用户一样操作和评判。

## 三个角色的分工

```
Planner ──spec.md──> 项目目录 <──feedback.md── Evaluator
                         │                         ↑
                         └──────> Generator ───────┘
                              (读 spec + feedback
                               写代码 + 更新组件清单)
```

**Planner**：把 1-4 句话的需求扩展成完整的产品规格（spec.md）。只做产品设计，不写代码。

**Generator**：读 spec.md 和 feedback.md，写代码。每轮构建后判断策略：
- 上轮总分 ≥ 5.0 且趋势上升 → **REFINE**（在现有代码基础上修复）
- 上轮总分 < 5.0 或连续两轮未提升 → **PIVOT**（推翻重来）

**Evaluator**：用 Playwright 实际操作页面，按四维标准打分，产出 feedback.md。**绝对不读代码**，只看产品。

## 文件通信协议

三个 Agent 通过 NFS 共享目录通信，核心文件：

| 文件 | 谁写 | 谁读 | 说明 |
|------|------|------|------|
| `spec.md` | Planner | Generator + Evaluator | 产品规格，版本归档 |
| `feedback.md` | Evaluator | Generator | 每轮覆盖写入 |
| `score_history.md` | Evaluator | Generator + Evaluator | 每轮追加，用于判断趋势 |
| `progress.md` | Generator | Generator | 每轮构建状态 |
| `project_summary.md` | Evaluator | Planner + Generator | PASS 后更新，跨版本记忆 |
| `component_registry.md` | Generator | Generator | 可复用组件清单 |

关键约束：
- Evaluator 绝对不读 `backend/`、`frontend/` 代码目录
- Generator 绝对不读 Evaluator 的对话历史
- 所有角色只读 `current` 版本的文件

## 四维评分标准

| 维度 | 权重 | 关注点 |
|------|------|--------|
| Design Quality | 30% | 颜色/字体/布局是否构成统一的视觉语言 |
| Originality | 20% | 是否有明显的设计意图，而非模板默认值 |
| Craft | 25% | 排版层级、间距一致性、对比度、响应式 |
| Functionality | 25% | 用户能否完成核心任务 |

通过线：加权总分 ≥ 7.0，且任一维度不低于 3.0。

## 反宽容指令

这是整个方案里最重要的部分，写进 Evaluator 的 SKILL.md：

1. **不要说服自己问题不重要。** 如果你注意到了一个问题，它就是问题。
2. **每个按钮都必须实际点击验证。** 不点 = 没测 = 评估失败。
3. **浅层测试 = 失败的评估。** 只看首页就打分是不可接受的。
4. **宁可误报也不要漏报。**
5. **7.0 是通过线，不是起步分。** 大多数首轮产出应该在 4.0-6.0 分。
6. **8.0 分以上需要论证。**
7. **不要因为"已经比上次好了"就放松标准。**

## 版本管理

一个项目会经历多个版本迭代。目录结构：

```
/harness/{project_name}/
├── project_summary.md    # 产品当前状态（滚动更新）
├── component_registry.md # 可复用组件清单
├── versions.json         # 版本索引
├── current -> v2/        # 软链接指向当前版本
├── v1/                   # 归档，不可修改
│   ├── spec.md
│   ├── feedback.md
│   └── score_history.md
└── v2/
    └── ...
```

`project_summary.md` 是跨版本记忆的核心：Planner 写新版本 spec 时，从这里了解产品当前状态；Generator 开始新版本时，从这里知道已有什么可以复用。

## Context Reset 策略

Evaluator 每轮强制清 session，确保用全新上下文评估，不受上一轮评分记忆的影响。Generator 按需清 session：执行 PIVOT 策略时清，REFINE 时可以保留。

这对应 Anthropic 原文的表述：*A reset provides a clean slate, at the cost of the handoff artifact having enough state for the next agent to pick up the work cleanly.*

## 成本参考

参考 Anthropic 的实测数据（DAW 音乐工作站，Opus 4.6）：

| 阶段 | 时间 | 成本 |
|------|------|------|
| Planner | 4.7 min | $0.46 |
| Build Round 1 | 2 hr 7 min | $71.08 |
| QA Round 1 | 8.8 min | $3.24 |
| Build Round 2 | 1 hr 2 min | $36.89 |
| QA Round 2 | 6.8 min | $3.09 |
| Build Round 3 | 10.9 min | $5.88 |
| QA Round 3 | 9.6 min | $4.06 |
| **合计** | **3 hr 50 min** | **$124.70** |

三轮 QA 合计 $10.39，只占总成本 8%。加一个独立 Evaluator，成本增量不到 10%，但它每轮都抓到了 Generator 自评遗漏的实际问题。

## 适用边界

| 场景 | 建议 |
|------|------|
| 简单页面、单组件修改 | 不用 harness，码仔自测够 |
| 中等复杂度看板/dashboard | 可选 |
| 完整前后端分离重构 | 推荐 |

Evaluator 的价值取决于任务是否在模型能力边界上。功能点越多、交互越复杂，独立评估的价值越高。

---

本质上，这是把 Anthropic 说的 context reset 做成了结构化实现——不是在同一个 Agent 里手动清空上下文，而是切换到一个全新的、干净的 Agent，通过文件传递状态。
