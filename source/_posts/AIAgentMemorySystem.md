---
title: AI Agent 记忆系统：临时记忆 vs 永久记忆的设计与实现
excerpt: 深入理解 AI Agent 的记忆架构，包括临时/永久记忆的自动分类、RAG 检索流程，以及完整的 Python 实现代码
date: 2026-05-06
categories:
- 技术文章
tags:
- AI
- Agent
- RAG
- Python
---

## 前言

在构建 AI Agent 时，记忆系统是核心能力之一。本文基于 DataWhale 的 HelloAgents 第八章内容，探讨如何设计一个具备临时记忆和永久记忆的智能体系统，以及如何让 Agent 自动判断信息该存哪里。

## 一、为什么 Agent 需要记忆？

大语言模型（LLM）存在天然局限：
- 上下文窗口有限，无法记住所有历史对话
- 缺乏个性化，不了解用户偏好
- 多轮对话可能出现不一致

记忆系统让 Agent 能够跨会话保留信息，实现个性化交互和持续学习。

## 二、临时记忆 vs 永久记忆

### 临时记忆（工作记忆）

- **特点**：只存当前会话、马上要用、用完就丢的信息
- **生命周期**：会话结束自动清理
- **典型内容**：当前对话上下文、一次性指令、临时任务状态
- **实现方式**：内存中的列表/滑动窗口

### 永久记忆（长期记忆）

- **特点**：需要长期记住、反复使用、有价值的信息
- **生命周期**：持久化存储，跨会话可用
- **典型内容**：用户偏好、知识点、历史事件、重要规则
- **实现方式**：向量数据库（Chroma、Milvus 等）

### 谁来决定存哪里？

**不是人工区分，是 Agent（LLM）自动判断。**

通过一次额外的 LLM 调用，让模型对每轮对话内容进行分类和重要性打分：
- 重要性 < 0.7 → 临时记忆
- 重要性 ≥ 0.7 → 永久记忆

## 三、完整的 Agent 记忆 + RAG 流程

```
用户输入
   ↓
RAG 召回永久记忆（从本地向量库检索）
   ↓
拼接：永久记忆 + 临时记忆 + 用户问题 → 发给 LLM
   ↓
LLM 生成回答，返回给用户
   ↓
本轮对话送入记忆管理模块
   ↓
调用 LLM 判定：临时/永久 + 重要性打分
   ↓
分流存入：临时工作记忆 | 永久向量库
```

### 关键澄清

**RAG 检索的是你自己本地的向量数据库，不是 Claude/GPT 云端的数据库。**

整个流程是：
1. 你的 Agent 代码主动去本地向量库查询
2. 把查到的相关记忆 + 用户问题拼接好
3. 发给 LLM（Claude/GPT）生成回答
4. LLM 本身不知道你有什么记忆库，它只处理你喂给它的文本

## 四、Python 完整实现

### 安装依赖

```bash
pip install chromadb python-dotenv anthropic
```

### 完整代码

```python
import os
import json
from dotenv import load_dotenv
from anthropic import Anthropic
import chromadb
from chromadb.utils import embedding_functions

load_dotenv()
client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

# =====================
# 1. 初始化：永久记忆（RAG向量库） + 临时记忆
# =====================
chroma_client = chromadb.PersistentClient(path="./memory_db")
embedding_func = embedding_functions.DefaultEmbeddingFunction()

# 永久记忆（RAG用）
permanent_collection = chroma_client.get_or_create_collection(
    name="permanent_memory",
    embedding_function=embedding_func
)

# 临时记忆（会话级，用完就丢）
working_memory = []

# =====================
# 2. LLM 自动判断临时/永久记忆
# =====================
def judge_memory_type(user_msg, assistant_msg):
    prompt = f"""
你是记忆分类器，只输出JSON，不要多余文字。
请判断以下对话内容应该存入【临时记忆】还是【永久记忆】，并给出0~1的重要性分数。

规则:
- 临时记忆: 闲聊、当前会话上下文、一次性内容
- 永久记忆: 用户偏好、习惯、知识点、长期信息、重要设定

对话:
用户: {user_msg}
助手: {assistant_msg}

输出JSON格式:
{{"memory_type":"临时/永久","importance":0.xx}}
"""
    response = client.messages.create(
        model="claude-3-haiku-20240307",
        max_tokens=100,
        messages=[{"role": "user", "content": prompt}]
    )
    return json.loads(response.content[0].text)

# =====================
# 3. 存储记忆：自动分流
# =====================
def save_memory(user_msg, assistant_msg):
    judge_result = judge_memory_type(user_msg, assistant_msg)
    memory_type = judge_result["memory_type"]
    importance = judge_result["importance"]

    if memory_type == "临时":
        working_memory.append({
            "user": user_msg,
            "assistant": assistant_msg
        })
        print("✅ 已存入【临时记忆】")
    else:
        content = f"用户: {user_msg}\n助手: {assistant_msg}"
        permanent_collection.add(
            documents=[content],
            ids=[f"mem_{len(permanent_collection.get()['documents'])}"],
            metadatas=[{"importance": importance}]
        )
        print("✅ 已存入【永久记忆】(RAG可用)")

# =====================
# 4. RAG 检索：从永久记忆找相关内容
# =====================
def retrieve_memory(query, top_k=3):
    result = permanent_collection.query(
        query_texts=[query],
        n_results=top_k
    )
    docs = result["documents"][0] if result["documents"] else []
    return "\n".join(docs)

# =====================
# 5. 主对话流程
# =====================
def chat(user_msg):
    # 1. RAG 检索永久记忆
    rag_context = retrieve_memory(user_msg)

    # 2. 拼接临时记忆
    working_context = "\n".join([
        f"用户: {w['user']}\n助手: {w['assistant']}"
        for w in working_memory[-5:]  # 只保留最近5轮
    ])

    # 3. 构造 Prompt
    prompt = f"""
相关历史记忆:
{rag_context if rag_context else "（无）"}

本次会话上下文:
{working_context if working_context else "（无）"}

用户问题: {user_msg}
请回答。
"""

    # 4. 调用 Claude 生成回答
    response = client.messages.create(
        model="claude-3-haiku-20240307",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )
    assistant_msg = response.content[0].text
    print(f"\n🤖 助手: {assistant_msg}")

    # 5. 自动判断并保存记忆
    save_memory(user_msg, assistant_msg)
    return assistant_msg

# =====================
# 运行
# =====================
if __name__ == "__main__":
    print("💬 Agent 记忆系统已启动（输入 exit 退出）")
    while True:
        user_input = input("\n你: ")
        if user_input in ["exit", "quit"]:
            break
        chat(user_input)
```

### 环境变量配置

创建 `.env` 文件：

```env
ANTHROPIC_API_KEY=your-api-key-here
```

## 五、核心要点总结

1. **临时记忆 = 当前会话用**，会话结束就丢
2. **永久记忆 = 以后长期用**，持久化到向量库
3. **谁判断？** Agent + LLM 自动分类，不需要人工干预
4. **RAG 查的是你自己的本地向量库**，不是 LLM 云端数据库
5. **每一轮对话都要走**：检索 → 回答 → 判断记忆类型 → 分流存储

## 六、进阶优化方向

- **记忆固化**：定期将高频使用的临时记忆升格为永久记忆
- **记忆遗忘**：对长期未被检索的永久记忆降权或清理
- **记忆摘要**：当永久记忆过多时，自动压缩合并相似内容
- **多类型记忆**：区分语义记忆（知识）和情景记忆（事件经历）

## 参考

- [DataWhale HelloAgents 第八章：记忆与检索](https://datawhalechina.github.io/hello-agents/#/./chapter8/)
