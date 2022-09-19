---
title: Node方法
excerpt: 关于Node方法的文章
categories:
- 技术文章
tags:
- DOM
- js
---

## Node.prototype.appendChild()
appendChild()方法接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点。该方法的返回值就是插入文档的子节点。

```javascript
var p = document.createElement('p');
document.body.appendChild(p);
```

上面代码新建一个<p>节点，将其插入document.body的尾部。

如果参数节点是 DOM 已经存在的节点，appendChild()方法会将其从原来的位置，移动到新位置。

```javascript
var div = document.getElementById('myDiv');
document.body.appendChild(div);
```

上面代码中，插入的是一个已经存在的节点myDiv，结果就是该节点会从原来的位置，移动到document.body的尾部。

如果appendChild()方法的参数是DocumentFragment节点，那么插入的是DocumentFragment的所有子节点，而不是DocumentFragment节点本身。返回值是一个空的DocumentFragment节点。

## Node.prototype.hasChildNodes()
hasChildNodes方法返回一个布尔值，表示当前节点是否有子节点。

```javascript
var foo = document.getElementById('foo');

if (foo.hasChildNodes()) {
  foo.removeChild(foo.childNodes[0]);
}
```

上面代码表示，如果foo节点有子节点，就移除第一个子节点。

注意，子节点包括所有类型的节点，并不仅仅是元素节点。哪怕节点只包含一个空格，hasChildNodes方法也会返回true。

判断一个节点有没有子节点，有许多种方法，下面是其中的三种。

- node.hasChildNodes()
- node.firstChild !== null
- node.childNodes && node.childNodes.length > 0

hasChildNodes方法结合firstChild属性和nextSibling属性，可以遍历当前节点的所有后代节点。

```javascript
function DOMComb(parent, callback) {
  if (parent.hasChildNodes()) {
    for (var node = parent.firstChild; node; node = node.nextSibling) {
      DOMComb(node, callback);
    }
  }
  callback(parent);
}

// 用法
DOMComb(document.body, console.log)
```

上面代码中，DOMComb函数的第一个参数是某个指定的节点，第二个参数是回调函数。这个回调函数会依次作用于指定节点，以及指定节点的所有后代节点。

## Node.prototype.cloneNode()
cloneNode方法用于克隆一个节点。它接受一个布尔值作为参数，表示是否同时克隆子节点。它的返回值是一个克隆出来的新节点。

```javascript
var cloneUL = document.querySelector('ul').cloneNode(true);
```

该方法有一些使用注意点。

1. 克隆一个节点，会拷贝该节点的所有属性，但是会丧失addEventListener方法和on-属性（即node.onclick = fn），添加在这个节点上的事件回调函数。
2. 该方法返回的节点不在文档之中，即没有任何父节点，必须使用诸如Node.appendChild这样的方法添加到文档之中。
3. 克隆一个节点之后，DOM 有可能出现两个有相同id属性（即id="xxx"）的网页元素，这时应该修改其中一个元素的id属性。如果原节点有name属性，可能也需要修改。

## Node.prototype.insertBefore()
insertBefore方法用于将某个节点插入父节点内部的指定位置。

```javascript
var insertedNode = parentNode.insertBefore(newNode, referenceNode);
```

insertBefore方法接受两个参数，第一个参数是所要插入的节点newNode，第二个参数是父节点parentNode内部的一个子节点referenceNode。newNode将插在referenceNode这个子节点的前面。返回值是插入的新节点newNode。

```javascript
var p = document.createElement('p');
document.body.insertBefore(p, document.body.firstChild);
```

上面代码中，新建一个<p>节点，插在document.body.firstChild的前面，也就是成为document.body的第一个子节点。

如果insertBefore方法的第二个参数为null，则新节点将插在当前节点内部的最后位置，即变成最后一个子节点。

```javascript
var p = document.createElement('p');
document.body.insertBefore(p, null);
```

上面代码中，p将成为document.body的最后一个子节点。这也说明insertBefore的第二个参数不能省略。

注意，如果所要插入的节点是当前 DOM 现有的节点，则该节点将从原有的位置移除，插入新的位置。

由于不存在insertAfter方法，如果新节点要插在父节点的某个子节点后面，可以用insertBefore方法结合nextSibling属性模拟。

```javascript
parent.insertBefore(s1, s2.nextSibling);
```

上面代码中，parent是父节点，s1是一个全新的节点，s2是可以将s1节点，插在s2节点的后面。如果s2是当前节点的最后一个子节点，则s2.nextSibling返回null，这时s1节点会插在当前节点的最后，变成当前节点的最后一个子节点，等于紧跟在s2的后面。

如果要插入的节点是DocumentFragment类型，那么插入的将是DocumentFragment的所有子节点，而不是DocumentFragment节点本身。返回值将是一个空的DocumentFragment节点。

## Node.prototype.removeChild()
removeChild方法接受一个子节点作为参数，用于从当前节点移除该子节点。返回值是移除的子节点。

```javascript
var divA = document.getElementById('A');
divA.parentNode.removeChild(divA);
```

上面代码移除了divA节点。注意，这个方法是在divA的父节点上调用的，不是在divA上调用的。

下面是如何移除当前节点的所有子节点。

```javascript
var element = document.getElementById('top');
while (element.firstChild) {
  element.removeChild(element.firstChild);
}
```

被移除的节点依然存在于内存之中，但不再是 DOM 的一部分。所以，一个节点移除以后，依然可以使用它，比如插入到另一个节点下面。

如果参数节点不是当前节点的子节点，removeChild方法将报错。

## Node.prototype.replaceChild()
replaceChild方法用于将一个新的节点，替换当前节点的某一个子节点。

```javascript
var replacedNode = parentNode.replaceChild(newChild, oldChild);
```

上面代码中，replaceChild方法接受两个参数，第一个参数newChild是用来替换的新节点，第二个参数oldChild是将要替换走的子节点。返回值是替换走的那个节点oldChild。

```javascript
var divA = document.getElementById('divA');
var newSpan = document.createElement('span');
newSpan.textContent = 'Hello World!';
divA.parentNode.replaceChild(newSpan, divA);
```

上面代码是如何将指定节点divA替换走。

## Node.prototype.contains()
contains方法返回一个布尔值，表示参数节点是否满足以下三个条件之一。

- 参数节点为当前节点。
- 参数节点为当前节点的子节点。
- 参数节点为当前节点的后代节点。

```javascript
document.body.contains(node)
```

上面代码检查参数节点node，是否包含在当前文档之中。

注意，当前节点传入contains方法，返回true。

```javascript
nodeA.contains(nodeA) // true
```

## Node.prototype.compareDocumentPosition()
compareDocumentPosition方法的用法，与contains方法完全一致，返回一个六个比特位的二进制值，表示参数节点与当前节点的关系。

| 二进制值 | 十进制值 | 含义                                               |
| :------- | :------- | :------------------------------------------------- |
| 000000   | 0        | 两个节点相同                                       |
| 000001   | 1        | 两个节点不在同一个文档（即有一个节点不在当前文档） |
| 000010   | 2        | 参数节点在当前节点的前面                           |
| 000100   | 4        | 参数节点在当前节点的后面                           |
| 001000   | 8        | 参数节点包含当前节点                               |
| 010000   | 16       | 当前节点包含参数节点                               |
| 100000   | 32       | 浏览器内部使用                                     |

```javascript
// HTML 代码如下
// <div id="mydiv">
//   <form><input id="test" /></form>
// </div>

var div = document.getElementById('mydiv');
var input = document.getElementById('test');

div.compareDocumentPosition(input) // 20
input.compareDocumentPosition(div) // 10
```

上面代码中，节点div包含节点input（二进制010000），而且节点input在节点div的后面（二进制000100），所以第一个compareDocumentPosition方法返回20（二进制010100，即010000 + 000100），第二个compareDocumentPosition方法返回10（二进制001010）。

由于compareDocumentPosition返回值的含义，定义在每一个比特位上，所以如果要检查某一种特定的含义，就需要使用比特位运算符。

```javascript
var head = document.head;
var body = document.body;
if (head.compareDocumentPosition(body) & 4) {
  console.log('文档结构正确');
} else {
  console.log('<body> 不能在 <head> 前面');
}
```

上面代码中，compareDocumentPosition的返回值与4（又称掩码）进行与运算（&），得到一个布尔值，表示<head>是否在<body>前面。

## Node.prototype.isEqualNode()，Node.prototype.isSameNode()
isEqualNode方法返回一个布尔值，用于检查两个节点是否相等。所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同。

```javascript
var p1 = document.createElement('p');
var p2 = document.createElement('p');

p1.isEqualNode(p2) // true
```

isSameNode方法返回一个布尔值，表示两个节点是否为同一个节点。

```javascript
var p1 = document.createElement('p');
var p2 = document.createElement('p');

p1.isSameNode(p2) // false
p1.isSameNode(p1) // true
```

## Node.prototype.normalize()
normalize方法用于清理当前节点内部的所有文本节点（text）。它会去除空的文本节点，并且将毗邻的文本节点合并成一个，也就是说不存在空的文本节点，以及毗邻的文本节点。

```javascript
var wrapper = document.createElement('div');

wrapper.appendChild(document.createTextNode('Part 1 '));
wrapper.appendChild(document.createTextNode('Part 2 '));

wrapper.childNodes.length // 2
wrapper.normalize();
wrapper.childNodes.length // 1
```

上面代码使用normalize方法之前，wrapper节点有两个毗邻的文本子节点。使用normalize方法之后，两个文本子节点被合并成一个。

该方法是Text.splitText的逆方法，可以查看《Text 节点对象》一章，了解更多内容。

## Node.prototype.getRootNode()
getRootNode()方法返回当前节点所在文档的根节点document，与ownerDocument属性的作用相同。

```javascript
document.body.firstChild.getRootNode() === document
// true
document.body.firstChild.getRootNode() === document.body.firstChild.ownerDocument
// true
```

该方法可用于document节点自身，这一点与document.ownerDocument不同。

```javascript
document.getRootNode() // document
document.ownerDocument // null
```