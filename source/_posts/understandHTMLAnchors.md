---
title: 了解HTML锚点
excerpt: 如题目
categories:
- 技术文章
tags:
- html
- 面试
---

## 概念
&lt;a&gt;元素 (或HTML锚元素, Anchor Element)通常用来表示一个锚点/链接。但严格来说，<a>元素不是一个链接，而是超文本锚点，可以链接到一个新文件、用id属性指向任何元素。如果没有<a>元素没有href属性的话，可以作为原本链接位置的占位符，常用于home链接。
**注意：**任何文档流内容都可以被嵌套，只要不是交互内容类别(如按钮、链接等)

## 属性
### href
href属性表示地址，共包括以下3种：
1. 链接地址
```html
<a href="http://www.baidu.com">百度</a>
```
2. 下载地址
```html
<a href="test.zip">下载测试</a>
```
3. 锚点
    - href:#id名
    ```html
    <a href="#test">目录</a>
    <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
    <div id="test" style="height: 200px;width: 200px; border: 1px solid black;margin-bottom: 300px;">内容</div>
    ```
    - href:页面地址#id名
    ```html
    <a href="http://baike.baidu.com/view/2202.htm#2">足球比赛规则</a>
    ```

**注意：**href属性一定不要留空，若暂时不需要写地址，则写#或javascript:;。若href留空，会刷新页面
**href与src的区别**
    - href(hypertext reference)指超文本引用，表示当前页面引用了别处的内容
    - src(source)表示来源地址，表示把别处的内容引入到当前页面
    - 所以<img>、<script>、<iframe>等应该使用src，而<a>和<map>应该使用href

4. 手机号码
在移动端，使用以下代码可以唤出手机拨号盘
```html
<a href="tel:12345678910"> 12345678910 </a>
```

### target
target属性表示链接打开方式：
1. _self    当前窗口（默认）
2. _blank    新窗口
3. _parent 父框架集
4. _top 整个窗口
5. _framename 指定框架

```html
//外层框架
<frameset cols = "20%, *">
    <frame src="left.html">
    <frame src="right.html">
</frameset>

//里层框架
<frameset rows = "50%,*">
    <frame src="top.html">
    <frame src="bottom.html" name="bottom">        
</frameset>

//锚点页
<ul class="list">
    <li class="in"><a href="chap1.html" target="_self">chap1(_self)</a></li>
    <li class="in"><a href="chap2.html" target="_blank">chap2(_blank)</a></li>
    <li class="in"><a href="chap3.html" target="_parent">chap3(_parent)</a></li>
    <li class="in"><a href="chap4.html" target="_top">chap4(_top)</a></li>    
    <li class="in"><a href="chap5.html" target="bottom">chap5(framename)</a></li>
</ul>
```

### download
download属性用来设置下载文件的名称(firefox/chrome/opera支持)
```html
<a href="test.zip" download="gogo">test</a>
```

### rel
rel属性表示表示链接间的关系

| 名称     | 功能     |
| ---- | ---- |
|alternate   |相较于当前文档可替换的呈现|
|author      |链接到当前文档或文章的作者|
|bookmark    |链接最近的父级区块的永久链接|
|help        |与当前上下文相关的帮助链接|
|license     |当前文档的许可证|
|next        |后一篇文档|
|prev        |前一篇文档|
|nofollow    |当前文档的原始作者不推荐超链接指向的文档|
|noreferer   |访问时链接时不发送referer字段|
|prefetch    |预加载链接指向的页面(对于chrome使用prerender)|
|search      |用于搜索当前文档或相关文档的资源|
|tag         |给当前文档打上标签|


**应用：**当一篇篇幅很长的文章需要多页显示时，配合next或prev可以实现前后页面导航的预加载
```html
<a href="prev.html" rel="prev prefetch prerender">前一页</a>
<a href="next.html" rel="next prefetch prerender">后一页</a>
    //当然prefetch也可以用于预加载其他类型的资源
<link rel="prefetch prerender" href="test.img">
```

### 注意事项
1. <a>标签的文本颜色只能自身进行设置，从父级继承不到
2. <a>标签的下划线颜色跟随文本颜色进行变化
3. <a>标签不可嵌套<a>标签
```html
<div style="color: red;">
    <a href="#">[1]从父级继承不到红色字体</a>
    <br>
    <a href="#" style="color: green">[2]下划线颜色与文本同色</a>
    <br>
    <a href="#">前面<a href="#">[3]a标签不可嵌套</a></a>
</div>
```

## 参考链接
[了解HTML锚点](https://www.cnblogs.com/xiaohuochai/p/5007282.html)