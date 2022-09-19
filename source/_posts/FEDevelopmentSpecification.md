---
title: 前端开发规范
excerpt: 如题目
categories:
- 技术文章
tags:
- 其他
---


## 原因
确定开发规范的好处有：
- 通过代码风格的一致性，降低维护代码的成本以及改善多人协作的效率。同时遵守最佳实践，确保页面性能得到最佳优化和高效的代码。
- 一个团队的代码风格如果统一了，首先可以培养良好的协同和编码习惯，其次可以减少无谓的思考，再次可以提升代码质量和可维护性。

## 通用书写规范
### 1. 基本原则：结构、样式、行为分离
- 尽量确保文档和模板只包含 HTML 结构，样式都放到 css 样式表里，行为都放到 js 脚本里
- 标记应该是结构良好、语义正确。
- Javascript应该起到渐进式增强用户体验的作用 。

### 2.文件/资源命名
在 web 项目中，使用驼峰命名法命名所有文件。

### 3.省略外链资源 URL 协议部分
省略外链资源（图片及其它媒体资源）URL 中的 http / https 协议，使 URL 成为相对地址，避免Mixed Content 问题。

![](https://api2.mubu.com/v3/document_image/fbecd6fc-9e64-4c8f-8ed7-757141b8655d-3807603.jpg)

### 4.写注释
写注释时请一定要注意：写明代码的作用，重要的地方一定记得写注释。 没必要每份代码都描述的很充分，它会增重HTML和CSS的代码。这取决于该项目的复杂程度。
1. 单行注释说明：
    ```javascript
    // 单行注释以两个斜线开始，后接一个空格，然后开始写内容，以行尾结束
    // 调用了一个函数
    setTitle();
    var maxCount = 10; // 设置最大量
    ```
2. 多行注释
    ```javascript
    // vscode的快捷键是alt + shift + A
    /*
    * 代码执行到这里后会调用setTitle()函数
    * setTitle()：设置title的值
    */
    ```
3. 函数注释
    ```javascript
    /**
    * 以星号开头，紧跟一个空格，第一行为函数说明 
    * @param {类型} 参数 单独类型的参数
    * @param {[类型|类型|类型]} 参数 多种类型的参数
    * @param {类型} [可选参数] 参数 可选参数用[]包起来
    * @return {类型} 说明
    * @author 作者 创建时间 修改时间（短日期）改别人代码要留名
    * @example 举例（如果需要）
    */
    ```
4. 文件头注释
    推荐：VScode 文件头部自动生成注释插件：koroFileHeader
    快捷键为：ctrl + alt + I
    ![](https://api2.mubu.com/v3/document_image/fe3c5592-cfdf-4dad-bcde-559666bf6fc5-3807603.jpg)
5. 条件注释
    <!--[if IE 9]>
        .... some HTML here ....
    <![endif]-->

## HTML规范
### 1.通用约定
#### 标签
- 自闭合（self-closing）标签，无需闭合 ( 例如： img input br hr 等 )；
- 可选的闭合标签（closing tag），需闭合 ( 例如：</li> 或 </body> )；
- 尽量减少标签数量；

#### Class 与 ID
- class 应以功能或内容命名，不以表现形式命名；
- class 与 id 单词字母小写，多个单词组成时，采用驼峰命名法；
- 使用唯一的 id 作为 Javascript hook, 同时避免创建无样式信息的 class；

eg:
```html
<!-- Not recommended -->
<div class="itemHook left contentWrapper"></div>

<!-- Recommended -->
<div id="itemHook" class="sidebar contentWrapper"></div>
```

#### 属性顺序
HTML 属性应该按照特定的顺序出现以保证易读性。
- id
- class
- name
- src, for, type, href
- title, alt
- role

#### 嵌套
&lt;a&gt;不允许嵌套 &lt;div&gt; 这种约束属于语义嵌套约束，与之区别的约束还有严格嵌套约束，比如&lt;a&gt;不允许嵌套 &lt;a&gt;。 严格嵌套约束在所有的浏览器下都不被允许；而语义嵌套约束，浏览器大多会容错处理，生成的文档树可能相互不太一样。

#### 语义嵌套约束
- &lt;li&gt;用于&lt;ul&gt;或 &lt;ol&gt;下；
- &lt;dd&gt;, &lt;dt&gt;用于&lt;dl&gt;下；
- &lt;thead&gt;,&lt;tbody&gt;, &lt;tfoot&gt;, &lt;tr&gt;, &lt;td&gt; 用于&lt;table&gt; 下；

#### 严格嵌套约束
- inline-Level 元素，仅可以包含文本或其它inline-Level元素;
- &lt;a&gt;里不可以嵌套交互式元素&lt;a&gt;、&lt;button&gt;、&lt;select&gt;等;
- &lt;p&gt;里不可以嵌套块级元素&lt;div&gt;、&lt;h1&gt;~&lt;h6&gt;、&lt;p&gt;、&lt;ul&gt;/&lt;ol&gt;/&lt;li&gt;、&lt;dl&gt;/&lt;dt&gt;/&lt;dd&gt;、&lt;form&gt;等。

### 2.基本文本
- 使用 HTML5 doctype，不区分大小写。
    ```html
    <!DOCTYPE html>
    ```
- 声明文档使用的字符编码
    ```html
    <meta charset="utf-8">
    ```
- 优先使用 IE 最新版本和 Chrome
    ```html
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    ```

### 3.SEO 优化
- 页面描述
- 每个网页都应有一个不超过 150 个字符且能准确反映网页内容的描述标签。
    ```html
    <meta name="description" content="不超过150个字符">  <!-- 页面描述 -->
    ```
- 页面关键词
    ```html
    <meta name="keywords" content="">  <!-- 页面关键词 -->
    ```
- 定义页面标题
    ```html
    <title>标题</title>
    ```
- 定义网页作者
    ```html
    <meta name="author" content="name, email@gmail.com">  <!-- 网页作者 -->
    ```
- 定义网页搜索引擎索引方式，robotterms是一组使用英文逗号「,」分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和follow。
    ```html
    <meta name="robots" content="index,follow">  <!-- 搜索引擎抓取 -->
    ```

### 4.可选标签
- 为移动设备添加 viewport
    ```html
    <meta name ="viewport" content ="initial-scale=1, maximum-scale=3, minimum-scale=1, user-scalable=no"> <!-- `width=device-width` 会导致 iPhone 5 添加到主屏后以 WebApp 全屏模式打开页面时出现黑边 http://bigc.at/ios-webapp-viewport-meta.orz --> 
    ```
- content 参数：
    - width viewport 宽度(数值/device-width)
    - height viewport 高度(数值/device-height)
    - initial-scale 初始缩放比例
    - maximum-scale 最大缩放比例
    - minimum-scale 最小缩放比例
    - user-scalable 是否允许用户缩放(yes/no)
- 页面窗口自动调整到设备宽度，并禁止用户缩放页面
    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    ```
- 电话号码识别
    iOS Safari ( Android 或其他浏览器不会) 会自动识别看起来像电话号码的数字，将其处理为电话号码链接，比如：
    - 7位数字，形如：1234567
    - 带括号及加号的数字，形如：(+86)123456789
    - 双连接线的数字，形如：00-00-00111
    - 11位数字，形如：13800138000

    ![](https://api2.mubu.com/v3/document_image/b4fb2292-b6bb-4542-b6a9-7336c6e292a4-3807603.jpg)
- 邮箱地址的识别
    在 Android （ iOS 不会）上，浏览器会自动识别看起来像邮箱地址的字符串，不论有你没有加上邮箱链接，当你在这个字符串上长按，会弹出发邮件的提示。
    ![](https://api2.mubu.com/v3/document_image/22b8e001-b2f8-4a6f-ae7e-a412622b2c4e-3807603.jpg)

### 5.样式表、脚本加载
css 样式表在head标签内引用，js 脚本引用在 body 结束标签之前引用。

## CSS规范
### 1.通用约定
#### 声明顺序
相关的属性声明应当归为一组，并按照下面的顺序排列：
1. Positioning
2. Box model
3. Typographic
4. Visual
5. Misc

由于定位（positioning）可以从正常的文档流中移除元素，并且还能覆盖盒模型（box model）相关的样式，因此排在首位。盒模型排在第二位，因为它决定了组件的尺寸和位置。
其他属性只是影响组件的 内部 或者是不影响前两组属性，因此排在后面。

> 完整的属性列表及其排列顺序请参考 Bootstrap property order for Stylelint。

eg:
```css
.declaration-order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* Box-model */
  display: block;
  float: right;
  width: 100px;
  height: 100px;

  /* Typography */
  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  color: #333;
  text-align: center;

  /* Visual */
  background-color: #f5f5f5;
  border: 1px solid #e5e5e5;
  border-radius: 3px;

  /* Misc */
  opacity: 1;
}
```

#### 链接的样式顺序
```
a:link -> a:visited -> a:hover -> a:active（LoVeHAte）
```

#### 命名
- 命名必须由单词、中划线①数字组成；
- 使用驼峰命名法
- 不允许使用拼音（约定俗成的除外，如：youku, baidu），尤其是缩写的拼音、拼音与英文的混合。 

不推荐：
```css
.xiangqing { sRules; }
.news_list { sRules; }
.zhuti { sRules; }
```
推荐
```css
.detail { sRules; }
.news-list { sRules; }
.topic { sRules; }
```

>①使用中划线 “-” 作为连接字符，而不是下划线 "_"。 2种方式都有不少支持者，但 "-" 能让你少按一次shift键，并且更符合CSS原生语法，所以只选一种目前普遍使用的方式

#### 词汇规范
- 不依据表现形式来命名；
- 可根据内容来命名；
- 可根据功能来命名。

不推荐：
```
left, right, center, red, black
```
推荐：
```
nav, aside, news, type, search
```

#### 缩写规范
- 保证缩写后还能较为清晰保持原单词所能表述的意思；
- 使用业界熟知的或者约定俗成的。

不推荐：
```
navigation   =>  navi 
header       =>  head
description  =>  des
```
推荐：
```
navigation   =>  nav 
header       =>  hd
description  =>  desc
```

#### 前缀规范

|   前缀   |   说明   |   示例   |
| ---- | ---- | ---- |
|   g-   |   全局通用样式命名，前缀g全称为global，一旦修改将影响全站样式   |   g-mod   |
|   m-   |   模块命名方式   |   m-detail   |
|   ui-   |   组件命名方式   |  ui-selector    |
|   js-   |   所有用于纯交互的命名，不涉及任何样式规则。JSer拥有全部定义权限   |   	js-switch   |

选择器必须是以某个前缀开头

不推荐：
```css
.info { sRules; }
.current { sRules; }
.news { sRules; }
```

> 这样将会带来不可预知的管理麻烦以及沉重的包袱。你永远也不会知道哪些样式名已经被用掉了，如果你是一个新人，你可能会遭遇，你每定义个样式名，都有同名的样式已存在，然后你只能是换样式名或者覆盖规则。

推荐：
```css
.m-detail .info { sRules; }
.m-detail .current { sRules; }
.m-detail .news { sRules; }
```

> 所有的选择器必须是以 g-, m-, ui- 等有前缀的选择符开头的，意思就是说所有的规则都必须在某个相对的作用域下才生效，尽可能减少全局污染。js- 这种级别的className完全交由JSer自定义，但是命名的规则也可以保持跟重构一致，比如说不能使用拼音之类的

#### 选择器权重等级
a = 行内样式style。 b = ID选择器的数量。 c = 类、伪类和属性选择器的数量。 d = 类型选择器和伪元素选择器的数量。

|   选择器                      |等级 (a,b,c,d)|
|   ----                        |    ----     |
|   style=””                    |   1,0,0,0     |
|   #wrapper #content {}        |   0,2,0,0     |
|   #content .dateposted {}     |   0,1,1,0     |
|   div#content {}              |   0,1,0,1     |
|   #content p {}               |   0,1,0,1     |
|   #content {}                 |   0,1,0,0     |
|   p.comment .dateposted {}    |   0,0,2,1     |
|   div.comment p {}            |   0,0,1,2     |
|   .comment p {}               |   0,0,1,1     |
|   p.comment {}                |   0,0,1,1     |
|   .comment {}                 |   0,0,1,0     |
|   div p {}                    |   0,0,0,2     |
|   p {}                        |   0,0,0,1     |

#### 简写/省略/缩写
**属性值书写尽量使用缩写**
CSS很多属性都支持缩写shorthand （例如 font ） 尽量使用缩写，甚至只设置一个值。 使用缩写可以提高代码的效率和方便理解。
![](https://api2.mubu.com/v3/document_image/72c6e807-1653-4e90-be26-e7bee8b3d199-3807603.jpg)

**省略0后面的单位**
非必要的情况下 0 后面不用加单位。
```css
div{
  padding: 0;
  margin: 0;
}
```

**省略0开头小数点前面的0**
值或长度在-1与1之间的小数，小数前的 0 可以忽略不写。
```css
p{
  font-size: .8em;
}
```

**省略URI外的引号**
不要在 url() 里用 ( “” , ” ) 。
```css
@import url(//www.google.com/css/go.css); 
```

**十六进制尽可能使用3个字符**
加颜色值时候会用到它，使用3个字符的十六进制更短与简洁。
```css
/* Not recommended */
color: #eebbcc;

/* Recommended*/
color: #ebc;
```

### 2.Less 和 Sass
#### 操作符
为了提高可读性，在圆括号中的数学计算表达式的数值、变量和操作符之间均添加一个空格。
```less
// Not recommended
.element {
  margin: 10px 0 @variable*2 10px;
}
// Recommended
.element {
  margin: 10px 0 (@variable * 2) 10px;
}
```

#### 嵌套
避免不必要的嵌套。因为虽然可以使用嵌套，但是并不意味着应该使用嵌套。只有在必须将样式限制在父元素内（也就是后代选择器），并且存在多个需要嵌套的元素时才使用嵌套。
```less
// Not recommended
.table > thead > tr > th {...}
.table > thead > tr > td {...}
// Recommended
.table > thead > tr > {
  > th {...}
  > td {...}
}
```

#### 代码组织
代码按一下顺序组织：
1. @import
2. 变量声明
3. 样式声明

```less
@import "mixins/size.less";

@default-text-color: #333;

.page {
  width: 960px;
  margin: 0 auto;
}
```

#### 混入（Mixin）
1. 在定义 mixin 时，如果 mixin 名称不是一个需要使用的 className，必须加上括号，否则即使不被调用也会输出到 CSS 中。
2. 如果混入的是本身不输出内容的 mixin，需要在 mixin 后添加括号（即使不传参数），以区分这是否是一个 className。

![](https://api2.mubu.com/v3/document_image/205a84d2-3e9b-4bf0-a04d-a8f4ebda7f41-3807603.jpg)

#### 模块化
- 每个模块必须是一个独立的样式文件，文件名与模块名一致；
- 模块样式的选择器必须以模块名开头以作范围约定；

> 任何超过3级的选择器，需要思考是否必要，是否有无歧义的，能唯一命中的更简短的写法

### 3.hack规范
**什么是CSS Hack？**
由于不同的浏览器，比如IE 6，IE 7，Mozilla Firefox等，对CSS的解析认识不一样，因此会导致生成的页面效果不一样，不同内核的浏览器之间页面效果存在差异。 这个时候我们就需要针对不同的浏览器去写不同的CSS，让它能够同时兼容不同的浏览器，能在不同的浏览器中实现页面效果的一致性。针对不同的浏览器写不同的CSS Code的过程，就叫CSS hack。 我们写 ie 和 firefox 的css hack，可以认为，兼容火狐浏览器，一般情况下都会兼容chorme、oprea、safari浏览器。firefox和它们还是有细微的区别，如果 html和css结构规范，一般情况下还是可以认为：兼容firefox，即兼容了chrome、oprea、safari，因为它们是同一个阵型的。另 外，firefox3.5以上的版本，和ie8兼容性接近。

- 尽可能的减少对Hack的使用和依赖，如果在项目中对Hack的使用太多太复杂，对项目的维护将是一个巨大的挑战；
- 使用其它的解决方案代替Hack思路；
- 如果非Hack不可，选择稳定且常用并易于理解的。

```css
.test {
    color: #000;       /* For all */
    color: #111\9;     /* For all IE */
    color: #222\0;     /* For IE8 and later, Opera without Webkit */
    color: #333\9\0;   /* For IE8 and later */
    color: #444\0/;    /* For IE8 and later */
    color: #666;      /* For IE7 and earlier */
    color: #777;      /* For IE6 and earlier */
}
```

> 注：严谨且长期的项目，针对IE可以使用条件注释作为预留Hack或者在当前使用

#### IE条件注释语法：
```html
<!--[if <keywords>? IE <version>?]>
<link rel="stylesheet" href="*.css" />
<![endif]-->
```
语法说明：
- keywords
  - if条件共包含6种选择方式：是否、大于、大于或等于、小于、小于或等于、非指定版本
  - 是否：指定是否IE或IE某个版本。关键字：空
  - 大于：选择大于指定版本的IE版本。关键字：gt（greater than）
  - 大于或等于：选择大于或等于指定版本的IE版本。关键字：gte（greater than or equal）
  - 小于：选择小于指定版本的IE版本。关键字：lt（less than）
  - 小于或等于：选择小于或等于指定版本的IE版本。关键字：lte（less than or equal）
  - 非指定版本：选择除指定版本外的所有IE版本。关键字：!
- version
  目前的常用IE版本为6.0及以上，推荐酌情忽略低版本，把精力花在为使用高级浏览器的用户提供更好的体验上，另从IE10开始已无此特性

### 4.动画与转换
使用 transition 时应指定 transition-property。
eg:
```css
/* good */
.box {
  transition: color 1s, border-color 1s;
}
/* bad */
.box {
  transition: all 1s;
}
```

在浏览器能高效实现的属性上添加过渡和动画。在可能的情况下应选择这样四种变换：
- transform: translate(npx, npx);
- transform: scale(n);
- transform: rotate(ndeg);
- opacity: 0..1;

典型的，可以使用 translate 来代替 left 作为动画属性。

> 示例：一般在 Chrome 中，3D或透视变换（perspective transform）CSS属性和对 opacity 进行 CSS 动画会创建新的图层，在硬件加速渲染通道的优化下，GPU 完成 3D 变形等操作后，将图层进行复合操作（Compesite Layers），从而避免触发浏览器大面积 重绘 和 重排。

```css
/* good */
.box {
  transition: transform 1s;
}
.box:hover {
  transform: translate(20px); /* move right for 20px */
}
/* bad */
.box {
  left: 0;
  transition: left 1s;
}
.box:hover {
  left: 20px; /* move right for 20px */
}
```

动画的基本概念：
- 帧：在动画过程中，每一幅静止画面即为一“帧”;
- 帧率：即每秒钟播放的静止画面的数量，单位是fps(Frame per second);
- 帧时长：即每一幅静止画面的停留时间，单位一般是ms(毫秒);
- 跳帧(掉帧/丢帧)：在帧率固定的动画中，某一帧的时长远高于平均帧时长，导致其后续数帧被挤压而丢失的现象。
- 如果使用基于 javaScript 的动画，尽量使用 requestAnimationFrame. 避免使用 setTimeout, setInterval.
- 避免通过类似 jQuery animate()-style 改变每帧的样式，使用 CSS 声明动画会得到更好的浏览器优化。
- 使用 translate 取代 absolute 定位就会得到更好的 fps，动画会更顺滑。

> 一般浏览器的渲染刷新频率是60 fps，所以在网页当中，帧率如果达到 50-60 fps 的动画将会相当流畅，让人感到舒适。

### 5.性能优化
#### 慎重选择高消耗的样式
高消耗属性在绘制前需要浏览器进行大量计算：
- box-shadows
- border-radius
- transparency
- transforms
- CSS filters（性能杀手）

#### 正确使用 Display 的属性
Display 属性会影响页面的渲染，请合理使用。
- display: inline后不应该再使用 width、height、margin、padding 以及 float；
- display: inline-block 后不应该再使用 float；
- display: block 后不应该再使用 vertical-align；
- display: table-* 后不应该再使用 margin 或者 float；

#### 提升 CSS 选择器性能
CSS 选择器是从右到左进行规则匹配。只要当前选择符的左边还有其他选择符，样式系统就会继续向左移动，直到找到和规则匹配的选择符，或者因为不匹配而退出。最右边选择符称之为关键选择器。
CSS 选择器的执行效率从高到低做一个排序：
1. id选择器（#myid）
2. 类选择器（.myclassname）
3. 标签选择器（div,h1,p）
4. 相邻选择器（h1+p）
5. 子选择器（ul < li）
6. 后代选择器（li a）
7. 通配符选择器（*）
8. 属性选择器（a[rel="external"]）
9. 伪类选择器（a:hover, li:nth-child）

#### 避免使用通用选择器
```css
/* Not recommended */
.content * {color: red;}
```
浏览器匹配文档中所有的元素后分别向上逐级匹配 class 为 content 的元素，直到文档的根节点。因此其匹配开销是非常大的，所以应避免使用关键选择器是通配选择器的情况。

#### 避免使用标签或 class 选择器限制 id 选择器
```css
/* Not recommended */
button#backButton {…}
/* Recommended */
#newMenuIcon {…}
```

#### 避免使用标签限制 class 选择器
```css
/* Not recommended */
treecell.indented {…}
/* Recommended */
.treecell-indented {…}
/* Much to recommended */
.hierarchy-deep {…}
```

#### 避免使用多层标签选择器。
使用 class 选择器替换，减少css查找
```css
/* Not recommended */
treeitem[mailfolder="true"] > treerow > treecell {…}
/* Recommended */
.treecell-mailfolder {…}
```

#### 避免使用子选择器
```css
/* Not recommended */
treehead treerow treecell {…}
/* Recommended */
treehead > treerow > treecell {…}
/* Much to recommended */
.treecell-header {…}
```

#### 使用继承
```css
/* Not recommended */
#bookmarkMenuItem > .menu-left { list-style-image: url(blah) }
/* Recommended */
#bookmarkMenuItem { list-style-image: url(blah) }
```

## JavaScript规范
### 1.通用约定
**注释原则**
- As short as possible（如无必要，勿增注释）：尽量提高代码本身的清晰性、可读性。
- As long as necessary（如有必要，尽量详尽）：合理的注释、空行排版等，可以让代码更易阅读、更具美感。

#### 命名
**变量** 使用驼峰命名。
```javascript
let loadingModules = {};
```
**私有属性、变量**和**方法**以下划线 _ 开头。
```javascript
let _that = this;
```
**常量** 使用全部字母大写且用const。
```javascript
const MAXCOUNT = 10;
```

#### 二元和三元操作符
操作符始终写在前一行, 以免分号的隐式插入产生预想不到的问题。
```javascript
let x = a ? b : c;
let y = a ?
    longButSimpleOperandB : longButSimpleOperandC;
let z = a ?
        moreComplicatedB :
        moreComplicatedC;
```
操作符也是如此：
```javascript
let x = foo.bar().
    doSomething().
    doSomethingElse();
```

#### 条件(三元)操作符 ( ? : )
三元操作符用于替代 if 条件判断语句。
```javascript
// Not recommended
if (val != 0) {
  return foo();
} else {
  return bar();
}
// Recommended
return val ? foo() : bar();
```

#### 具备强类型的设计
**解释：**
- 如果一个属性被设计为 boolean 类型，则不要使用 1 或 0 作为其值。对于标识性的属性，如对代码体积有严格要求，可以从一开始就设计为 number 类型且将 0 作为否定值。
- 从 DOM 中取出的值通常为 string 类型，如果有对象或函数的接收类型为 number 类型，提前作好转换，而不是期望对象、函数可以处理多类型的值。

### 2.函数设计
> 函数设计基本原则：低耦合，高内聚。（假如一个程序有50个函数；一旦你修改其中一个函数，其他49个函数都需要做修改，这就是高耦合的后果。）

#### 函数长度
**建议**一个函数的长度控制在 50 行以内。 将过多的逻辑单元混在一个大函数中，易导致难以维护。一个清晰易懂的函数应该完成单一的逻辑单元。复杂的操作应进一步抽取，通过函数的调用来体现流程。 特定算法等不可分割的逻辑允许例外。

例：
![](https://api2.mubu.com/v3/document_image/79726943-7739-4e24-82ed-cfcaacbf90aa-3807603.jpg)

#### 参数设计
**建议**一个函数的参数控制在 6 个以内。
解释：

除去不定长参数以外，函数具备不同逻辑意义的参数建议控制在 6 个以内，过多参数会导致维护难度增大。
某些情况下，如使用 AMD Loader 的 require 加载多个模块时，其 callback可能会存在较多参数，因此对函数参数的个数不做强制限制。

**建议**通过 options 参数传递非数据输入型参数。 解释：

有些函数的参数并不是作为算法的输入，而是对算法的某些分支条件判断之用，此类参数建议通过一个 options 参数传递。

如下函数：
![](https://api2.mubu.com/v3/document_image/70643c97-3bbb-40f6-ae34-4b8bb1ec46eb-3807603.jpg)

可以转换为下面的函数：
![](https://api2.mubu.com/v3/document_image/3ca131b4-6b3c-4128-b5a5-cf13872a0f7d-3807603.jpg)

这种模式有几个显著的优势：
- boolean 型的配置项具备名称，从调用的代码上更易理解其表达的逻辑意义。
- 当配置项有增长时，无需无休止地增加参数个数，不会出现 removeElement(element, true, false, false, 3) 这样难以理解的调用代码。
- 当部分配置参数可选时，多个参数的形式非常难处理重载逻辑，而使用一个 options 对象只需判断属性是否存在，实现得以简化。

#### 箭头函数
当你必须使用函数表达式（或传递一个匿名函数）时，使用箭头函数符号。
- 为什么？因为箭头函数创造了新的一个 this 执行环境，通常情况下都能满足你的需求，而且这样的写法更为简洁。
- 为什么不？如果你有一个相当复杂的函数，你或许可以把逻辑部分转移到一个函数声明上。

eg:
```javascript
// Not recommended
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});
// Recommended
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

如果一个函数适合用一行写出并且只有一个参数，那就把花括号、圆括号和 return 都省略掉。如果不是，那就不要省略。
- 为什么？语法糖。在链式调用中可读性很高。 为什么不？当你打算回传一个对象的时候。

eg:
```javascript
// Recommended
[1, 2, 3].map(x => x * x);
// Recommended
[1, 2, 3].reduce((total, n) => {
  return total + n;
}, 0);
```

#### 函数和变量
同一个函数内部，局部变量的声明必须置于顶端 因为即使放到中间，js解析器也会提升至顶部（hosting）
eg:
```javascript
// Recommended
var clear = function(el) {
    var id = el.id,
        name = el.getAttribute("data-name");
    .........
    return true;
}

// Not recommended
var clear = function(el) {
    var id = el.id;
    ......
    var name = el.getAttribute("data-name");
    .........
    return true;
}
```

#### 块内函数必须用局部变量声明
```javascript
// Not recommended
var call = function(name) {
    if (name == "hotel") {
        function foo() {
            console.log("hotel foo");
        }
    }

    foo && foo();
}

// Recommended
var call = function(name) {
    var foo;

    if (name == "hotel") {
        foo = function() {
            console.log("hotel foo");
        }
    }

    foo && foo();
}
```

> 引起的bug：第一种写法foo的声明被提前了; 调用call时：第一种写法会调用foo函数，第二种写法不会调用foo函数

#### 空函数
**建议**对于性能有高要求的场合，建议存在一个空函数的常量，供多处使用共享。
eg:
![](https://api2.mubu.com/v3/document_image/5293e8a1-ab11-4a9f-99c7-07aded6d933d-3807603.jpg)

### 3.性能优化
#### 避免不必要的 DOM 操作
浏览器遍历 DOM 元素的代价是昂贵的。最简单优化 DOM 树查询的方案是，当一个元素出现多次时，将它保存在一个变量中，就避免多次查询 DOM 树了。

#### 缓存数组长度
循环无疑是和 JavaScript 性能非常相关的一部分。通过存储数组的长度，可以有效避免每次循环重新计算。

> 注: 虽然现代浏览器引擎会自动优化这个过程，但是不要忘记还有旧的浏览器。

```javascript
var arr = new Array(1000),
    len, i;// Recommended - size is calculated only 1 time and then stored
for (i = 0, len = arr.length; i < len; i++) {

}
// Not recommended - size needs to be recalculated 1000 times
for (i = 0; i < arr.length; i++) {

}
```

#### 优化 JavaScript 的特征
- 编写可维护的代码
- 单变量模式
- Hoisting：把所有变量声明统一放到函数的起始位置 （在后部声明的变量也会被JS视为在头部定义，由此会产生问题）
- 不要扩充内置原型（虽然给Object(), Function()之类的内置原型增加属性和方法很巧妙，但是会破坏可维护性）
- 不要用隐含的类型转换
- 不要用 eval()
- 用 parseInt() 进行数字转换
- （规范）左大括号的位置
- 构造器首字母大写
- 写注释
- 不要用 void
- 不要用 with 语句
- 不要用 continue 语句
- 尽量不要用位运算

#### 优化 JavaScript 执行
在页面加载的时候，通常会有很多脚本等待执行，但你可以设定优先级。下面的顺序就是基于用户反馈设定的优先级：
1. 改变页面视觉习性的脚本绝对要首先执行。这包括任何的字体调整、盒子布局、或IE6 pngfix。
2. 页面内容初始化
3. 页面标题、顶部导航和页脚的初始化
4. 绑定事件处理器
5. 网页流量分析、Doubleclick，以及其他第三方脚本

#### 异步加载第三方内容
当你无法保证嵌入第三方内容比如 Youtube 视频或者一个 like/tweet 按钮可以正常工作的时候，你需要考虑用异步加载这些代码，避免阻塞整个页面加载。
eg:
![](https://api2.mubu.com/v3/document_image/3faebd97-e616-4e9d-b3f0-40a1f6b2a2a3-3807603.jpg)


### 5.调试
浏览器的怪异性会不可避免地产生一些问题。有几个工具可以帮助优化代码的健全性和加载速度。推荐先在Google Chrome和Firefox做开发，然后用Safari上和Opera，最后针对IE用条件判断做一些额外的微调。下面列出的是一些有帮助的调试器和速度分析器：
- Google Chrome: Developer Tools
- Firefox: Firebug, Page Speed, YSlow
- Safari: Web Inspector
- Opera: Dragonfly
- Internet Explorer 6-7: Developer Toolbar
- Internet Explorer 8-10: Developer Tools

## vue
**开发规范请看**[vue开发规范]()

## react
**开发规范请看**[react开发规范]()

## angular
**开发规范请看**[angular开发规范]()
