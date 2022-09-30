---
title: GitHub 风格随机头像 API
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- api
---

## 前言
返回的是github风格的头像

## 如何使用
返回 Base64： `https://api.prodless.com/avatar`
```
<img src="data:image/png;base64,iVBOR..." />
```
返回图片： `https://api.prodless.com/avatar.png`
```
<img src="https://api.prodless.com/avatar.png" />
```

## 可选参数

|   字段    |   描述   |
|   ----    | ---- |
|   size    |   头像尺寸，单位 px，最小 24。例如： ?size=24   |
|   color   |   方块颜色，例如： ?color=50DCC7   |
|   backgroundColor   |   背景颜色，例如： ?backgroundColor=f0f   |

## 参考链接
[GitHub 风格随机头像 API](https://tools.prodless.com/avatar)