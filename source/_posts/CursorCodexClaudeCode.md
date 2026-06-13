---
title: Cursor、Codex、Claude Code：竞品还是共生？
excerpt: 聊聊这三个最火的 AI 编程工具背后的关系、商业逻辑，以及 Cursor 收费模式引发的争议。
date: 2026-05-08
categories:
- AI技术
tags:
  - AI
  - 编程工具
  - Cursor
  - Claude Code
  - Codex
---

最近有人问我：Cursor、Codex、Claude Code 这三个工具是竞品吗？背后有什么关系？

聊完之后觉得挺有意思的，整理一下。

## 三个工具，三种定位

**Cursor**

独立公司 Anysphere 做的，2022 年成立。本质是 fork 了 VS Code 的编辑器，深度集成 AI。用的模型是 Claude + GPT-4 + 自家微调模型，多家混用。目前最火的 AI 编辑器，估值已经几十亿美元。

**Claude Code**

Anthropic 自己出的，2025 年初发布。是个命令行工具（CLI），不是编辑器，跑在终端里。直接用 Claude 模型，Anthropic 亲儿子。定位是 agentic coding，能自主读写文件、跑命令、做复杂任务。

**Codex**

OpenAI 出的，2025 年发布（注意：不是早期那个代码补全 API，是新的 agent）。也是 CLI/agent 形态，跑在云端沙箱里。用 GPT 系列模型，OpenAI 亲儿子。

## 关系有点微妙

Cursor 是独立产品，但重度依赖 Anthropic（Claude）和 OpenAI 的模型，某种程度上是"寄生"在两家大模型公司上的。

Claude Code 和 Codex 是 Anthropic vs OpenAI 的直接对抗，两家都想自己做 coding agent 而不只是卖 API。

Anthropic 一边把模型卖给 Cursor，一边用 Claude Code 跟 Cursor 竞争——有点微妙。

简单说：**Cursor 是编辑器派，Claude Code 和 Codex 是 agent 派**，打法不完全一样，但目标用户高度重叠。

## Cursor 的收费逻辑，以及为什么让人觉得离谱

Cursor 现在免费版只能用有限的"快速请求"次数，用完就降速或者只能用便宜模型。想用 Claude Sonnet、GPT-4o 这些好模型，得开 Pro（$20/月）。

有人觉得这很离谱：VS Code 是开源的，Cursor fork 了它，接了两个别人的模型，然后还要买会员才能切换到别人的模型？

但这就是商业逻辑。

Cursor 本质上是个模型转售商——它把 Anthropic 和 OpenAI 的 API 包一层卖给你，每次对话都要付 API 费。用户越多、用得越狠，亏得越多。早期靠融资补贴，现在估值上去了反而要证明能赚钱，所以收紧了。

它的价值主张从来不是"我给你免费用模型"，而是"我把 AI 和编辑器的体验做得极好"——快捷键、上下文感知、多文件编辑这些。

## 但护城河在缩小

你完全可以绕过 Cursor：

- 自己装 VS Code + Continue 插件，直接填自己的 API key，想用哪个模型用哪个
- 或者直接用 Claude Code CLI，原厂直连，买 Anthropic Max 套餐，比 Cursor Pro 划算

随着 Claude Code 和 Codex 越来越好用，Cursor 的"开箱即用、体验顺滑"优势在缩小。

它现在的处境有点像早期的 Notion——靠体验溢价，但一旦竞品追上来，护城河就很浅。

---

AI 编程工具这个赛道还在快速变化，今天的格局明年可能完全不一样。不过有一点是确定的：对开发者来说，选择越来越多，议价权越来越强。
