---
title: 初探babel
excerpt: 关于babel.js的简单安装与使用
categories:
- 技术文章
tags:
- js
---

## 官网
https://babeljs.io/

## 方法1——引入
```html
<script src="./browser.min.js"></script>
<script type="text/babel">
    let a=12; 
    let show=n=>alert(n); 
    show(a);
</script>
```
**注意：**
1. 代码部分要添加type="text/babel"
2. 这个方法很影响性能，建议使用方法2，这里方法1只做简单举例。

## 方法2——编译
1. 安装node，初始化项目
```
npm init
```
2. 安装babel-cli
```
npm i @babel/core @babel/cli @babel/preset-env -D
npm i @babel/polyfill -S
```
3. 添加执行脚本
```javascript
//"任意名":"babel 输入文件 -d 输出文件"
"scripts":{
    "build":"babel src -d dest"
}
```
4. 添加babelrc配置文件
```javascript
{
    "presets":["@babel/preset-env"]
}
```
5. 执行
```
npm run build
```

## 参考链接
[【智能社】ES6精讲—主讲老师：石川（Blue）—高清版本](https://www.bilibili.com/video/BV1wt411t7hg)