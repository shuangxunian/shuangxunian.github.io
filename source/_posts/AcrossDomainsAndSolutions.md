---
title: 什么是跨域以及几种简单解决方案
excerpt: 跨域是什么与两种解决方法
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 什么是跨域？
要明白什么是跨域之前，首先要明白什么是同源策略？
同源策略就是用来限制从一个源加载的文档或脚本与来自另一个源的资源进行交互。那怎样判断是否是同源呢？
如果协议，端口（如果指定了）和主机对于两个页面是相同的，则两个页面具有相同的源，也就是同源。也就是说，要同时满足以下3个条件，才能叫同源：
1. 协议相同
2. 端口相同
3. 主机相同

举个例子：
我们来看下面的页面是否与 http://store.company.com/dir/index.html 是同源的？
1. http://store.company.com/dir/index2.html 同源
2. http://store.company.com/dir2/index3.html 同源 虽然在不同文件夹下
3. https://store.company.com/secure.html 不同源 不同的协议(https)
4. http://store.company.com:81/dir/index.html 不同源 不同的端口(81)
5. http://news.company.com/dir/other.html 不同源 不同的主机(news)

所以当面对跨域问题的时候，有什么解决方案呢？

## 跨域的几种解决方案
### document.domain方法
我们来看一个具体场景：有一个页面 http://www.example.com/a.html ，它里面有一个iframe，这个iframe的源是 http://example.com/b.html ，很显然它们是不同源的，所以我们无法在父页面中操控子页面的内容。

解决方案如下：
```html
<!-- b.html -->
<script>
    document.domain = 'example.com';
</script>
<!-- a.html -->
<script>
    document.domain = 'example.com';
    var iframe = document.getElementById('iframe').contentWindow.document;
    //后面就可以操作iframe里的内容了...
</script>
```
所以我们只要将两个页面的document.domain设置成一致就可以了，要注意的是，document.domain的设置是有限制的，我们只能把document.domain设置成自身或更高一级的父域。

但是，这种方法只能解决主域相同的跨域问题。

### window.name方法
window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。

我们来看一个具体场景，在一个页面 example.com/a.html 中，我们想获取 data.com/data.html 中的数据，以下是解决方案：
```html
<!-- data.html -->
<script>
    window.name = 'data'; //这是就是我们需要通信的数据
</script>
<!-- a.html -->
<html>
<head>
<script>
    function getData () {
        var iframe = document.getElementById('iframe');
        iframe.src = 'example.com/b.html'; // 这里让iframe与父页面同源
        
        iframe.onload = function () {
            var data = iframe.contentWindow.name; //在这里我们得到了跨域页面中传来的数据
        };
    }
</script>
</head>
<body>
</body>
</html>
```

### JSONP方法
JONSP(JSON with Padding)是JSON的一种使用模式。基本原理如下：
```html
<!-- a.html -->
<script>
    function dealData (data) {
        console.log(data);
    }
</script>

<script src='http://example.com/data.php?callback=dealData'></script>
```
```php
<?php
    $callback = $_GET['callback'];
    $data = 'data';
    echo $callback.'('.json_encode($data).')';
?>
```
这时候在a.html中我们得到了一条js的执行语句dealData('data')，从而达到了跨域的目的。

所以JSONP的原理其实就是利用引入script不限制源的特点，把处理函数名作为参数传入，然后返回执行语句，仔细阅读以上代码就可以明白里面的意思了。

如果在jQuery中用JSONP的话就更加简单了：
```html
<script>
$.getJSON('http://example.com/data.php?callback=?', function (data) {
    console.log(data);
});
</script>
```

### CORS(更通用更强大)
#### 介绍
关于跨域问题有很多的解决方案，这里我们总结一下目前最通用最强大的解决方案：CORS。

W3C 的 Web 工作组推荐了一种新的机制，即跨域资源共享（Cross-origin Resource Sharing），简称CORS。其实这个机制就是实现了跨站访问控制，使得安全地进行跨站数据传输成为可能。

跨源资源共享标准( cross-origin sharing standard) 使得以下场景可以使用跨站 HTTP 请求：
1. 使用 XMLHttpRequest 或 Fetch发起跨站 HTTP 请求。
2. Web 字体 (CSS 中通过 @font-face 使用跨站字体资源)，因此，网站就可以发布 TrueType 字体资源，并只允许已授权网站进行跨站调用。
3. WebGL 贴图
4. 使用drawImage绘制
5. Images/video 画面到canvas.
6. 样式表（使用 CSSOM）
7. Scripts (for unmuted exceptions)

CORS分为简单请求和复杂请求，处理方法也是有不同的，所以我们分别总结。

#### 简单请求
什么是简单请求呢？同时满足以下两个条件，就是简单请求：
1. 请求是下列之一：
- HEAD
- GET
- POST
2. HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

实现方法非常简单，只需要把服务器的响应报文里的Access-Control-Allow-Origin设置为*或者包含由 Origin指明的站点。

Access-Control-Allow-Origin是HTTP响应报文中的一个字段，Origin是HTTP请求报文中的以一个字段,如果不清楚这两个字段的话，可以自行查阅关于HTTP报文的知识，比如HTTP | MDN。

#### 复杂请求
如果不是简单请求，那就是复杂请求，比如请求的方法是PUT或者DELETE，比如Content-Type字段的类型是application/json，比如设置了自定义头信息。

复杂请求就是比简单请求多了个预检请求（preflight）而已。

预检请求就是浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

预检请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。除了Origin字段，还有两个字段非常重要：Access-Control-Request-Method和Access-Control-Request-Headers，分别表示允许的请求方法和请求头。

举一个具体的例子：

现在，我们有一个页面向服务器发送了一个POST请求，并且我们自己定义了一个请求头字段My-HEADER，这时候浏览器就会首先发送一个OPTION请求来做预检请求，请求头里有以下字段：
```
Access-Control-Request-Method: POST
Access-Control-Request-Headers: My-HEADER
```

如果预检请求成功的话，响应头里的内容如下：
```
Access-Control-Allow-Origin: http://example.com //表明服务器允许http://example.com的请求
Access-Control-Allow-Methods: POST, GET, OPTIONS //表明服务器可以接受POST, GET和 OPTIONS的请求方法
Access-Control-Allow-Headers: My-HEADER //传递一个可接受的自定义请求头列表
Access-Control-Max-Age: 3000000 //告诉浏览器，本次预检请求的响应结果有效时间是多久
```

## 参考链接
[什么是跨域以及几种简单解决方案](https://segmentfault.com/a/1190000013278814)
[跨域问题的根本解决方案CORS](https://segmentfault.com/a/1190000013285839)