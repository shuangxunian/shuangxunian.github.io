---
title: 基于hexo+gitee更换主题
banner_img: https://api2.mubu.com/v3/document_image/bdc5425b-8fcc-424f-a94f-0c4b375f528c-3807603.jpg
excerpt: 如何更换个人网站的主题
date: 2022-09-30
categories:
- 技术文章
tags:
- 建站
- gitee
- 博客主题
---

# 前言

上文已经讲了如何建站，本文讲一下如何更换主题

# 正文

你可以自己去[官网](https://hexo.io/themes/)上找你自己喜欢的主题，这里用我喜欢的主题来举例。
[点此](https://github.com/theme-kaze/hexo-theme-Kaze)跳转到本主题的github仓库中，点击code——Download ZIP，将压缩包下载到本地；

![正文0](https://api2.mubu.com/v3/document_image/88ebd4e5-e640-44e6-b904-4a13a8ae986c-3807603.jpg)

找到你个人网站所在的文件夹，将压缩包解压到themes里，并重命名为kaze，并将原来的删除。

打开个人网站下的_config.yml，将101行左右的theme:landscape改为theme:kaze。

打开个人网站下的source文件，新建about文件夹，在此文件夹中新建index.md，写上关于自己的一些话；新写的Markdown放在同文件下的_posts中。

找到themes/kaze/_config.yml文件，对其做如下修改。

- 3~7行为个人信息，填上自己信息就行，这里不再赘述

- 19行about为一句话，20行写上自己的一句话就行

- 49行为版权声明，看个人习惯决定是否使用

# Markdown

```
---
title: 标题
banner_img: 首页图
excerpt: 简介
date: 2022-09-30
categories:
- 分类
tags:
- 标签
---
```
以上是Markdown开始需要写的东西，只有写上才能看见。

# 收尾

处理完后执行：

- **hexo clean**

- **hexo g**

- **hexo d**

然后回到部署的页面，点击更新即可。
