---
title: HTML5离线存储原理
excerpt: 如题目
categories:
- 技术文章
tags:
- html
---

## 前言：
使用HTML5，通过创建cache manifest文件，可轻松创建web应用的离线版本。
HTML5引入了应用程序缓存，这意味着web应用可进行缓存，并可在没有网络时进行访问。
应用程序缓存为应用带来三个优势：
- 离线浏览--用户可在离线时使用它们。
- 速度--已经缓存的资源加载得更快。
- 减少服务器负载--浏览器将只从服务器下载更改过的资源。

## 原理和环境
如上面提到的HTML5的离线存储是基于一个新建的.appcache文件的，通过这个文件上的解析清单离线存储资源，这些资源就会像cookie一样被存储了下来。之后当网络在处于离线状态下时，浏览器会通过被离线存储的数据进行页面展示。

就像cookie一样，html5的离线存储也需要服务器环境。

## 解析清单
在开始之前要先了解下manifest（即.appcache文件），上面的解析清单要怎么写。

manifest 文件是简单的文本文件，它告知浏览器被缓存的内容（以及不缓存的内容）。

manifest 文件可分为三个部分：
- CACHE MANIFEST - 在此标题下列出的文件将在首次下载后进行缓存
- NETWORK - 在此标题下列出的文件需要与服务器的连接，且不会被缓存
- FALLBACK - 在此标题下列出的文件规定当页面无法访问时的回退页面（比如 404 页面）

在线的情况下,用户代理每次访问页面，都会去读一次manifest.如果发现其改变, 则重新加载全部清单中的资源。

## CACHE MANIFEST
第一行，CACHE MANIFEST，是必需的：

`CACHE MANIFEST /theme.css /logo.gif /main.js`

上面的 manifest 文件列出了三个资源：一个 CSS 文件，一个 GIF 图像，以及一个 JavaScript 文件。当 manifest 文件加载后，浏览器会从网站的根目录下载这三个文件。然后，无论用户何时与因特网断开连接，这些资源依然是可用的。

## NETWORK
白名单，使用通配符”*”. 则会进入白名单的open状态. 这种状态下.所有不在相关Cache区域出现的url都默认使用HTTP相关缓存头策略.

下面的 NETWORK 小节规定文件 “login.asp” 永远不会被缓存，且离线时是不可用的，即：`NETWORK: login.asp`

可以使用*来指示所有其他资源/文件都需要因特网连接，即：`NETWORK: *`

## FALLBACK
下面的 FALLBACK 小节规定如果无法建立因特网连接，则用 “offline.html” 替代 /html5/ 目录中的所有文件：

`ALLBACK：/html5/ /404.html`

注释：第一个 URI 是资源，第二个是替补。

## 更新缓存
一旦应用被缓存，它就会保持缓存直到发生下列情况：
- 用户清空浏览器缓存
- manifest 文件被修改
- 由程序来更新应用缓存

## 注意事项
- 站点离线存储的容量限制是5M
- 如果manifest文件，或者内部列举的某一个文件不能正常下载，整个更新过程将视为失败，浏览器继续全部使用老的缓存
- 引用manifest的html必须与manifest文件同源，在同一个域下
- 在manifest中使用的相对路径，相对参照物为manifest文件
- CACHE MANIFEST字符串应在第一行，且必不可少
- 系统会自动缓存引用清单文件的 HTML 文件
- manifest文件中CACHE则与NETWORK，FALLBACK的位置顺序没有关系，如果是隐式声明需要在最前面
- FALLBACK中的资源必须和manifest文件同源
- 当一个资源被缓存后，该浏览器直接请求这个绝对路径也会访问缓存中的资源。
- 站点中的其他页面即使没有设置manifest属性，请求的资源如果在缓存中也从缓存中访问
- 当manifest文件发生改变时，资源请求本身也会触发更新


