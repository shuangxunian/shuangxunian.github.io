---
title: 网页消息通信
excerpt: 如题目
categories:
- 技术文章
tags:
- 网络
---

## 网页消息通信是什么
即在浏览器中，两个不同页面（A页面的window ！= B页面的window）网页之间的消息传递。

## 传递方式
1. WebSocket （可跨域）
    WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
2. postMessage（可跨域）
    window.postMessage()方法安全地启用Window对象之间的跨源通信。
    对将接收消息的窗口的引用，获得此类引用的方法包括：
    - iframe标签
    - Window.open
    - Window.opener
    - Worker打开的页面
3. SharedWorker
    - 先说一下webworker，作为浏览器的一个新特性，可以提供一个额外的线程来执行一些js代码（真正的多线程），并且不会影响到浏览器用户界面，但是不能DOM操作。
    - SharedWorker可以被多个window共同使用，所以可以用来跨页面传输数据，但必须保证这些标签页都是同源的(相同的协议，主机和端口号)。
4. Server-Sent Events
    HTML5 服务器发送事件（server-sent event）允许网页获得来自服务器的更新。
    Server-Sent 事件指的是网页自动获取来自服务器的更新。
    以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。
    服务端 事件流的对应MIME格式为text/event-stream，而且其基于HTTP长连接。针对HTTP1.1规范默认采用长连接，针对HTTP1.0的服务器需要特殊设置。
5. localStorage（可以添加事件监听）
    localstorage是浏览器多个标签共用的存储空间，所以可以用来实现多标签之间的通信(ps：session是会话级的存储空间，每个标签页都是单独的）。 直接在window对象上添加监听即可。
6. Cookies
    Cookies在同一个域名内，并且目录也得相同，可以参考第三方库
7. BroadcastChannel(Chrome商店的api)
    这个方式，只要是在同一原始域和用户代理下，所有窗口、iframe之间都可以进行交互。这个感觉就有点类似于广播了。

## 参考链接
[网页消息通信](https://xv700.gitee.io/message-communication-for-web/#BroadcastChannel)