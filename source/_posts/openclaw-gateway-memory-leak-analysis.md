---
title: OpenClaw Gateway Memory Leak Analysis
date: 2026-06-25
categories:
  - Tech
tags:
  - OpenClaw
  - Ops
  - Memory Leak
  - Troubleshooting
excerpt: OpenClaw Gateway OOM root cause analysis and fix plan
---

# OpenClaw Gateway Memory Leak Analysis Report

## 1. Fault Overview

### 1.1 Symptoms

网关重启后内存从低水位迅速攀升至 8.7GB RSS。CPU 持续高负载。预计 2-3 小时内触发 OOM。故障时间窗口：10:10-11:03。

### 1.2 Timing Data

| Time | RSS Memory | Event |
|------|------------|-------|
| 10:10 | Very Low | Gateway restart |
| 10:27 | 3.08GB | Init complete |
| 10:34 | 5.29GB | Steady increase |
| 10:47 | 5.33GB | Brief pause, GC recovery |
| 10:55 | 8.70GB | Surge spike, critical risk |
| 11:03 | 8.2GB | GC intermittent, CPU 78 percent |

## 2. Traffic Source Analysis

### 2.1 Source Location

所有高频请求均来自 WebUI WebSocket 连接对 sessions.list 的轮询。共 5 个独立客户端连接，均来自内网。

#### Connection Stats

| Conn ID | Internal IP | Client Ver | Requests/5min | Notes |
|---------|-------------|------------|---------------|-------|
| conn-01 | Internal-01 | vcontrol-ui | 15 | Normal polling |
| conn-02 | Internal-02 | vcontrol-ui | 27 | Highest, reconnects |
| conn-03 | Internal-03 | vcontrol-ui | 17 | Normal polling |
| conn-04 | Internal-04 | vcontrol-ui | 17 | Normal polling |
| conn-05 | Internal-05 | v2026.4.23 | 14 | Normal polling |

### 2.2 Frontend Polling Behavior

WebUI 每 12 秒轮询一次 sessions.list 以获取实时 Agent 会话状态。只要浏览器标签页处于活跃状态，轮询即持续进行。多个用户打开多个标签页会导致请求量线性叠加。

## 3. Root Cause Analysis

### 3.1 Direct Trigger

5 个 WebUI 客户端高频并发轮询 sessions.list 是本次故障的直接触发条件。

### 3.2 Core Technical Root Causes

1. 全量会话文件遍历：每次调用均执行 loadCombinedSessionStoreForGateway，遍历全部 207 个 Agent 会话文件。
2. 大字段冗余：skillsSnapshot 字段单个大小为 3-5KB，全量聚合后单次请求约 26MB。
3. 深拷贝开销：读路径和缓存路径均使用 structuredClone，每次请求均在 V8 堆中创建新副本。5 分钟内发生 127 次调用。
4. 缓存无淘汰机制：SESSION_STORE_SERIALIZED_CACHE 使用 Map 实现，无 TTL 和容量上限，数据永久驻留。

### 3.3 Aggravating Factors

1. 232 个阻塞会话每次请求均触发额外诊断逻辑。
2. 195 个活跃 WebSocket 连接各自维护独立的状态上下文。
3. Tracing 插件每次请求创建新实例而非单例，进一步推高 CPU 负载。

## 4. Current Status

V8 GC 将内存从峰值 8.7GB 回收至 8.2GB，但未能阻断根因。内存仍处于高位，OOM 风险持续存在。

## 5. Fix Plan

### 5.1 Emergency (Immediate)

1. 前端优化：将轮询间隔从 12 秒提升至最低 30 秒。增加页面可见性监听，标签页置于后台时暂停轮询。
2. 通知运维关闭冗余控制台标签页。
3. API 限流：sessions.list 接口按 WebSocket 连接设置最低 30 秒调用间隔，超频请求返回缓存快照。

### 5.2 Root Fix (Version Release)

1. 数据裁剪：从 sessions.list 中移除 skillsSnapshot 字段，仅在 session.get 详情查询时返回。
2. 拷贝优化：将 structuredClone 替换为只读引用，读查询采用写时复制策略。
3. 缓存淘汰：为会话序列化缓存增加容量上限和 TTL 过期机制。

### 5.3 Long-term Architecture

1. 全局会话聚合快照缓存，供所有客户端共享。
2. Tracing 插件重构为单例模式。
3. 定期阻塞会话清理机制。

## 6. Notes

所有异常 IP 均为内网 NAT 地址。用户追溯需结合 DHCP 和堡垒机日志关联分析。优先推进限流与代码修复。
