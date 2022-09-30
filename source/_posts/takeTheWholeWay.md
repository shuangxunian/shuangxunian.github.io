---
title: javascript取整方法floor、round、ceil
excerpt: 关于js取整
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## floor向下取整:
```javascript
Math.floor(0.20); // 0
Math.floor(0.90); // 0
Math.floor(-0.90); // -1
Math.floor(-0.20); // -1
```

## round四舍五入
```javascript
Math.round(0.2) // 0
Math.round(0.9) // 1
Math.round(-0.9) // -1
Math.round(-0.2) // 0
```

## ceil向上取整
```javascript
Math.ceil(0.2) // 1
Math.ceil(0.9) // 1
Math.ceil(-0.9) // 0
Math.ceil(-0.2) // 0
```