---
title: JavaScript基础题
excerpt: 一些基础，如果感觉答案有问题请联系我
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- 面试
---

## 前言
从网上到处整理的一些基础题，可能有部分题因为改版答案出问题，如果感觉答案有问题请联系我

## 介绍JavaScript的基本数据类型
值类型(基本类型)：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。
引用数据类型：对象(Object)、数组(Array)、函数(Function)。
注：Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。

## 说说写JavaScript的基本规范？
1. 不要在同一行声明多个变量
2. 使用 ===或!==来比较true/false或者数值
3. switch必须带有default分支
4. 函数应该有返回值
5. for if else 必须使用大括号
6. 语句结束加分号
7. 命名要有意义，使用驼峰命名法

## jQuery使用建议
1. 尽量减少对dom元素的访问和操作
2. 尽量避免给dom元素绑定多个相同类型的事件处理函数，可以将多个相同类型事件处理函数合并到一个处理函数，通过数据状态来处理分支
3. 尽量避免使用toggle事件

## Ajax使用
全称 ： Asynchronous Javascript And XML
所谓异步，就是向服务器发送请求的时候，我们不必等待结果，而是可以同时做其他的事情，等到有了结果它自己会根据设定进行后续操作，与此同时，页面是不会发生整页刷新的，提高了用户体验。
创建Ajax的过程：
1. 创建XMLHttpRequest对象（异步调用对象）
```javascript
var xhr = new XMLHttpRequest();
```

2. 创建新的Http请求（方法、URL、是否异步）
```javascript
xhr.open('get','example.php',false);
```

3. 设置响应HTTP请求状态变化的函数。
onreadystatechange事件中readyState属性等于4。响应的HTTP状态为200(OK)或者304(Not Modified)。

4. 发送http请求
```javascript
xhr.send(data);
```

5. 获取异步调用返回的数据

**注意：**
1. 页面初次加载时，尽量在web服务器一次性输出所有相关的数据，只在页面加载完成之后，用户进行操作时采用ajax进行交互。
2. 同步ajax在IE上会产生页面假死的问题。所以建议采用异步ajax。
3. 尽量减少ajax请求次数
4. ajax安全问题，对于敏感数据在服务器端处理，避免在客户端处理过滤。对于关键业务逻辑代码也必须放在服务器端处理。

## JavaScript有几种类型的值？你能画一下他们的内存图吗？
基本数据类型存储在栈中，引用数据类型（对象）存储在堆中，指针放在栈中。
两种类型的区别是：存储位置不同；原始数据类型直接存储在栈中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；引用数据类型存储在堆中的对象,占据空间大、大小不固定,如果存储在栈中，将会影响程序运行的性能
引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

## 栈和堆的区别？
栈（stack）：由编译器自动分配释放，存放函数的参数值，局部变量等；
堆（heap）：一般由程序员分配释放，若程序员不释放，程序结束时可能由操作系统释放。

## Javascript作用链域
作用域链的原理和原型链很类似，如果这个变量在自己的作用域中没有，那么它会寻找父级的，直到最顶层。
注意：JS没有块级作用域，若要形成块级作用域，可通过（function（）｛｝）（）；立即执行的形式实现。

## 谈谈this的理解
1. this总是指向函数的直接调用者（而非间接调用者）
2. 如果有new关键字，this指向new出来的那个对象
3. 在事件中，this指向目标元素，特殊的是IE的attachEvent中的this总是指向全局对象window。

##  eval是做什么的？
它的功能是把对应的字符串解析成JS代码并运行；应该避免使用eval，不安全，非常耗性能（2次，一次解析成js语句，一次执行）。

## 什么是window对象? 什么是document对象?
window对象代表浏览器中打开的一个窗口。document对象代表整个html文档。实际上，document对象是window对象的一个属性。

## null，undefined的区别？
null表示一个对象被定义了，但存放了空指针，转换为数值时为0。
undefined表示声明的变量未初始化，转换为数值时为NAN。
typeof(null) -- object;
typeof(undefined) -- undefined

## 关于事件，IE与火狐的事件机制有什么区别？ 如何阻止冒泡？
IE为事件冒泡，Firefox同时支持事件捕获和事件冒泡。但并非所有浏览器都支持事件捕获。jQuery中使用event.stopPropagation()方法可阻止冒泡;
旧IE的方法 ev.cancelBubble = true;

## 什么是闭包（closure），为什么要用它？
闭包指的是一个函数可以访问另一个函数作用域中变量。常见的构造方法，是在一个函数内部定义另外一个函数。内部函数可以引用外层的变量；外层变量不会被垃圾回收机制回收。
注意，闭包的原理是作用域链，所以闭包访问的上级作用域中的变量是个对象，其值为其运算结束后的最后一个值。
优点：避免全局变量污染。缺点：容易造成内存泄漏。
例子：
```javascript
function makeFunc() {
    var name = "Mozilla";
    function displayName() {
        console.log(name); 
    }
    return displayName;
}
var myFunc = makeFunc();
myFunc();   //输出Mozilla
```

myFunc 变成一个 闭包。闭包是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。在我们的例子中，myFunc 是一个闭包，由 displayName 函数和闭包创建时存在的 "Mozilla" 字符串形成。

## javascript 代码中的"use strict";是什么意思 ? 使用它区别是什么？
除了正常模式运行外，ECMAscript添加了第二种运行模式：“严格模式”。
作用：
1. 消除js不合理，不严谨地方，减少怪异行为
2. 消除代码运行的不安全之处，
3. 提高编译器的效率，增加运行速度
4. 为未来的js新版本做铺垫。

## 如何判断一个对象是否属于某个类？
使用instanceof 即
```javascript
if(a instanceof Person){alert('yes');}
```

## new操作符具体干了什么呢?
1. 创建一个空对象，并且 this 变量引用该对象，同时还继承了该函数的原型。
2. 属性和方法被加入到 this 引用的对象中。
3. 新创建的对象由 this 所引用，并且最后隐式的返回 this 。

## Javascript中，执行时对象查找时，永远不会去查找原型的函数？
Object.hasOwnProperty(proName)：是用来判断一个对象是否有你给出名称的属性。不过需要注意的是，此方法无法检查该对象的原型链中是否具有该属性，该属性必须是对象本身的一个成员。

## 对JSON的了解？
全称：JavaScript Object Notation
JSON中对象通过“{}”来标识，一个“{}”代表一个对象，如{“AreaId”:”123”}，对象的值是键值对的形式（key：value）。JSON是JS的一个严格的子集，一种轻量级的数据交换格式，类似于xml。数据格式简单，易于读写，占用带宽小。
两个函数：
JSON.parse(str)
解析JSON字符串 把JSON字符串变成JavaScript值或对象
JSON.stringify(obj)
将一个JavaScript值(对象或者数组)转换为一个 JSON字符串
eval('('＋json＋')')
用eval方法注意加括号 而且这种方式更容易被攻击

## JS延迟加载的方式有哪些？
JS的延迟加载有助与提高页面的加载速度。
defer和async、动态创建DOM方式（用得最多）、按需异步载入JS
defer：延迟脚本。立即下载，但延迟执行（延迟到整个页面都解析完毕后再运行），按照脚本出现的先后顺序执行。
async：异步脚本。下载完立即执行，但不保证按照脚本出现的先后顺序执行。

## 同步和异步的区别?
同步的概念在操作系统中：不同进程协同完成某项工作而先后次序调整（通过阻塞、唤醒等方式），同步强调的是顺序性，谁先谁后。异步不存在顺序性。
同步：浏览器访问服务器，用户看到页面刷新，重新发请求，等请求完，页面刷新，新内容出现，用户看到新内容之后进行下一步操作。
异步：浏览器访问服务器请求，用户正常操作，浏览器在后端进行请求。等请求完，页面不刷新，新内容也会出现，用户看到新内容。

## 页面编码和被请求的资源编码如果不一致如何处理？
若请求的资源编码，如外引js文件编码与页面编码不同。可根据外引资源编码方式定义为 charset="utf-8"或"gbk"。
比如：http://www.yyy.com/a.html 中嵌入了一个http://www.xxx.com/test.js
a.html 的编码是gbk或gb2312的。 而引入的js编码为utf-8的 ，那就需要在引入的时候
```javascript
<script src="http://www.xxx.com/test.js&quot; charset="utf-8"></script>
```

## 模块化开发怎么做？
模块化开发指的是在解决某一个复杂问题或者一系列问题时，依照一种分类的思维把问题进行系统性的分解。模块化是一种将复杂系统分解为代码结构更合理，可维护性更高的可管理的模块方式。对于软件行业：系统被分解为一组高内聚，低耦合的模块。
1. 定义封装的模块
2. 定义新模块对其他模块的依赖
3. 可对其他模块的引入支持。在JavaScript中出现了一些非传统模块开发方式的规范。 CommonJS的模块规范，AMD（Asynchronous Module Definition），CMD（Common Module Definition）等。AMD是异步模块定义，所有的模块将被异步加载，模块加载不影响后边语句运行。

## AMD（Modules/Asynchronous-Definition）、CMD（Common Module Definition）规范区别？
AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。
区别：
1. 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。
2. CMD 推崇依赖就近，AMD 推崇依赖前置。
3. AMD 的 API 默认是一个当多个用，CMD 的 API 严格区分，推崇职责单一。
```javascript
// CMD
define(function(require, exports, module) {
    var a = require('./a')
    a.doSomething()
    // 此处略去 100 行
    var b = require('./b') // 依赖可以就近书写
    b.doSomething()
})
// AMD 默认推荐
define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
    a.doSomething();
    // 此处略去 100 行
    b.doSomething();
})
```

## requireJS的核心原理是什么？（如何动态加载的？如何避免多次加载的？如何缓存的？）
核心是js的加载模块，通过正则匹配模块以及模块的依赖关系，保证文件加载的先后顺序，根据文件的路径对加载过的文件做了缓存。

## call和apply
call（）方法和apply（）方法的作用相同，动态改变某个类的某个方法的运行环境。他们的区别在于接收参数的方式不同。在使用call（）方法时，传递给函数的参数必须逐个列举出来。使用apply（）时，传递给函数的是参数数组。

## documen.write和 innerHTML的区别
document.write()只能重绘整个页面
```javascript
setTimeout(function(){
       document.write('<p>5 secs later</p>');
}, 5000);
```
或
```javascript
window.onload = function() { document.write("HI");
```
innerHTML可以重绘页面的一部分

## 回流与重绘
当渲染树中的一部分(或全部)因为元素的规模尺寸，布局，隐藏等改变而需要重新构建。这就称为回流(reflow)。每个页面至少需要一次回流，就是在页面第一次加载的时候。在回流的时候，浏览器会使渲染树中受到影响的部分失效，并重新构造这部分渲染树。完成回流后，浏览器会重新绘制受影响的部分到屏幕中，该过程成为重绘

## DOM操作
1. 创建新节点
createDocumentFragment() //创建一个DOM片段
createElement() //创建一个具体的元素
createTextNode() //创建一个文本节点
2. 添加、移除、替换、插入
appendChild()
removeChild()
replaceChild()
insertBefore() //在已有的子节点前插入一个新的子节点
3. 查找
getElementsByTagName() //通过标签名称
getElementsByName() //通过元素的Name属性的值(IE容错能力较强，会得到一个数组，其中包括id等于name值的)
getElementById() //通过元素Id，唯一性

## 数组对象有哪些原生方法，列举一下
pop、push、shift、unshift、splice、reverse、sort、concat、join、slice、toString、indexOf、lastIndexOf、reduce、reduceRight
forEach、map、filter、every、some

## 那些操作会造成内存泄漏
全局变量、闭包、DOM清空或删除时，事件未清除、子元素存在引用

## 什么是Cookie 隔离？（或者：请求资源的时候不要带cookie怎么做）
通过使用多个非主要域名来请求静态文件，如果静态文件都放在主域名下，那静态文件请求的时候带有的cookie的数据提交给server是非常浪费的，还不如隔离开。因为cookie有域的限制，因此不能跨域提交请求，故使用非主要域名的时候，请求头中就不会带有cookie数据，这样可以降低请求头的大小，降低请求时间，从而达到降低整体请求延时的目的。同时这种方式不会将cookie传入server，也减少了server对cookie的处理分析环节，提高了server的http请求的解析速度。

## 响应事件
onclick鼠标点击某个对象；onfocus获取焦点；onblur失去焦点；onmousedown鼠标被按下

## flash和js通过什么类如何交互?
Flash提供了ExternalInterface接口与JavaScript通信，ExternalInterface有两个方法，call和addCallback，call的作用是让Flash调用js里的方法，addCallback是用来注册flash函数让js调用。

## Flash与Ajax各自的优缺点？
Flash：适合处理多媒体、矢量图形、访问机器。但对css、处理文本不足，不容易被搜索。
Ajax：对css、文本支持很好，但对多媒体、矢量图形、访问机器不足。

## 有效的javascript变量定义规则
第一个字符必须是一个字母、下划线（_）或一个美元符号（$）；其他字符可以是字母、下划线、美元符号或数字。

## XML与JSON的区别？
1. 数据体积方面。JSON相对于XML来讲，数据的体积小，传递的速度更快些。
2. 数据交互方面。JSON与JavaScript的交互更加方便，更容易解析处理，更好的数据交互。
3. 数据描述方面。JSON对数据的描述性比XML较差。
4. 传输速度方面。JSON的速度要远远快于XML。

## HTML与XML的区别？
1. XML用来传输和存储数据，HTML用来显示数据；
2. XML使用的标签不用预先定义
3. XML标签必须成对出现
4. XML对大小写敏感
5. XML中空格不会被删减
6. XML中所有特殊符号必须用编码表示
7. XML中的图片必须有文字说明

## 渐进增强与优雅降级
渐进增强：针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进，达到更好的用户体验。
优雅降级：一开始就构建完整的功能，然后再针对低版本浏览器进行兼容。

## Web Worker和Web Socket？
web socket：在一个单独的持久连接上提供全双工、双向的通信。使用自定义的协议（ws://、wss://），同源策略对web socket不适用。
web worker：运行在后台的JavaScript，不影响页面的性能。
创建worker：var worker = new Worker(url);
向worker发送数据：worker.postMessage(data);
接收worker返回的数据：worker.onmessage
终止一个worker的执行：worker.terminate();

## JS垃圾回收机制？
1. 标记清除：
这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。
这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。定期的，垃圾回收器将从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象。从根开始，垃圾回收器将找到所有可以获得的对象和所有不能获得的对象。
2. 引用计数：
这是最简单的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。
该算法有个限制：无法处理循环引用。两个对象被创建，并互相引用，形成了一个循环。它们被调用之后不会离开函数作用域，所以它们已经没有用了，可以被回收了。然而，引用计数算法考虑到它们互相都有至少一次引用，所以它们不会被回收。

## web应用从服务器主动推送data到客户端的方式？
JavaScript数据推送：commet（基于http长连接的服务器推送技术）。
基于web socket的推送：SSE（server-send Event）

## 如何删除一个cookie？
1. 将cookie的失效时间设置为过去的时间（expires）
```javascript
document.cookie = ‘user=’+ encodeURIComponent(‘name’) + ';
expires=’+ new Date(0);
```
2. 将系统时间设置为当前时间往前一点时间
```javascript
var data = new Date();
date.setDate(date.getDate()-1);
```

## attribute与property的区别？
attribute是dom元素在文档中作为html标签拥有的属性
property是dom元素在js中作为对象拥有的属性。
所以，对于html的标准属性来说，attribute和property是同步的，是会自动更新的。但对于自定义属性，他们不同步。

## Ajax请求的页面历史记录状态问题？
1. 通过location.hash记录状态，让浏览器记录Ajax请求时页面状态的变化。
2. 通过HTML5的history.pushstate，来实现浏览器地址栏的无刷新改变。

## 防抖节流
**防抖**
触发高频事件后n秒内函数只会执行一次，如果n秒内高频事件再次被触发，则重新计算时间
思路：
每次触发事件时都取消之前的延时调用方法
```javascript
function debounce(fn) {
    let timeout = null; // 创建一个标记用来存放定时器的返回值
    return function () {
        clearTimeout(timeout); // 每当用户输入的时候把前一个 setTimeout clear 掉
        timeout = setTimeout(() => {
            // 然后又创建一个新的 setTimeout,
            // 这样就能保证输入字符后的 interval 间隔内如果还有字符输入的话
            // 就不会执行 fn 函数
            fn.apply(this, arguments);
        }, 500);
    };
}
function sayHi() {
    console.log('防抖成功');
}

var inp = document.getElementById('inp');
inp.addEventListener('input', debounce(sayHi)); // 防抖
```
**节流**
高频事件触发，但在n秒内只会执行一次，所以节流会稀释函数的执行频率
思路：
每次触发事件时都判断当前是否有等待执行的延时函数
```javascript
function throttle(fn) {
    let canRun = true; // 通过闭包保存一个标记
    return function () {
        if (!canRun) return; // 在函数开头判断标记是否为true，不为true则return
        canRun = false; // 立即设置为false
        setTimeout(() => { // 将外部传入的函数的执行放在setTimeout中
            fn.apply(this, arguments);
            // 最后在setTimeout执行完毕后再把标记设置为true(关键)表示可以执行下一次循环了。
            // 当定时器没有执行的时候标记永远是false，在开头被return掉
            canRun = true;
        }, 500);
    };
}
function sayHi(e) {
    console.log(e.target.innerWidth, e.target.innerHeight);
}
window.addEventListener('resize', throttle(sayHi));
```

## 模块化发展历程
可从IIFE、AMD、CMD、CommonJS、UMD、webpack(require.ensure)、ES Module、&lt;script type="module"&gt; 这几个角度考虑。
模块化主要是用来抽离公共代码，隔离作用域，避免变量冲突等。
**IIFE：** 使用自执行函数来编写模块化，特点：在一个单独的函数作用域中执行代码，避免变量冲突。
```javascript
(function(){
  return {
    data:[]
  }
})()
```
**AMD：** 使用requireJS 来编写模块化，特点：依赖必须提前声明好。
```javascript
define('./index.js',function(code){
    // code 就是index.js 返回的内容
})
```
**CMD：** 使用seaJS 来编写模块化，特点：支持动态引入依赖文件。
```javascript
define(function(require, exports, module) {  
  var indexCode = require('./index.js');
})
```
**CommonJS：** nodejs 中自带的模块化。
```javascript
var fs = require('fs');
```
**UMD：**兼容AMD，CommonJS 模块化语法。
**webpack(require.ensure)：**webpack 2.x 版本中的代码分割。
**ES Modules：** ES6 引入的模块化，支持import 来引入另一个 js 。
```javascript
import a from 'a';
```

## npm 模块安装机制：
- 发出npm install命令
- 查询node_modules目录之中是否已经存在指定模块
- 若存在，不再重新安装
- 若不存在
    - npm 向 registry 查询模块压缩包的网址
    - 下载压缩包，存放在根目录下的.npm目录里
    - 解压压缩包到当前项目的node_modules目录

## npm 实现原理
输入 npm install 命令并敲下回车后，会经历如下几个阶段（以 npm 5.5.1 为例）：
1. **执行工程自身 preinstall**，当前 npm 工程如果定义了 preinstall 钩子此时会被执行。
2. **确定首层依赖模块**，首先需要做的是确定工程中的首层依赖，也就是 dependencies 和 devDependencies 属性中直接指定的模块（假设此时没有添加 npm install 参数）。工程本身是整棵依赖树的根节点，每个首层依赖模块都是根节点下面的一棵子树，npm 会开启多进程从每个首层依赖模块开始逐步寻找更深层级的节点。
3. **获取模块**，获取模块是一个递归的过程，分为以下几步：
    - 获取模块信息。在下载一个模块之前，首先要确定其版本，这是因为 package.json 中往往是 semantic version（semver，语义化版本）。此时如果版本描述文件（npm-shrinkwrap.json 或 package-lock.json）中有该模块信息直接拿即可，如果没有则从仓库获取。如 packaeg.json 中某个包的版本是 ^1.1.0，npm 就会去仓库中获取符合 1.x.x 形式的最新版本。
    - 获取模块实体。上一步会获取到模块的压缩包地址（resolved 字段），npm 会用此地址检查本地缓存，缓存中有就直接拿，如果没有则从仓库下载。
    - 查找该模块依赖，如果有依赖则回到第1步，如果没有则停止。
4. **模块扁平化（dedupe）**，上一步获取到的是一棵完整的依赖树，其中可能包含大量重复模块。比如 A 模块依赖于 loadsh，B 模块同样依赖于 lodash。在 npm3 以前会严格按照依赖树的结构进行安装，因此会造成模块冗余。从 npm3 开始默认加入了一个 dedupe 的过程。它会遍历所有节点，逐个将模块放在根节点下面，也就是 node-modules 的第一层。当发现有重复模块时，则将其丢弃。
这里需要对重复模块进行一个定义，它指的是模块名相同且 semver 兼容。每个 semver 都对应一段版本允许范围，如果两个模块的版本允许范围存在交集，那么就可以得到一个兼容版本，而不必版本号完全一致，这可以使更多冗余模块在 dedupe 过程中被去掉。
比如 node-modules 下 foo 模块依赖 lodash@^1.0.0，bar 模块依赖 lodash@^1.1.0，则 ^1.1.0为兼容版本。
而当 foo 依赖 lodash@^2.0.0，bar 依赖 lodash@^1.1.0，则依据 semver 的规则，二者不存在兼容版本。会将一个版本放在 node_modules 中，另一个仍保留在依赖树里。
5. **安装模块**，这一步将会更新工程中的 node_modules，并执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。
6. **执行工程自身生命周期**，当前 npm 工程如果定义了钩子此时会被执行（按照 install、postinstall、prepublish、prepare 的顺序）。最后一步是生成或更新版本描述文件，npm install 过程完成。

## ES5的继承和ES6的继承有什么区别？
ES5的继承时通过prototype或构造函数机制来实现。ES5的继承实质上是先创建子类的实例对象，然后再将父类的方法添加到this上（Parent.apply(this)）。
ES6的继承机制完全不同，实质上是先创建父类的实例对象this（所以必须先调用父类的super()方法），然后再用子类的构造函数修改this。
具体的：ES6通过class关键字定义类，里面有构造方法，类之间通过extends关键字实现继承。子类必须在constructor方法中调用super方法，否则新建实例报错。因为子类没有自己的this对象，而是继承了父类的this对象，然后对其进行加工。如果不调用super方法，子类得不到this对象。
ps：super关键字指代父类的实例，即父类的this对象。在子类构造函数中，调用super后，才可使用this关键字，否则报错。

## 定时器的执行顺序或机制？
因为js是单线程的，浏览器遇到setTimeout或者setInterval会先执行完当前的代码块，在此之前会把定时器推入浏览器的待执行事件队列里面，等到浏览器执行完当前代码之后会看一下事件队列里面有没有任务，有的话才执行定时器的代码。所以即使把定时器的时间设置为0还是会先执行当前的一些代码。
```javascript
function test(){
    var aa = 0;
    var testSet = setInterval(function(){
        aa++;
        console.log(123);
        if(aa<10){
            clearInterval(testSet);
        }
    },20);
  var testSet1 = setTimeout(function(){
    console.log(321)
  },1000);
  for(var i=0;i<10;i++){
    console.log('test');
  }
}
test()

//输出结果：
// test 10次
// undefined
// 123
// 321
```

## 参考链接
[50道JavaScript基础面试题（附答案）](https://segmentfault.com/a/1190000015288700)