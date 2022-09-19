---
title: DOM~
excerpt: 本文将从什么是DOM，BOM入手，讲述什么是DOM、BOM，什么是虚拟DOM，什么是diff算法~
categories:
- 技术文章
tags:
- js
- 面试
- dom
- vue
- react
---

## 什么是DOM
DOM就是文档对象模型，什么是文档对象模型？这就需要好好说说了。

HTML的文档document页面是一切的基础，没有它dom就无从谈起。

当创建好一个页面并加载到浏览器时，DOM就悄然而生，它会把网页文档转换为一个文档对象，主要功能是处理网页内容。

在这个文档对象里，所有的元素呈现出一种层次结构，就是说除了顶级元素html外，其他所有元素都被包含在另外的元素中。那么它的树就应该是下面这样的一颗倒长的树。

## 节点类型
节点表示网络中的一个连接点，一个网络就是由一些节点构成的。而文档就是由节点构成集合，只不过他们是构成节点树上的树枝树叶而已。这些节点有许多不同的类型，我们先来看看其中的三种：元素节点、文本节点和属性节点。

HTML的标签元素就是DOM的**元素节点**，它提供了一份文档的结构。但这份文档本身不会包含任何内容，因此元素节点可以包含其他的节点。**文本节点**是节点类型的一种，它总是被包含在元素节点内部，形成页面文档的主要内容。**属性节点**用于对元素做出个个具体的描述，例如：a元素的href属性，img元素的alt属性。属性总是被放在起始标签里，因此属性节点也总是被包含在元素节点中。虽然并非所有的元素节点都包含属性，但所有的属性一定都被元素节点包含。

## 节点操作
### 获取元素节点
获取元素节点有4种方法，分别通过元素ID，标签名字，类名和css选择器来获取。

#### 元素ID
getElementById方法是document对象特有的函数，传入一个参数即元素的id属性值，将返回一个对象。

这个对象对应那个id属性为指定值的节点，我们还可以用typeof来验证它的类型：

```javascript
document.getElementById("car");
alert(typeof document.getElementById("car"));
```

实际上文档中每一个元素都是一个对象，利用DOM提供的方法可以得到任意一个对象。

不过要是为每一个元素都定义一个独一无二的ID值那就太麻烦了，所以DOM还提供了另外的方法来获取没有id的对象。

#### 标签名字
getElementsByTagName方法会返回一个对象数组，数组的元素就是和getElementById差不多的获取到的对象：

```javascript
document.getElementsByTagName("li");
```

还可以用数组的方法length来获取这个数组的长度：

```javascript
document.getElementsByTagName("li").length;
```

如果觉得在编程的过程中反复书写这段代码会很麻烦，那还可以将它赋值给一个变量即可：

```javascript
var lis = document.getElementsByTagName("li");
for (var i = 0; i < lis.length; i++) {
    alert(typeof lis[i]);
}
```

另外还可以将getElementById和getElementsByTagName结合起来使用，缩小选取范围。

比如现在只想知道id是car的元素下面有多少个列表项，那么就可以这样：

```javascript
var car = document.getElementById("car");
var lis = car.document.getElementsByTagName("li");
```

####  类名
getElementsByClassName方法让我们能够通过class类名来访问元素。

它的返回值和getElementsByTagName类似，都是返回一个对象数组：

```javascript
document.getElementsByClassName("sale");
```

值得注意的是它还可以匹配含有多个class的元素，指定多个类名的时候只需要在字符串参数中间yoga空格分隔类名就可以了，顺序不重要前后都可以。

另外它也可以和前面两种方法混合使用，用法和getElementById和getElementsByTagName结合使用的例子一致。

#### CSS选择器
还有html5中新增的两个方法，让我们可以用css选择器的方法来选择DOM节点，这两个方法必须在IE8以上的现代浏览器中才能使用。

第一个方法是返回了单个节点，如果有多个匹配元素就只返回第一个，如果找不到匹配就返回null。

第二个方法是返回一个节点列表集合。参数则都为CSS选择器字符串：

```javascript
document.querySelector("#foo");
document.querySelectorAll(".bar");
```

### 获取和设置属性
在得到元素后，就可以获取他们的属性，然后更改属性的值了：

#### getAttribute
getAttribute函数是一个属于节点对象的方法，可以通过传入参数获取节点对象下的各种属性：

```javascript
var p = document.getElementById("parse");
var title = p.getAttribute("title");
alert(title);
```

#### setAttribute
setAttribute方法与getAttribute对应，它不再是获取，而是修改，因此它的参数有两个：

第一个是属性名，第二个是想要修改为的值：

```javascript
var p = document.getElementById("parse");
p.setAttribute("title","nice day!");
alert(p.getAttribute("title"));
```

值得注意的是，这个时候如果去查看源代码，会发现P元素的title属性值并没改变。

这是因为DOM的工作模式是：先加载静态内容，再动态刷新，动态刷新不影响文档的静态内容。

### 在树上爬行
**childNodes**，在一颗节点树上，这个属性可以用来获取一个元素的所有子元素，得到一个包含所有子元素的数组：

```javascript
// 如果要获得body元素下的全体子元素
document.getElementsByTagName(“body”)[0].childNodes;
```

**parentNode**，获取当前节点的父节点元素，如果指定节点没有父元素那么会返回null。

**nodeType**, 每个节点都有nodeType属性，这个属性是一个数值，让我们可以知道正在处理哪种类型的节点。

节点类型有十多种，但其中我们最需要了解的有3种：

1. 元素节点的nodeType属性值是1
2. 属性节点的nodeType属性值是2
3. 文本节点的nodeType属性值是3

这就意味着我们可以只对特定类型的节点进行处理，比如只处理元素节点。

**nodeValue**，如果想改变文本节点的值，就可以使用这个属性：

比如当有一个p元素节点，里面有一些文本内容，如果想取得这些文本内容，那么直接对p元素使用nodeValue会得到一个null空值。因为这样得到的是p元素节点的值而不是它的子元素文本节点的值，因此可以这样来得到真正需要的内容：


```javascript
p.childNodes[0].nodeValue;
```

firstChild和lastChild是对子元素数组更简易操作的属性，就是childNodes的第一个和最后一个，这样更简短也更有可读性。

### 动态创建
前面的方法都是对已经存在的元素做出搜索和修改。

然而js也可以用来改变网页的结构和内容，可以通过创建新元素和改变现有元素来改变网页结构。

#### 传统方法
document.write()方法可以方便快捷的把字符串插入到文档中

innerHTML属性可以用来读写html的内容

#### DOM操作法
如果想把一段文本内容放到p元素中，然后将p元素插入到页面的某个节点后，那么这个任务可以分为几个步骤：

1. 创建一个p元素节点
2. 把这个p元素节点最佳到文档中的#parent元素节点上
3. 创建一个文本节点
4. 把这个文本节点追加到刚才创建的p元素节点上

```javascript
var para = document.createElement("p");
var parent = document.getElementById("parent");
parent.appendChild(para);
var txt = document.createTextNode("hello world");
para.appendChild(txt);
```

来认识一下这几个方法：

- **createElement**，创建一个元素节点，传入的参数就是标签名字符串。但它只是一个文档碎片，还不是DOM节点树上的组成部分，无法显示在浏览器里。
- **createTextNode**，创建一个文本节点用于放文本内容，和上面几乎一样，只是传入的参数就是文本字符串，创建好后依旧是文档中的一个游荡的孤儿。
- **appendChild**，想把新创建的节点插入节点树的最简单办法之一，让它成为某个节点的一个子节点。
- **insertBefore**，这个方法可以在已有元素前插入一个新元素。

关于insertBefore，调用这个方法时必须知道三件事：新元素和目标元素和父元素，其语法是这样的：

```javascript
parentElement.insertBefore(newElement, targetElement);
```

比如想在一个img图片前插入一个p标签，那么可以这样做：

```javascript
var img = document.getElementById("img");
var p = document.getElementById("p");
img.parentNode.insertBefore(p, img);
```

## 操作封装
前面说到过insertBefore方法，那是不是也该有个对应的insertAfter()呢，很遗憾，没有，那么我们其实可以自己实现一个这样的方法：

```javascript 
function insertAfter(newEle, targetEle) {
    var parent = targetEle.parentNode;
    if (parent.lastChild == targetEle) {
        parent.appendChild(newEle);
    } 
    else {
        parent.insertBefore(newEle, targetEle.nextSibling);
    }
}
```

它的实现步骤是这样的：
1. 首先，这个函数有两个参数，一个是将被插入的元素，一个是目标元素；
2. 把目标元素的父元素保存到变量parent里；
3. 检查目标元素是不是父元素parent的最后一个子元素；
4. 如果是，就用appendChild方法把新元素追加到父元素parent上，这样新元素就恰好被插入到目标元素之后；
5. 如果不是，就把新元素插入到目标元素的下一个兄弟元素之前，这样新元素就在目标元素之后

通过这样一个函数，加上已知的几个方法，就能自己封装出自己所需要的方法了。

## BOM
BOM，browser object model，浏览器对象模型，这个对象就是对应着浏览器窗口window。

它提供了一些方法用于访问浏览器的功能，这些功能和网页内容无关。

### window对象
window对象是BOM的核心，表示浏览器正打开的窗口，它是一个全局对象。它还有一些属性方法和子对象，我们其实已经默默的使用过它了。比如alert()方法，但因为它是widow对象的直接后代，所以不需要加上window前缀。另外我们定义的全局变量，其实也是定义到了window上的。

#### 全局作用域

```javascript
var car = "Tesla";
function run() {
    alert(this.car);
};
alert(window.car);
run();
window.run();
```

在全局作用域中定义了变量car和函数run，它们会自动归为window对象的，因此可以通过window点来访问它们。

另外run函数存在于全局作用域中，因此this也被指向window。

#### 系统对话框
alert(), confirm(), prompt()这几个方法可以调用系统对话框向用户显示消息。

**alert**生成一个警告框，用于显示一些用户无法控制的消息，看过后只能关闭了事。

**confirm**生成的对话框和前者的不同在于多一个cancel按钮，可以让用户决定是否执行，并返回一个布尔值；true表示点击了确定，false表示点击了取消。

**prompt**则是生成一个提示框，用于提示用户输入一些文本内容，这个方法接受2个参数：文本提示和输入框的默认值。用户操作点击ok后，返回文本框输入的值；如果点击cancel或者直接关闭，则会返回null。

#### 窗口操作
方法说明window.moveBy()用于把窗口移到一个相对位置window.moveTo()用于把窗口移到一个特定位置window.resizeBy()用来按一个相对量来改变窗口大小window.resizeTo()用来把窗口大小改变到特定大小。

###  location对象
这个对象让我们可以访问当前载入的URI（统一资源标识符）的任意信息。可以通过location的属性来改变当前url，当然下面这两种写法是一样的效果：

```javascript
window.location = "www.baidu.com";
location.href = "www.baidu.com";
```

另外也能使用location的属性改变URL，以重新加载

### navigator对象

这个对象提供几个属性，用于辅助检测浏览器环境，因为js经常做的事情之一就检测用户正在使用哪种浏览器。不过这个对象的属性非常多，可以在浏览器的调试工具中直接打出来看看它的属性和方法

### history对象
这个对象提供了通过用户访问产生的浏览历史来向前或向后移动的方法。用go()方法可以在历史记录中任意跳转，可以向前也可以向后，这个方法接受一个参数，表示向前或向后页面数的一个整数，负值表示向后，正数表示向前。

另外还有两个简写方法back和forward来替代go，只是每次只能前进或后退一页，无需参数：

```javascript
//后退一页
history.go(-1);
history.back();
//前进一页
history.go(1);
history.forward();
//后退两页
history.go(-2);
```

## 虚拟DOM
虚拟 DOM （Virtual DOM ）说简单点，就是一个普通的 JavaScript 对象，包含了 tag、props、children 三个属性。

```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
```

上面的 HTML 转换为虚拟 DOM 如下：

```javascript
{
  tag: 'div',
  props: {
    id: 'app'
  },
  chidren: [
    {
      tag: 'p',
      props: {
        className: 'text'
      },
      chidren: [
        'hello world!!!'
      ]
    }
  ]
}
```

该对象就是我们常说的虚拟 DOM 了，因为 DOM 是树形结构，所以使用 JavaScript 对象就能很简单的表示。而原生 DOM 因为浏览器厂商需要实现众多的规范（各种 HTML5 属性、DOM事件），即使创建一个空的 div 也要付出昂贵的代价。虚拟 DOM 提升性能的点在于 DOM 发生变化的时候，通过 diff 算法比对 JavaScript 原生对象，计算出需要变更的 DOM，然后只对变化的 DOM 进行操作，而不是更新整个视图。

### 虚拟DOM慢？
DOM 操作慢其实是对比于 JS 原生 API，如数组操作，任何基于 DOM 的库（Vue/React）都不可能在操作 DOM 时比 DOM 快。

### 虚拟DOM的优点
1. 减少 DOM 操作
- 虚拟 DOM 可以将多次操作合并为一次操作，比如你添加 1000 个节点，却是一个接一个操作的（减少频率）
- 虚拟 DOM 借助 DOM diff 可以把多余的操作省掉，比如你添加 1000 个节点，其实只有 10 个是新增的（减少范围）

2. 跨平台
- 虚拟 DOM 不仅可以变成 DOM，还可以变成小程序、iOS 应用、安卓应用，因为虚拟 DOM 本质上只是一个 JS 对象

### 虚拟DOM在React和Vue中的样子
在Vue中：

```javascript
const vNode = {
  tag: "div", // 标签名 or 组件名
  data: {
    class: "red", // 标签上的属性
    on: {
      click: () => {} // 事件
    }
  },
  children: [ // 子元素们
    { tag: "span", ... },
    { tag: "span", ... }
  ],
  ...
}
```

在React中：

```javascript
const vNode = {
  key: null,
  props: {
    children: [  // 子元素们
       { type: 'span', ... }, 
       { type: 'span', ... }
    ],
    className: "red" // 标签上的属性
    onClick: () => {} // 事件
  },
  ref: null,
  type: "div", // 标签名 or 组件名
  ...
}
```

### 如何创建虚拟DOM
- React.createElement

```javascript
createElement('div',{className:'red',onClick:()=> {}},[
    createElement('span', {}, 'span1'),
    createElement('span', {}, 'span2')
  ]
)
```

- Vue

```javascrip
//只能在 render 函数里得到 h
h('div', {
  class: 'red',
  on: {
    click: () => { }
  },
}, [h('span',{},'span1'), h('span', {}, 'span2'])
```

### 创建虚拟 DOM
- React

```javascript
<div className="red" onClick={fn}>
    <span>span1</span>
    <span>span2</span>
</div>
```

通过 babel 转为 createElement 形式

- Vue Template

```javascrip
<div class="red" @click="fn">
  <span>span1</span>
  <span>span2</span>
</div>
```

通过 vue-loader 转为 h 形式

### 小结
虚拟 DOM 是什么
- 一个能代表 DOM 树的对象，通常含有标签名、标签上的属性、事件监听和子元素们，以及其他属性

虚拟 DOM 有什么优点
- 能减少不必要的 DOM 操作
- 能跨平台渲染

虚拟 DOM 有什么缺点
- 需要额外的创建函数，如 createElement 或 h，但可以通过 JSX 来简化成 XML 写法

## DOM diff
DOM diff算法是虚拟 DOM 的对比算法，就是一个函数，我们称之为 patch；patches = patch(oldVNode, newVNode)；patches 就是要运行的 DOM 操作。

### 大概逻辑
**Tree diff**
- 将新旧两棵树逐层对比，找出哪些节点需要更新
- 如果节点是组件就看 Component diff
- 如果节点是标签就看 Element diff

**Component diff**
- 如果节点是组件，就先看组件类型
- 类型不同直接替换（删除旧的）
- 类型相同则只更新属性
- 然后深入组件做 Tree diff（递归）

**Element diff**
- 如果节点是原生标签，则看标签名
- 标签名不同直接替换，相同则只更新属性
- 然后进入标签后代做 Tree diff（递归）




## 参考链接
[DOM 是什么？](https://www.zhihu.com/question/34219998)
[虚拟 DOM 到底是什么？](https://juejin.im/post/6844903870229905422)
饥人谷


