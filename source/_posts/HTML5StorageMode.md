---
title: HTML5存储方式
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- html
---


## Cookies的野蛮生长
在HTML5出现之前，Cookies便占据了客户端存储的整个江山，就像是蛮荒时代的野蛮生长，cookies很好、快速地满足实际应用的需求。但是它的问题也很明显，cookies会在请求头上带着数据，而且大小限制在4K以内，这是非常不安全的，容易被外部截取，还存在domain污染。

IE浏览器特别喜欢搞自己的一套，对于增加存储容量加入了UserData，大小是64K，但是其他浏览器不喜欢跟它玩，也就只有它自己一家支持。

那么，重点来了。既然cookies问题那么多，就要想办法解决，不然没法继续往前发展。首先要确认它的问题有哪些，然后根据这些问题寻找解决方案。
- 解决4K存储容量问题
- 解决请求头带有存储信息的问题，也就是增加安全性，通过加密通道或方式进行数据存储和传输
- 解决关系型存储的问题
- 跨浏览器

## 本地存储localstorage
**存储方式**

以键值对（key-value）的方式存储，永久存储，永不失效，除非手动删除。

**存储容量**

每个域名5M。

**常用的API**
- getItem       取记录
- setItem       设置记录
- removeItem    移除记录
- key           取key所对应的值
- clear         清除记录

## 本地存储sessionstorage
HTML5的本地存储API中的localstorage与sessionstorage在使用方法上是相同的，区别在于sessionstorage在关闭页面后即被清空，而localstorage则会一直保存，除非手动删除。

## 离线缓存（application cache）
本地缓存应用所需的文件

**使用方法**
配置manifest文件

页面上：
```html
<!DOCTYPE HTML>
<html manifest="demo.appcache">
...
</html>
```

**manifest文件：**
manifest是最简单的文本文件，它告知浏览器被缓存的内容（以及不缓存的内容）。
manifest文件分为三个部分：
- CACHE MANIFEST-在此标题下列出的文件将在首次下载后进行缓存
- NETWOrK-在此标题下的文件需要与服务器进行连接，且不会被缓存
- FALLBACK-在此标题下的文件规定当页面无法被访问时的回退页面（比如404页面）

**完整demo:**
```
CACHE MANIFEST
# 2016-07-24 v1.0.0
/theme.css
/main.js

NETWORK:
login.jsp

FALLBACK:
/html/ /offline.html
```

**服务器上：**
manifest文件需要配置正确的MIME-type，即text/cache-manifest。

**常用API:**
核心是applicationCache对象，有个status属性，表示应用缓存的当前状态：
- 0 （UNCACHED）：无缓存，没有和页面相关的应用缓存
- 1 （IDLE）：闲置，应用缓存没有得到更新
- 2 （CHECKING）：检查中，正在下载描述文件并检查更新
- 3 （DOWNLOADING）：下载中，应用缓存正在下载与描述文件中指定的资源
- 4 （UPDATEREADY）：更新完成，所有资源都已经下载完毕
- 5 （IDLE）：废弃，应用缓存的描述文件已经不存在了，因此页面无法再访问应用缓存

**相关事件**
表示应用缓存状态的改变：
- checking：在浏览器为应用缓存查找更新时触发
- error：在检查更新或下载资源期间发生错误时触发
- noupdate：在检查描述文件发现文件无变化时触发
- downloading：在开始下载应用缓存资源时触发
- progress：在文件下载应用缓存的过程中持续不断地下载时触发
- updateready：在页面新的应用缓存下载完毕时触发
- cached：在应用缓存完整可用时触发

**application cache的三个优势：**
- 离线浏览
- 提升页面载入速度
- 降低服务器压力

**注意事项：**
- 浏览器对缓存数据的容量限制可能不太一样（某些浏览器设置的限制是每个站点5M）
- 如果是manifest文件，或者内部列举的某一个文件不能正常下载，整个更新过程将视为失败，浏览器继续全部使用旧的缓存
- 引用manifest的html必须与manifest文件同源，在同一个域下
- 浏览器会自动缓存引用manifest文件的html文件，这就导致了如果更改了html内容，也需要更新版本才能做到最新
- manifest文件中的CACHE与NETWOrK、FALLBACK的位置顺序没有关系，如果是隐式声明需要在最前面
- FALLBACK中的资源必须和manifest文件同源
- 更新完版本后，必须刷新一次才会启动新版本（会出现重刷一次页面的情况），需要添加监听版本事件
- 站点中的其他页面即使没有设置manifest属性，请求的资源如果在缓存中也从缓存中访问
- 当manifest文件发生改变时，资源请求本身也会触发更新

离线缓存和传统浏览器缓存的区别
- 离线缓存是针对整个应用，浏览器缓存是单个文件
- 离线缓存可以主动通知浏览器更新资源

## Web SQL
Web SQL数据库API并不是HTML5规范的一部分，但它是一个独立的规范，引入了一组使用SQL操作客户端数据库的APIs。

**核心方法:**
- openDatabase：使用现有的数据库或新建的数据库创建一个数据库对象
- transaction： 控制一个事务，以及基于这种情况执行提交或回滚
- executeSql：用于执行实际的SQL查询

**打开数据库**
```javascript
var db = openDatabase('mydb', '1.0', 'TEST DB', 2 * 1024 * 1024, fn);
```
**执行查询操作**
```javascript
var db = openDatabase('mydb', '1.0', 'TEST DB', 2 * 1024 * 1024);
db.transaction(function (tx) {
    tx.executeSql('CREATE TABLE IF NOT EXISTS WIN (id unique, name)');
})
```
**插入数据**
```javascript
var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
db.transaction(function (tx) {
   tx.executeSql('CREATE TABLE IF NOT EXISTS WIN (id unique, name)');
   tx.executeSql('INSERT INTO WIN (id, name) VALUES (1, "winty")');
   tx.executeSql('INSERT INTO WIN (id, name) VALUES (2, "LuckyWinty")');
});
```
**读取数据**
```javascript
db.transaction(function (tx) {
   tx.executeSql('SELECT * FROM WIN', [], function (tx, results) {
      var len = results.rows.length, i;
      msg = "<p>查询记录条数: " + len + "</p>";
      document.querySelector('#status').innerHTML +=  msg;

      for (i = 0; i < len; i++){
         alert(results.rows.item(i).name );
      }

   }, null);
});
```

## IndexedDB
索引数据库（IndexedDB）API（作为HTML5的一部分）对创建具有丰富本地存储数据的数据密集型的离线HTML5 Web应用程序很有用，同时它还有助于本地缓存数据，使传统在线Web应用程序（比如移动Web应用程序）能够快速的运行和响应。

**异步API**

在IndexedDB大部分操作并不是我们常用的调用方法（返回结果的模式），而是（请求-响应模式），比如打开数据库的操作

## 参考链接
[HTML5存储方式](https://segmentfault.com/a/1190000011516871)