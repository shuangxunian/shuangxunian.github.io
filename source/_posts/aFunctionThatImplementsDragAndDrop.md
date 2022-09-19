---
title: 一个实现拖拽的函数
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---

## code
```javascript
//只通过几个函数实现图片拖拽
let getElementDrags = el =>
    el.mouseDowns.map(mouseDown =>
        document.mouseMoves.takeUntil(document.mouseUps)
    )
    .concatAll()
getElementDrags(div).forEach(position => {
    img.position = position;
});
```