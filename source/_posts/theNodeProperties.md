---
title: Node属性
excerpt: 关于Node属性的文章
categories:
- 技术文章
tags:
- DOM
- js
---

## Node.prototype.nodeType
nodeType属性返回一个整数值，表示节点的类型。

```javascript
document.nodeType // 9
```

上面代码中，文档节点的类型值为9。

Node 对象定义了几个常量，对应这些类型值。

```javascript
document.nodeType === Node.DOCUMENT_NODE // true
```

上面代码中，文档节点的nodeType属性等于常量Node.DOCUMENT_NODE。

不同节点的nodeType属性值和对应的常量如下：
- 文档节点（document）：9，对应常量Node.DOCUMENT_NODE
- 元素节点（element）：1，对应常量Node.ELEMENT_NODE
- 属性节点（attr）：2，对应常量Node.ATTRIBUTE_NODE
- 文本节点（text）：3，对应常量Node.TEXT_NODE
- 文档片断节点（DocumentFragment）：11，对应常量Node.DOCUMENT_FRAGMENT_NODE
- 文档类型节点（DocumentType）：10，对应常量Node.DOCUMENT_TYPE_NODE
- 注释节点（Comment）：8，对应常量Node.COMMENT_NODE

确定节点类型时，使用nodeType属性是常用方法。
```javascript
var node = document.documentElement.firstChild;
if (node.nodeType === Node.ELEMENT_NODE) {
    console.log('该节点是元素节点');
}
```

## Node.prototype.nodeName
nodeName属性返回节点的名称。

```javascript
// HTML 代码如下
// <div id="d1">hello world</div>
var div = document.getElementById('d1');
div.nodeName // "DIV"
```

上面代码中，元素节点&lt;div&gt;的nodeName属性就是大写的标签名DIV。

不同节点的nodeName属性值如下：
- 文档节点（document）：#document
- 元素节点（element）：大写的标签名
- 属性节点（attr）：属性的名称
- 文本节点（text）：#text
- 文档片断节点（DocumentFragment）：#document-fragment
- 文档类型节点（DocumentType）：文档的类型
- 注释节点（Comment）：#comment

## Node.prototype.nodeValue
nodeValue属性返回一个字符串，表示当前节点本身的文本值，该属性可读写。

只有文本节点（text）、注释节点（comment）和属性节点（attr）有文本值，因此这三类节点的nodeValue可以返回结果，其他类型的节点一律返回null。同样的，也只有这三类节点可以设置nodeValue属性的值，其他类型的节点设置无效。

```javascript
// HTML 代码如下
// <div id="d1">hello world</div>
var div = document.getElementById('d1');
div.nodeValue // null
div.firstChild.nodeValue // "hello world"
```

上面代码中，div是元素节点，nodeValue属性返回null。div.firstChild是文本节点，所以可以返回文本值。

## Node.prototype.textContent
textContent属性返回当前节点和它的所有后代节点的文本内容。

```javascript
// HTML 代码为
// <div id="divA">This is <span>some</span> text</div>
document.getElementById('divA').textContent
// This is some text
```

textContent属性自动忽略当前节点内部的 HTML 标签，返回所有文本内容。

该属性是可读写的，设置该属性的值，会用一个新的文本节点，替换所有原来的子节点。它还有一个好处，就是自动对 HTML 标签转义。这很适合用于用户提供的内容。

```javascript
document.getElementById('foo').textContent = '<p>GoodBye!</p>';
```

上面代码在插入文本时，会将&lt;p&gt;标签解释为文本，而不会当作标签处理。

对于文本节点（text）、注释节点（comment）和属性节点（attr），textContent属性的值与nodeValue属性相同。对于其他类型的节点，该属性会将每个子节点（不包括注释节点）的内容连接在一起返回。如果一个节点没有子节点，则返回空字符串。

文档节点（document）和文档类型节点（doctype）的textContent属性为null。如果要读取整个文档的内容，可以使用document.documentElement.textContent。

## Node.prototype.baseURI
baseURI属性返回一个字符串，表示当前网页的绝对路径。浏览器根据这个属性，计算网页上的相对路径的 URL。该属性为只读。

```javascript
// 当前网页的网址为
// http://www.example.com/index.html
document.baseURI
// "http://www.example.com/index.html"
```

如果无法读到网页的 URL，baseURI属性返回null。

该属性的值一般由当前网址的 URL（即window.location属性）决定，但是可以使用 HTML 的&lt;base&gt;标签，改变该属性的值。

```html
<base href="http://www.example.com/page.html">
```

设置了以后，baseURI属性就返回&lt;base&gt;标签设置的值。

## Node.prototype.ownerDocument
Node.ownerDocument属性返回当前节点所在的顶层文档对象，即document对象。

```javascript
var d = p.ownerDocument;
d === document // true
```

document对象本身的ownerDocument属性，返回null。

## Node.prototype.nextSibling
Node.nextSibling属性返回紧跟在当前节点后面的第一个同级节点。如果当前节点后面没有同级节点，则返回null。

```javascript
// HTML 代码如下
// <div id="d1">hello</div>
// <div id="d2">world</div>
var d1 = document.getElementById('d1');
var d2 = document.getElementById('d2');
d1.nextSibling === d2 // true
```

上面代码中，d1.nextSibling就是紧跟在d1后面的同级节点d2。

注意，该属性还包括文本节点和注释节点（&lt;!-- comment --&gt;）。因此如果当前节点后面有空格，该属性会返回一个文本节点，内容为空格。

nextSibling属性可以用来遍历所有子节点。

```javascript
var el = document.getElementById('div1').firstChild;
while (el !== null) {
  console.log(el.nodeName);
  el = el.nextSibling;
}
```

上面代码遍历div1节点的所有子节点。

## Node.prototype.previousSibling
previousSibling属性返回当前节点前面的、距离最近的一个同级节点。如果当前节点前面没有同级节点，则返回null。

```javascript
// HTML 代码如下
// <div id="d1">hello</div><div id="d2">world</div>
var d1 = document.getElementById('d1');
var d2 = document.getElementById('d2');

d2.previousSibling === d1 // true
```

上面代码中，d2.previousSibling就是d2前面的同级节点d1。

注意，该属性还包括文本节点和注释节点。因此如果当前节点前面有空格，该属性会返回一个文本节点，内容为空格。

## Node.prototype.parentNode
parentNode属性返回当前节点的父节点。对于一个节点来说，它的父节点只可能是三种类型：元素节点（element）、文档节点（document）和文档片段节点（documentfragment）。

```javascript
if (node.parentNode) {
  node.parentNode.removeChild(node);
}
```

上面代码中，通过node.parentNode属性将node节点从文档里面移除。

文档节点（document）和文档片段节点（documentfragment）的父节点都是null。另外，对于那些生成后还没插入 DOM 树的节点，父节点也是null。

## Node.prototype.parentElement
parentElement属性返回当前节点的父元素节点。如果当前节点没有父节点，或者父节点类型不是元素节点，则返回null。

```javascript
if (node.parentElement) {
  node.parentElement.style.color = 'red';
}
```

上面代码中，父元素节点的样式设定了红色。

由于父节点只可能是三种类型：元素节点、文档节点（document）和文档片段节点（documentfragment）。parentElement属性相当于把后两种父节点都排除了。

## Node.prototype.firstChild，Node.prototype.lastChild
firstChild属性返回当前节点的第一个子节点，如果当前节点没有子节点，则返回null。

```javascript
// HTML 代码如下
// <p id="p1"><span>First span</span></p>
var p1 = document.getElementById('p1');
p1.firstChild.nodeName // "SPAN"
```

上面代码中，p元素的第一个子节点是span元素。

注意，firstChild返回的除了元素节点，还可能是文本节点或注释节点。

```javascript
// HTML 代码如下
// <p id="p1">
//   <span>First span</span>
//  </p>
var p1 = document.getElementById('p1');
p1.firstChild.nodeName // "#text"
```

上面代码中，p元素与span元素之间有空白字符，这导致firstChild返回的是文本节点。

lastChild属性返回当前节点的最后一个子节点，如果当前节点没有子节点，则返回null。用法与firstChild属性相同。

## Node.prototype.childNodes
childNodes属性返回一个类似数组的对象（NodeList集合），成员包括当前节点的所有子节点。

```javascript
var children = document.querySelector('ul').childNodes;
```

上面代码中，children就是ul元素的所有子节点。

使用该属性，可以遍历某个节点的所有子节点。

```javascript
var div = document.getElementById('div1');
var children = div.childNodes;

for (var i = 0; i < children.length; i++) {
  // ...
}
```

文档节点（document）就有两个子节点：文档类型节点（docType）和 HTML 根元素节点。

```javascript
var children = document.childNodes;
for (var i = 0; i < children.length; i++) {
  console.log(children[i].nodeType);
}
// 10
// 1
```

上面代码中，文档节点的第一个子节点的类型是10（即文档类型节点），第二个子节点的类型是1（即元素节点）。

注意，除了元素节点，childNodes属性的返回值还包括文本节点和注释节点。如果当前节点不包括任何子节点，则返回一个空的NodeList集合。由于NodeList对象是一个动态集合，一旦子节点发生变化，立刻会反映在返回结果之中。

## Node.prototype.isConnected
isConnected属性返回一个布尔值，表示当前节点是否在文档之中。

```javascript
var test = document.createElement('p');
test.isConnected // false

document.body.appendChild(test);
test.isConnected // true
```

上面代码中，test节点是脚本生成的节点，没有插入文档之前，isConnected属性返回false，插入之后返回true。

## 参考链接
[Node 接口](https://wangdoc.com/javascript/dom/node.html)