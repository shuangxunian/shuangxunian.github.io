---
title: 端口可以用6666吗
excerpt: 一篇关于为什么端口不能使用6666的文章
categories:
- 技术文章
tags:
- 网络
- 面试
---

## 前言
在搭建微服务分布式配置中心 Spring Cloud Config 时，如果将端口设置为 6000，总是访问不成功，像下面这样：

![](https://api2.mubu.com/v3/document_image/d26b4f6c-f95d-4937-bb02-a5c3e142de3d-3807603.jpg)

如果换成 Tomcat 默认的 8080 就可以访问了。
其实不止 6000，如果你配置成 6666 ，也是无法访问成功的！

## 分析
首先，当我们将项目的端口设置为 6000 之后，并非仅仅只有 Chrome 无法访问，Firefox、Safari 也是无法访问的，反而是经常被大家忽略的坐在角落的 IE/Edge 这对难兄难弟可以访问！看看 Safari 访问 6000 端口怎么说：

![](https://api2.mubu.com/v3/document_image/e476dbe5-9f64-4380-bbd6-e56a24e879b4-3807603.jpg)

再看看 Firefox 访问 6000 端口怎么说：

![](https://api2.mubu.com/v3/document_image/8ce541f5-1d06-4c3f-bae0-8990dda6a54b-3807603.jpg)

但是 Edge 就可以访问，如下：

![](https://api2.mubu.com/v3/document_image/f7235345-e369-4f22-bba1-b67379c49f0c-3807603.jpg)

看到这里，大家首先可以确认出现这个问题，和你的代码没有关系的，这个问题实际上是由 Chrome 默认的非安全端口限制导致的，除了上文说的 6000， 还有其他端口也无法在 Chrome 、Firefox 以及 Safari 中访问（具体端口见文末列表）。

这些无法访问的端口大部分都是小于 1024，小于 1024 的端口大家应该会很少使用， 基本上不会在这个上面栽跟头。大于 1024 的端口也并非每一个都可以使用，这才是容易犯错的地方。

## 解决
解决此问题有两个思路：

1. 修改项目端口（推荐）
2. 修改浏览器配置，使之允许访问非安全端口

这里推荐大家使用第一种方案，省事；如果非要使用第二种方案：

- Chrome 修改办法为：右键单击Chrome快捷方式 -> 目标 -> 末尾添加参数：--explicitly-allowed-ports=6000

![](https://api2.mubu.com/v3/document_image/2c1968ee-f09c-4e73-aebf-b6c47d3adcb8-3807603.jpg)

- Firefox 修改办法为浏览器地址栏输入 about:config 打开配置页面，然后搜索 network.security.ports.banned.override ，将其值设为 6000 即可(如果没有则右键单击新建即可)。

![](https://api2.mubu.com/v3/document_image/cd6095d6-2493-42fa-bfe0-27d39b070d9e-3807603.jpg)

受限端口列表：

| 端口 | 原因                           |
| :--- | :----------------------------- |
| 1    | tcpmux                         |
| 7    | echo                           |
| 9    | discard                        |
| 11   | systat                         |
| 13   | daytime                        |
| 15   | netstat                        |
| 17   | qotd                           |
| 19   | chargen                        |
| 20   | ftp data                       |
| 21   | ftp access                     |
| 22   | ssh                            |
| 23   | telnet                         |
| 25   | smtp                           |
| 37   | time                           |
| 42   | name                           |
| 43   | nicname                        |
| 53   | domain                         |
| 77   | priv-rjs                       |
| 79   | finger                         |
| 87   | ttylink                        |
| 95   | supdup                         |
| 101  | hostriame                      |
| 102  | iso-tsap                       |
| 103  | gppitnp                        |
| 104  | acr-nema                       |
| 109  | pop2                           |
| 110  | pop3                           |
| 111  | sunrpc                         |
| 113  | auth                           |
| 115  | sftp                           |
| 117  | uucp-path                      |
| 119  | nntp                           |
| 123  | NTP                            |
| 135  | loc-srv /epmap                 |
| 139  | netbios                        |
| 143  | imap2                          |
| 179  | BGP                            |
| 389  | ldap                           |
| 465  | smtp+ssl                       |
| 512  | print / exec                   |
| 513  | login                          |
| 514  | shell                          |
| 515  | printer                        |
| 526  | tempo                          |
| 530  | courier                        |
| 531  | chat                           |
| 532  | netnews                        |
| 540  | uucp                           |
| 556  | remotefs                       |
| 563  | nntp+ssl                       |
| 587  | stmp?                          |
| 601  | ??                             |
| 636  | ldap+ssl                       |
| 993  | ldap+ssl                       |
| 995  | pop3+ssl                       |
| 2049 | nfs                            |
| 3659 | apple-sasl / PasswordServer    |
| 4045 | lockd                          |
| 6000 | X11                            |
| 6665 | Alternate IRC [Apple addition] |
| 6666 | Alternate IRC [Apple addition] |
| 6667 | Standard IRC [Apple addition]  |
| 6668 | Alternate IRC [Apple addition] |
| 6669 | Alternate IRC [Apple addition] |

## 参考链接
[项目端口可以设置为 6666 吗？](https://mp.weixin.qq.com/s/MqCFoxRm8aHbTRZa6AARzA)