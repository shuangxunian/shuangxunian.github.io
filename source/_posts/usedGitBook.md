---
title: GitBook 使用教程
excerpt: 如题目~
categories:
- 技术文章
tags:
- GitBook
---

## 简介
[GitBook 官网](https://www.gitbook.com)
[GitBook 文档](https://github.com/GitbookIO/gitbook)

## 准备工作
### 安装 Node.js
GitBook 是一个基于 Node.js 的命令行工具，下载安装 Node.js，安装完成之后，你可以使用下面的命令来检验是否安装成功。
```
$ node -v
//如果成功应该输出一个版本号
v7.7.1
```

### 安装 GitBook
输入下面的命令来安装 GitBook。
```
npm install gitbook-cli -g
```
安装完成之后，你可以使用下面的命令来检验是否安装成功。
```
$ gitbook -V
//如果成功应该输出两个版本号
CLI version: 2.3.2
GitBook version: 3.2.3
```

### 编辑器
[VsCode](https://code.visualstudio.com/)

## 启动
准备工作做好之后，我们进入一个你要写书的目录，输入如下命令。
```
$ gitbook init
warn: no summary file in this book
info: create README.md
info: create SUMMARY.md
info: initialization is finished
```
**注：**视个人网络，第一次时间会特别长，请耐心等待。
可以看到他会创建 README.md 和 SUMMARY.md 这两个文件，README.md 应该不陌生，就是说明文档，而 SUMMARY.md 其实就是书的章节目录，其默认内容如下所示：
```markdown
# Summary

* [Introduction](README.md)
```
接下来，我们输入 $ gitbook serve 命令，然后在浏览器地址栏中输入 http://localhost:4000 便可预览书籍。
效果如下所示：

![](https://api2.mubu.com/v3/document_image/98db8c9e-57be-42a9-b873-1fa7bae6683b-3807603.jpg)

运行该命令后会在书籍的文件夹中生成一个 _book 文件夹, 里面的内容即为生成的 html 文件，我们可以使用下面命令来生成网页而不开启服务器。
```
gitbook build
```
下面我们来详细介绍下 GitBook 目录结构及相关文件。

## 目录结构
GitBook 基本的目录结构如下所示：
```
.
├── README.md
├── SUMMARY.md
├── chapter-1/
|   ├── README.md
|   └── something.md
└── chapter-2/
    ├── README.md
    └── something.md
```
下面我们主要来讲讲 SUMMARY.md 文件。
这个文件主要决定 GitBook 的章节目录，它通过 Markdown 中的列表语法来表示文件的父子关系，下面是一个简单的示例：
```markdown
# Summary
* [Introduction](README.md)
* [Part I](part1/README.md)
    * [Writing is nice](part1/writing.md)
    * [GitBook is nice](part1/gitbook.md)
* [Part II](part2/README.md)
    * [We love feedback](part2/feedback_please.md)
    * [Better tools for authors](part2/better_tools.md)
```
这个配置对应的目录结构如下所示:

![](https://api2.mubu.com/v3/document_image/5308793c-4bb5-4fe3-8915-9b4cc7edfe1f-3807603.jpg)

我们通过使用 标题 或者 水平分割线 将 GitBook 分为几个不同的部分，如下所示：
```markdown
# Summary

### Part I

* [Introduction](README.md)
* [Writing is nice](part1/writing.md)
* [GitBook is nice](part1/gitbook.md)

### Part II

* [We love feedback](part2/feedback_please.md)
* [Better tools for authors](part2/better_tools.md)

---

* [Last part without title](part3/title.md)
```
这个配置对应的目录结构如下所示:

![](https://api2.mubu.com/v3/document_image/e0a8ea87-e514-4541-8b9c-291dba57ce4b-3807603.jpg)

## 插件
GitBook 有 插件官网，默认带有 5 个插件，highlight、search、sharing、font-settings、livereload，如果要去除自带的插件， 可以在插件名称前面加 -，比如：
```json
{
    "plugins": [
        "-search"
    ]
}
```
如果要配置使用的插件可以在 book.json 文件中加入即可，比如我们添加 plugin-github，我们在 book.json 中加入配置如下即可：
```json
{
    "plugins": [ "github" ],
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/your/repo"
        }
    }
}
```
然后在终端输入 gitbook install ./ 即可。

如果要指定插件的版本可以使用 plugin@0.3.1，因为一些插件可能不会随着 GitBook 版本的升级而升级。

## 参考链接
[GitBook 使用教程](https://www.jianshu.com/p/421cc442f06c)