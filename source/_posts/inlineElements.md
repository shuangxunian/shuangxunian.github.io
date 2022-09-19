---
title: 行内元素、块级元素和行内块级元素
excerpt: 关于css行内元素、块级元素和行内块级元素概念
categories:
- 技术文章
tags:
- html
- css
- 面试
---

## 行内元素和块级元素

在html中元素主要分为行内元素和块级元素。

**行内元素**
- 不独占一行
- 书写完成后不会自动换行
- 没有宽和高

常见的行级标签：a， span,  strong,  u,  em
如：
```html
<p>This <span style="background-color:red;">span</span> is an inline element; </p>
```

![图0](https://api2.mubu.com/v3/document_image/8ad6cfc7-a685-4bcb-a9af-e03613e3ab6b-3807603.jpg)

**块级元素**
- 独占一行
- 书写完会自动换行
- 宽和高可以修改

常见的块级标签：div,  p,  h1-h6,  ul,  li,  dl,  dt,  dd
如：
```html
<p style="background-color:red;">This paragraph is a block-level element.</p>
```

![图1](https://api2.mubu.com/v3/document_image/ad18f9c2-90a4-4444-b20f-eb1d64769ca9-3807603.jpg)

## 行内块级元素

行内块状元素综合了行内元素和块状元素的特性，但是各有取舍。因此行内块状元素在日常的使用中，由于其特性，使用的次数也比较多。特点如下：
- 不自动换行
- 能够识别宽高
- 默认排列方式为从左到右

常见的行内块标签：img,  input,   textarea

## 常见的元素

### 常见的行内元素

| 元素        | 描述                             |
| ---------- | --------------------------------- |
| &lt;a&gt;        | 标签可定义锚                      |
| &lt;abbr&gt;     | 表示一个缩写形式                  |
| &lt;acronym&gt;  | 定义只取首字母缩写                |
| &lt;b&gt;        | 字体加粗                          |
| &lt;bdo&gt;      | 可覆盖默认的文本方向              |
| &lt;big&gt;      | 大号字体加粗                      |
| &lt;br&gt;       | 换行                              |
| &lt;cite&gt;     | 引用进行定义                      |
| &lt;code&gt;     | 定义计算机代码文本                |
| &lt;dfn&gt;      | 定义一个定义项目                  |
| &lt;em&gt;       | 定义为强调的内容                  |
| &lt;i&gt;        | 斜体文本效果                      |
| &lt;img&gt;      | 向网页中嵌入一幅图像              |
| &lt;input&gt;    | 输入框                            |
| &lt;kbd&gt;      | 定义键盘文本                      |
| &lt;label&gt;    | 标签为 input 元素定义标注（标记） |
| &lt;q&gt;        | 定义短的引用                      |
| &lt;samp&gt;     | 定义样本文本                      |
| &lt;select&gt;   | 创建单选或多选菜单                |
| &lt;small&gt;    | 呈现小号字体效果                  |
| &lt;span&gt;     | 组合文档中的行内元素              |
| &lt;strong&gt;   | 语气更强的强调的内容              |
| &lt;sub&gt;      | 定义下标文本                      |
| &lt;sup&gt;      | 定义上标文本                      |
| &lt;textarea&gt; | 多行的文本输入控件                |
| &lt;tt&gt;       | 打字机或者等宽的文本效果          |
| &lt;var&gt;      | 定义变量                          |

### 常见的块级元素

| 元素        | 描述                                                 |
| ---------- | ------------------------------------------------------ |
| &lt;address&gt;  | 定义地址                                               |
| &lt;caption&gt;  | 定义表格标题                                           |
| &lt;dd&gt;       | 定义列表中定义条目                                     |
| &lt;div&gt;      | 定义文档中的分区或节                                   |
| &lt;dl&gt;       | 定义列表                                               |
| &lt;dt&gt;       | 定义列表中的项目                                       |
| &lt;fieldset&gt; | 定义一个框架集                                         |
| &lt;form&gt;     | 创建 HTML 表单                                         |
| &lt;h1&gt;       | 定义最大的标题                                         |
| &lt;h2&gt;       | 定义副标题                                             |
| &lt;h3&gt;       | 定义标题                                               |
| &lt;h4&gt;       | 定义标题                                               |
| &lt;h5&gt;       | 定义标题                                               |
| &lt;h6&gt;       | 定义最小的标题                                         |
| &lt;hr&gt;       | 创建一条水平线                                         |
| &lt;legend&gt;   | 元素为 fieldset 元素定义标题                           |
| &lt;li&gt;       | 标签定义列表项目                                       |
| &lt;noframes&gt; | 为那些不支持框架的浏览器显示文本，于 frameset 元素内部 |
| &lt;noscript&gt; | 定义在脚本未被执行时的替代内容                         |
| &lt;ol&gt;       | 定义有序列表                                           |
| &lt;ul&gt;       | 定义无序列表                                           |
| &lt;p&gt;        | 标签定义段落                                           |
| &lt;pre&gt;      | 定义预格式化的文本                                     |
| &lt;table&gt;    | 标签定义 HTML 表格                                     |
| &lt;tbody&gt;    | 标签表格主体（正文）                                   |
| &lt;td&gt;       | 表格中的标准单元格                                     |
| &lt;tfoot&gt;    | 定义表格的页脚（脚注或表注）                           |
| &lt;th&gt;       | 定义表头单元格                                         |
| &lt;thead&gt;    | 标签定义表格的表头                                     |
| &lt;tr&gt;       | 定义表格中的行                                         |

### 可变元素

可变元素为根据上下文语境决定该元素为块元素或者内联元素

| 元素      | 描述                                        |
| -------- | -------------------------------------------- |
| &lt;button&gt; | 按钮                                         |
| &lt;del&gt;    | 定义文档中已被删除的文本                     |
| &lt;iframe&gt; | 创建包含另外一个文档的内联框架（即行内框架） |
| &lt;ins&gt;    | 标签定义已经被插入文档中的文本               |
| &lt;map&gt;    | 客户端图像映射（即热区）                     |
| &lt;object&gt; | object对象                                   |
| &lt;script&gt; | 客户端脚本                                   |

### 行内块级元素

| 元素          | 描述                                                |
| ------------ | --------------------------------------------------- |
| none         |                                   此元素不会被显示。 |
| block        |     此元素将显示为块级元素，此元素前后会带有换行符。 |
| inline       | 默认。此元素会被显示为内联元素，元素前后没有换行符。 |
| inline-block |                      行内块元素。（CSS2.1 新增的值） |
| list-item    |                               此元素会作为列表显示。 |

## 转换方法

这三者是可以互相转换的，使用display属性能够将三者任意转换：

- display:inline;转换为行内元素

- display:block;转换为块状元素

- display:inline-block;转换为行内块状元素

## 参考链接

[HTML行内元素、块状元素、行内块状元素的区别](https://blog.csdn.net/zhanglir333/article/details/79178370)
[行内元素和块级元素的具体区别是什么？inline-block是什么？（面试题目）](https://www.cnblogs.com/iceflorence/p/6626187.html)
[说说行内元素和块级元素](https://www.jianshu.com/p/d69878549d92)
[探究行内元素和块级元素](https://juejin.im/post/6844903840420986888#heading-0)