---
title: 移除GitBook目录下方的“本书使用GitBook发布”字样
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- gitbook
---


## 前言
在使用GitBook生成书籍之后，目录的最下方会有“本书使用GitBook发布字样”，如果是英文环境则显示“Published with Gitbook”，如何移除这段文字呢？

## 一
首先，在book的根目录里创建styles文件夹，然后在其中创建website.css文件，添加以下内容:
```css
.gitbook-link {
  display: none !important;
}
 
```

## 二
其次，编辑book.json文件，添加下方内容。如果该文件不存在，请创建。更多关于book.json内容，请参考官方文档。
```json
{
  "styles": {
    "website": "styles/website.css"
  }
}
```

## 三
重新使用gitbook build生成book即可。

## 参考链接
[移除GitBook目录下方的“本书使用GitBook发布”字样](https://yimouleng.com/2018/12/04/%E7%A7%BB%E9%99%A4gitbook%E7%9B%AE%E5%BD%95%E4%B8%8B%E6%96%B9%E7%9A%84%E6%9C%AC%E4%B9%A6%E4%BD%BF%E7%94%A8gitbook%E5%8F%91%E5%B8%83%E5%AD%97%E6%A0%B7/)