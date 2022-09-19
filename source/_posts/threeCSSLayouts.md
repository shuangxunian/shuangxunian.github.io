---
title: CSS-水平居中、垂直居中、水平垂直居中
excerpt: 本文介绍几种常用的居中方法。
categories:
- 技术文章
tags:
- html
- css
- 面试
---

## 水平居中
水平居中可分为行内元素水平居中和块级元素水平居中

### 行内元素水平居中
这里行内元素是指文本text、图像img、按钮超链接等，只需给父元素设置text-align:center即可实现。

```css
.center{
    text-align:center;
}
```

### 块级元素水平居中
#### 定宽块级元素水平居中
只需给需要居中的块级元素加margin:0 auto即可，但这里需要注意的是，这里块状元素的宽度width值一定要有。

```css
.center{
    width:200px;
    margin:0 auto;
    border:1px solid red;
}
```

#### 不定宽块级元素水平居中
不定宽，即块级元素宽度不固定

**方法1：设置table**
通过给要居中显示的元素，设置display:table，然后设置margin:0 auto来实现

```css
.center{
    display:table;
    margin:0 auto;
    border:1px solid red;
}
```

**方法2：设置inline-block**
子元素设置inline-block，同时父元素设置text-align:center

```html
<style>
.center{
    text-align:center;
}
.inlineblock-div{
    display:inline-block;
}
</style>
<div class="center">
    <div class="inlineblock-div">1</div>
    <div class="inlineblock-div">2</div>
</div>
```

**方法3：设置flex布局**
只需把要处理的块状元素的父元素设置display:flex,justify-content:center;

```html
<style>
.center{
    display:flex;
    justify-content:center;
}
</style>
<div class="center">
    <div class="flex-div">1</div>
    <div class="flex-div">2</div>
</div>
```

**方法4：position + 负margin；**
**方法5：position + margin：auto；**
**方法6：position + transform；**
**注**：这里方法4、5、6同下面垂直居中一样的道理，只不过需要把top/bottom改为left/right，在垂直居中部分会详细讲述。

## 垂直居中
### 单行文本垂直居中
对于单行文本，我们只需要将文本行高(line-height)和所在区域高度(height)设为一致即可：

```html
<style>
#div1{
    width: 300px;
    height: 100px;
    margin: 50px auto;
    border: 1px solid red;
    line-height: 100px; /*设置line-height与父级元素的height相等*/
    text-align: center; /*设置文本水平居中*/
    overflow: hidden; /*防止内容超出容器或者产生自动换行*/
}
</style>
<div id="div1">
    这是单行文本垂直居中
</div>
```
### 多行文本垂直居中
多行文本垂直居中分为两种情况，一个是父级元素高度不固定，随着内容变化；另一个是父级元素高度固定。

#### 父级元素高度不固定
父级高度不固定的时，高度只能通过内部文本来撑开。这样，我们可以通过设置内填充（padding）的值来使文本看起来垂直居中，只需设置padding-top和padding-bottom的值相等：

```html
<style>
#div1{
    width: 300px;
    margin: 50px auto;
    border: 1px solid red;
    text-align: center; /*设置文本水平居中*/
    /*       上下  左右   */
    padding: 50px 20px;
}
</style>
<div id="div1">
        这是多行文本垂直居中，
        这是多行文本垂直居中，
        这是多行文本垂直居中，
        这是多行文本垂直居中。
</div>
```

#### 父级元素高度固定
本文一开始就提到css中的vertical-align属性，但是它只对拥有valign特性的元素才生效，结合display: table;，可以使得div模拟table属性。因此我们可以设置父级div的display属性：display: table;；然后再添加一个div包含文本内容，设置其display:table-cell;和vertical-align:middle;。具体代码如下：

```html
<style>
#outer{
    width: 400px;
    height: 200px;
    margin: 50px auto;
    border: 1px solid red;
    display: table;
}
#middle{ 
    display:table-cell; 
    vertical-align:middle;  
    text-align: center; /*设置文本水平居中*/  
    width:100%;   
}
</style>
<div id="outer">
    <div id="middle">
        这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中。
    </div>
</div>
```

**注：**IE7及以下会效果失效。

### 块级元素垂直居中
#### 方法1：flex布局
在需要垂直居中的父元素上，设置display:flex和align-items：center
**要求：**父元素必须显示设置height值

```html
<style>
.center{
    width:200px;
    height:300px;
    display:flex;
    align-items:center;
    border:2px solid blue;
}
</style>
<div class="center">
    <div>垂直居中</div>
</div>
```

#### 利用position和top和负margin（需知宽高）
设置元素为absolute/relative/fixed
margin=负一半

```html
<style>
.parent{
    position:relative;
    height:200px;
    width:200px;
    border:1px solid blue;
}
.child{
    position:absolute;
    height:100px;
    width:150px;
    top:50px;
    margin-top:-50px;
    line-height:100px;
    background-color:red;
}
</style>
<div class="parent">
    <div class="child">已知高度垂直居中</div>
</div>
```

#### 利用position和top/bottom和margin:auto

```html
<style>
.parent{
    position:relative;
    height:200px;
    width:200px;
    border:1px solid blue;
}
.child{
    position:absolute;
    height:10px;
    width:150px;
    top:0;
    bottom:0;
    margin:auto;
    line-height:100px;
    background-color:red;
}
</style>
<div class="parent">
    <div class="child">已知高度垂直居中</div>
</div>
```

#### 利用position和top和transform
transform中translate偏移的百分比就是相对于元素自身的尺寸而言的。

```html
<style>
.parent{
    position:relative;
    height:200px;
    width:200px;
    border:1px solid blue;
}
.child{
    position:absolute;
    top:50%;
    transform:translate(0,-50%);
    line-height:100px;
    background-color:red;
}
</style>
<div class="parent">
    <div class="child">已知高度垂直居中</div>
</div>
```

**注：**
- 上述的块级垂直居中方法，稍加改动，即可成为块级水平居中方法，如top/bottom换成left/right
- transform方法，可用于未知元素大小的居中

## 水平垂直居中
### 方法1：绝对定位+margin:auto

```html
<style>
div{
    width: 200px;
    height: 200px;
    background: green;
    
    position:absolute;
    left:0;
    top: 0;
    bottom: 0;
    right: 0;
    margin: auto;
}
</style>
```

### 绝对定位+负margin

```html
<style>
    div{
        width:200px;
        height: 200px;
        background:green;
        
        position: absolute;
        left:50%;
        top:50%;
        margin-left:-100px;
        margin-top:-100px;
    }
</style>
```

### 绝对定位+transform

```html
<style>
    div{
        width: 200px;
        height: 200px;
        background: green;
        
        position:absolute;
        left:50%;    /* 定位父级的50% */
        top:50%;
        transform: translate(-50%,-50%); /*自己的50% */
    }
</style>
```

### flex布局

```html
<style>
   .box{
         height:600px;  
         
         display:flex;
         justify-content:center;  //子元素水平居中
         align-items:center;      //子元素垂直居中
           /* aa只要三句话就可以实现不定宽高水平垂直居中。 */
    }
    .box>div{
        background: green;
        width: 200px;
        height: 200px;
    }
</style>
```

### table-cell实现居中

```html
<style>
    div{
        display:table-cell;
        text-align:center;
        vertical-align: middle;
    }
</style>
```

## 参考链接
[CSS-水平居中、垂直居中、水平垂直居中](https://segmentfault.com/a/1190000014116655)
[css 文本和div垂直居中方法汇总](https://blog.csdn.net/u014607184/article/details/51820508)