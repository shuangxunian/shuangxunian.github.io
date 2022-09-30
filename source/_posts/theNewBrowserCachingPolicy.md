---
title: 新的浏览器缓存策略变更：舍弃性能、确保安全
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- 浏览器
---

## 前言
通常，缓存可以通过存储数据来提高性能，从而可以更快后面相同数据的请求。例如，来自网络的缓存资源可以避免频繁的和服务器交互。缓存计算结果可以省去进行相同计算的时间。
在 Chrome 中，缓存机制以多种方式使用，HTTP 缓存就是一个示例。

## Chrome 的 HTTP 缓存当前的工作方式
从 85 版开始，Chrome 会使用它们各自的资源URL作为缓存键来缓存从网络获取的资源。
下面我们来看几个示例：
用户访问了页面(https://a.example)，然后请求了一个图像(https://x.example/doge.png)。该图像是从网络请求的，浏览器会使用 https://x.example/doge.png 用作 key 进行缓存。
同一用户访问另一个页面（https://b.example），这个页面请求了相同的图像（https://x.example/doge.png）。浏览器使用图像 URL 作为 key ，检查其 HTTP 缓存是否已经缓存了此资源。浏览器在其缓存中找之前缓存的资源，因此它使用了资源的缓存版本。
图像是否从 iframe 中加载都没有关系。如果网站 https://c.example 使用 iframe（https://d.example）访问另一个网站，并且 iframe 中请求了相同的图片（https://x.example/doge.png） ，则浏览器仍可以从缓存中加载图片，因为所有页面的缓存 key 均相同。

## 缓存机制存在的问题
从性能的角度来看，这种机制已经运行了很长时间了。但是，网站响应 HTTP 请求所花费的时间可以表明浏览器过去曾经访问过相同的资源，这使浏览器容易受到安全和隐私的攻击，比如：
- 检测用户是否访问过特定站点：攻击者可以通过检查缓存是否具有特定于特定站点或一组站点的资源来检测用户的浏览历史记录。
- 跨站点搜索攻击：攻击者可以通过检查特定网站使用的“无搜索结果”图像是否在浏览器的缓存中来检测用户的搜索结果中是否包含任意字符串。
- 跨站点跟踪：缓存可用于存储类似 cookie 的标识符，作为跨站点跟踪机制。

为了减轻这些风险，Chrome 将从 Chrome 86 开始对 HTTP 缓存进行分区。

## 缓存分区将如何影响 Chrome 的 HTTP 缓存？
通过缓存分区，除了资源 URL 外，还将使用新的 “网络隔离密钥” 来对缓存的资源进行密钥设置。网络隔离密钥由顶级站点和当前 frame 中的站点组成。
**注意：**“站点”使用 “scheme://eTLD+1 ”识别，因此，如果请求来自不同的页面，但是它们具有相同的 scheme 和有效的 eTLD+1，则它们将使用相同的缓存分区。
再次查看前面的示例，以了解缓存分区如何在不同的上下文中工作：
用户访问 https://a.example 请求图像（https://x.example/doge.png）。在这种情况下，图像是从网络请求的，并使用由 https://a.example（顶级站点）， https://a.example（当前 frame 中的站点）和 https://x.example/doge.png（资源URL）组成的元组作为 key 进行缓存。（请注意，当资源请求来主页面时，网络隔离密钥中的顶级站点和当前 frame 中的站点是相同的。）
同一用户访问了 https://b.example 请求相同图片（https://x.example/doge.png）。尽管在上一个示例中加载了相同的图像，但是由于密钥不匹配，因此不会被缓存命中。
现在用户回到了 https://a.example ，但是这次图像（https://x.example/doge.png）被嵌入到了 iframe 中。在这种情况下，图片缓存的 key 和直接在主页面加载的图片的缓存 key 是相同的，因此可以使用之前缓存的图片资源。
这个例子中，图像在 https://c.example 的 iframe 中加载，在这种情况下，图像是从网络上下载的，因为缓存中找不到相同的密钥。
如果域包含子域或端口号怎么办？用户访问 https://subdomain.a.example ，其中嵌入的 iframe (https://c.example:8080) 请求了图像的。
由于密钥是基于 scheme://eTLD+1 创建的，因此将忽略子域和端口号。所以本次发生缓存命中。
如果 iframe 多次嵌套该怎么办？用户访问 https://a.example，其中嵌入了一个 iframe（https://b.example），它又嵌入了另一个 iframe（https://c.example），这个 iframe 最终请求了图像。
由于密钥是从 https://a.example 加载资源的顶部 frame 和直接frame （https://c.example）获取的，因此会发生缓存命中。

## 对现有网站的影响
这不是一个重大变化，但可能会影响某些网页的性能。
例如，在许多站点上为大量可高度缓存的资源提供服务的站点（例如字体和流行的脚本）可能会看到其流量增加。同样，使用此类服务的人可能会越来越依赖于它们。
下面是一些性能指标的变化：
- 整体缓存未命中率增加了约 3.6％
- FCP 增加约 0.3％
- 从网络加载的字节的总体比例增加了约 4％

## 其他浏览器的行为
- Chrome: 使用顶级 scheme://eTLD+1 加 frame scheme://eTLD+1
- Safari: 使用顶级 eTLD+1
- Firefox: 计划实施顶级 scheme://eTLD+1 然后也考虑像 Chrome 一样增加第二个 key

## 参考链接
[新的浏览器缓存策略变更：舍弃性能、确保安全](https://mp.weixin.qq.com/s/s0E88ClBXEiayU7xSy0XPg)