---
title: 文件下载
excerpt: 一个关于通过ajax下载文件的轮子
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- 轮子
---

## 前言
公司有一个业务，特定的人下载特定的文件，之前想直接用`<a>`标签，然后自己把自己骂了= =，我真傻、真的。

## code
直接复制粘贴就行
```javascript
downFile (index) {
    this.$http({
    method: 'post',
    // url换成路径
    url: url ,
    // data里面放数据
    data: {
        data1: ,
        data2: 
    },
    responseType: 'blob',
    // token
    headers: {
        token: ,
        'Content-Type': 'application/json; charset=UTF-8'
    }
    }).then(
    (response) => {
        const blob = new Blob([response.data])
        const downloadElement = document.createElement('a')
        const href = window.URL.createObjectURL(blob)
        downloadElement.href = href
        downloadElement.download = this.allTestResults[index].reportUrl
        document.body.appendChild(downloadElement)
        downloadElement.click()
        document.body.removeChild(downloadElement)
        window.URL.revokeObjectURL(href)
    }
    ).catch((error) => {
        console.log(error)
    })
}
```