---
title: css如何去掉input的边框
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

css去掉input边框代码：
```css
input {
    /* 去除未选中状态边框 */
    border: 0;  
    /* 去除选中状态边框 */
    outline: none; 
    /* 透明背景 */
    background-color: rgba(0, 0, 0, 0);
}
```
border 简写属性在一个声明设置所有的边框属性。

可以按顺序设置如下属性：
- border-width：规定边框的宽度。
- border-style：规定边框的样式。
- border-color：规定边框的颜色。

outline （轮廓）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。

注释：轮廓线不会占据空间，也不一定是矩形。

outline 简写属性在一个声明中设置所有的轮廓属性。

可以按顺序设置如下属性：
- outline-color：规定边框的颜色。
- outline-style：规定边框的样式。
- outline-width：规定边框的宽度。

css去掉input边框示例：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
   input {     
       border: 0;  // 去除未选中状态边框
       outline: none; // 去除选中状态边框
       background-color: rgba(0, 0, 0, 0);// 透明背景
   }
    </style>
</head>
<body>
    <input type="text" >
</body>
</html>
```

**参考链接**
[css如何去掉input的边框？](https://www.html.cn/qa/css3/14094.html)