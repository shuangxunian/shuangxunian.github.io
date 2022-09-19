---
title: js焦点事件:onfocus、onblur、focus()、blur()、select()
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---


## 什么是焦点事件

**焦点：**使浏览器能够区分用户输入的对象，当一个元素有焦点的时候，那么他就可以接收用户的输入

只有能够响应用户操作额元素才可以接收焦点事件，比如：a button input...

- onfocus：当元素获取到焦点的时候触发
    ```javascript
    div.onfocus = funcion(){}
    ```
- onblur:当元素失去焦点的时候
- obj.focus():给指定的元素设置焦点
- obj.blur()：取消指定元素的焦点
- obj.select()：全选当前的文字

```html
<body>
    <input type="text" />
    <button >全选</button>
    <script>
    window.onload = function() {
        var oinpt = document.getElementsByTagName("input")[0];
        var obtn = document.getElementsByTagName("button")[0];
        obtn.onclick = function() {
            oinpt.select();
        }
    }
    </script>
</body>

```

## 参考链接
[js焦点事件:onfocus、onblur、focus()、blur()、select()](https://segmentfault.com/a/1190000006168220)