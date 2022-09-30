---
title: 基于hexo+gitee快速建站
banner_img: https://api2.mubu.com/v3/document_image/bdc5425b-8fcc-424f-a94f-0c4b375f528c-3807603.jpg
excerpt: 如何创建一个个人网站
date: 2022-09-30
categories:
- 技术文章
tags:
- 建站
- gitee
---

# 声明
本文只是让大家体验一下如何建立一个属于自己的站点，不涉及任何更深层次的技术，如果想走前端，建议去深了解一下，其他方向按本文步骤来即可。

# 关于hexo
hexo是一个快捷、简单并且强大，基于Node.js的一个静态博客框架。支持Markdown解析文章，能够很快的生成特定主题的静态网页。

# 关于gitee
基于 Git 的代码托管服务，除了提供最基础的 Git 代码托管之外，还提供代码在线查看、历史版本查看、Fork、Pull Request、打包下载任意版本、Issue、Wiki 、保护分支、代码质量检测、PaaS项目演示等方便管理、开发、协作、共享的功能。

# 安装工具
1.安装Node.js：[官网](https://nodejs.org/en/)，[教程](https://www.runoob.com/nodejs/nodejs-install-setup.html)
2.安装git bash工具：[官网](https://gitforwindows.org/)，点down，然后一路next就行
3.安装hexo：

- 新建一个存放blog所有东西的文件夹

- 文件夹内右键，点击git bash here

- 输入：**npm install -g hexo-cli**

- 安装完成后，查看版本，输入命令：**hexo -v**

  若告诉你版本，即是成功：

  ![版本图片](https://api2.mubu.com/v3/document_image/d944da36-7253-4da6-8e34-d7c50e6b79b5-3807603.jpg)

# 开始搭建

## 0.前言
以下步骤根据个人网速会有不同时间，不要总终止，耐心等待

## 1.初始化hexo
输入：**hexo init**；生成好的文件夹你会看到以下：

![生成文件](https://api2.mubu.com/v3/document_image/d4121266-e4fe-45b4-a792-ebc7efe2aac3-3807603.jpg)

其中：

- source：放文件图片等，你所写的Markdown也放在这里

- themes：主题

- _config.yml：站点配置文件，很多全局配置都在这个文件中。

## 2.运行到本机
输入：**hexo s**，会出现以下：

![运行](https://api2.mubu.com/v3/document_image/8ac93db9-ec2a-4e99-b23a-2cea5d0a43eb-3807603.jpg)

这些的意思是运行成功，关闭输入ctrl+c，运行结果要在浏览器输入 localhost:4000 即可看见：

![运行成功](https://api2.mubu.com/v3/document_image/bdc5425b-8fcc-424f-a94f-0c4b375f528c-3807603.jpg)

## 3.生成静态文件
此步是为了上传到gitee做的。因为目前只能自己在本地访问博客，但是想让其他人看到就要结合gitee来做了，这里多说一嘴，其实github也可以，但是好多人访问会很慢，所以选择gitee。输入：**hexo g**；目录中就会多出一个public文件夹，这个文件夹就是我们托管到码云上用的文件夹。

## 4.部署到gitee
1.注册：[官网](https://gitee.com/)
2.点击加号：

![创建0](https://api2.mubu.com/v3/document_image/9afd1e1c-adf1-49d4-b2ad-9586c819f80b-3807603.jpg)

点击新建仓库：

![创建1](https://api2.mubu.com/v3/document_image/37fa98e9-07b8-4c9c-90a0-49c6113562e9-3807603.jpg)

仓库名称**一定要**用个人空间地址，个人空间地址**一定不要**带大写字母，如果你不知道自己的空间地址是什么或者想修改，请点个人主页，个人空间地址进行操作，因为我已经建了同名仓库，这里被禁止，正常是绿的。

![创建2](https://api2.mubu.com/v3/document_image/3886795b-1562-404e-8d90-c1b031726d9f-3807603.jpg)

3.创建完成后我们需要安装插件，输入：**npm install hexo-deployer-git --save**。

4.接下来是配置根目录_config.jml文件，进行以下操作：

- 6行title改成自己网站名，10行author改成自己，11行language改成zh-CN

- 47行，改成以下（此步是为了代码高亮）；

```
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace: false
  wrap: true
  hljs: true
prismjs:
  enable: false
  preprocess: true
  line_number: true
  tab_replace: ''
```

- 105行，修改deploy的值，修改前如下图：

![创建3](https://api2.mubu.com/v3/document_image/aa330005-dce9-4ab8-acde-62a4e0c46a4c-3807603.jpg)

将其修改成：

```
type: git
repo: https://gitee.com/shuangxunian/shuangxunian.git
branch: master
message: blog
```

其中的repo为你刚刚建的仓库中的：

![创建4](https://api2.mubu.com/v3/document_image/374ec3c3-5be5-48f7-9e8e-41fc050b1793-3807603.jpg)


5.输入：**hexo d**
之后会弹出输入码云账号密码的对话框，输入账号密码，等待上传结束，之前创建的项目中出现了本地项目中public文件夹中的文件，这就代表部署成功了。

![创建5](https://api2.mubu.com/v3/document_image/d7c5d753-d109-4404-a397-21ad5a6e6a5c-3807603.jpg)

6.开启码云的Pages功能，点击启动服务：

![创建6](https://api2.mubu.com/v3/document_image/d7c5d753-d109-4404-a397-21ad5a6e6a5c-3807603.jpg)

7.点击链接，成功！

## 5.问题

有时候会出现上传没有css和js样式的情况，这里我们打开_config.yml，找到第14行左右

![p0](https://api2.mubu.com/v3/document_image/5738a97c-6728-4d66-9e00-aa1b6c182f6a-3807603.jpg)

将其url修改成自己的url：

![p1](https://api2.mubu.com/v3/document_image/58752849-f1fe-4e18-9e10-493f3242f653-3807603.jpg)

然后执行：

- **hexo clean**

- **hexo g**

- **hexo d**

然后回到部署的页面，点击更新：

![更新0](https://api2.mubu.com/v3/document_image/9a36c42a-cce6-4b12-b2c5-ba0f515d6c05-3807603.jpg)