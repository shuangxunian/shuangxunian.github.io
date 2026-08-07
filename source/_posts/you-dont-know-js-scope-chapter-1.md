---
title: "《你不知道的 JavaScript（上卷）》第一章：作用域是什么"
excerpt: "从一次变量访问出发，厘清 JavaScript 的编译、词法作用域，以及 LHS/RHS 查询；同时补充 let、const、TDZ 和现代 JavaScript 中常见的误区。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 作用域, 闭包, 前端基础]
---

> 本文以《你不知道的 JavaScript（上卷）》第 1 章的 LHS/RHS 心智模型为主线，并结合现代 JavaScript 的 `let`、`const` 与块级作用域补充说明。LHS/RHS 不是 ECMAScript 规范中的术语，但它非常适合用来训练“这一次是在读还是在写”的判断能力。

## 为什么从作用域开始

变量并不是“写过就能用”。下面这些常见问题，根源都是没有弄清标识符在何处被创建、何时可访问、以及运行时如何查找它：

- 为什么回调可以记住外层变量？
- 为什么循环里的 `var` 和 `let` 行为不同？
- 为什么有时是 `ReferenceError`，有时又是 `TypeError`？
- 为什么漏写声明会污染全局环境？

**作用域（scope）是一套关于标识符可见性与查找路径的规则。** 它决定代码在某个位置能访问哪些绑定（binding），也为闭包建立基础。它和 `this` 的绑定规则、对象的原型链是三件不同的事，不应混为一谈。

本文目标很明确：面对任意一段简单代码，都能回答“这个名字从哪里来”“此次访问是在读取还是写入”“找不到时会发生什么”。

## JavaScript 也有编译阶段

把 JavaScript 简化为“逐行解释、读到哪里执行到哪里”并不准确。现代 JavaScript 引擎会在执行代码前处理源码：分析语法、建立声明信息，并生成适合执行的内部表示；具体是否生成字节码、何时优化为机器码，则由各引擎自行决定。

可以把过程粗略理解为三步：

1. **词法分析（tokenizing / lexing）**：把 `var total = price + tax;` 拆为 `var`、`total`、`=`、`price`、`+`、`tax`、`;` 等语法单元。
2. **语法分析（parsing）**：根据语法规则组织这些单元，形成抽象语法树（AST）。
3. **生成并执行**：引擎根据 AST 建立执行所需的信息，并执行生成的内部指令；执行过程中还可能继续优化。

这不是在描述某一个引擎的精确实现，而是理解声明与运行时访问的实用模型。重点是：**声明的处理发生在语句逐条执行之前，变量的读取和赋值发生在运行时。**

## 用三个角色理解一次声明

书中把 `var count = 2;` 的处理拆成三个协作角色：

- **引擎**：驱动整体执行，负责在运行时发起标识符查询。
- **编译器**：分析代码并收集声明信息。
- **作用域**：保存当前可见的绑定，并按作用域链参与查找。

这三个名字是教学模型，不是规范要求引擎必须具备的三个独立模块。借助它，可以将下面这句代码分成两件事：

```js
var count = 2;
```

1. 声明 `count`，让它成为当前函数作用域（或全局作用域）中的绑定；
2. 执行 `count = 2`，将值写入该绑定。

对 `let`、`const` 也应这样理解，但有一个关键差异：它们属于块级绑定，在进入其作用域时已经存在，却会在声明语句执行前保持不可访问状态。这段区间称为**暂时性死区**（TDZ）。

```js
{
  console.log(status); // ReferenceError：仍处于 TDZ
  let status = "ready";
}
```

因此，“变量提升”是一个容易造成误解的说法。`var` 的绑定会以 `undefined` 初始化；`let` 和 `const` 的绑定也会预先建立，但在初始化前不能读取或写入。

## LHS 与 RHS：判断读还是写

LHS 是 **Left-Hand Side**，RHS 是 **Right-Hand Side**。它们并不只是等号左右两边的字面位置：

- **RHS 查询**：需要取得一个标识符当前保存的值，也就是“读”。
- **LHS 查询**：需要找到一个标识符对应的绑定，以便向其中写入值，也就是“写”。

下面的代码同时出现两种查询：

```js
let total = price + tax;
```

- `price`、`tax` 需要拿到值参与计算，是 RHS 查询；
- `total` 被初始化为计算结果，可将其理解为一次 LHS 写入。

### 常见的 RHS 查询

```js
console.log(total);  // 读取 total
if (score >= 60) {}  // 读取 score
format(data);        // 读取 format，读取 data
```

函数调用本身也先要读取函数标识符：`format(data)` 中，`format` 和 `data` 都是 RHS 查询。

### 常见的 LHS 查询

```js
name = "张三";      // 向 name 写入
let { id } = user;  // 初始化 id；读取 user

function greet(message) {
  return message;
}
greet("你好");     // 调用时，实参会写入形参 message
```

`++`、`--` 与复合赋值尤其值得注意：它们既要读旧值，又要写回新值。

```js
index += 1; // 读取 index（RHS），再将结果写回 index（LHS）
```

### 不要把属性名当成标识符查询

LHS/RHS 讨论的是**作用域中的标识符**，不能机械地套到点号后的属性名上。

```js
user.profile.age = 18;
```

这里 `user` 是一次 RHS 查询，引擎先要取得对象。之后的 `profile`、`age` 是对象属性访问与属性写入，遵循对象和原型相关规则，并不是在作用域链中对名为 `profile` 或 `age` 的变量做 LHS/RHS 查询。同理，`user.name` 中是 `user` 被读取，而非对变量 `name` 做 RHS 查询。

## 作用域链：从内向外查找

作用域通常由全局、函数和块嵌套而成。一个标识符访问会先在当前位置所在的作用域中找，找不到就逐层向外，直到全局作用域；仍找不到，查询失败。

```js
const siteName = "My Blog";

function render() {
  const section = "JavaScript";

  function title() {
    return `${siteName}: ${section}`;
  }

  return title();
}
```

在 `title` 内读取 `section` 时，会先查 `title` 自己的作用域，再查 `render` 的作用域。读取 `siteName` 时则继续查到全局作用域。这条由内向外的路径就是作用域链。

内层的同名绑定会遮蔽外层绑定（shadowing）：

```js
const theme = "light";

function preview() {
  const theme = "dark";
  return theme; // "dark"
}
```

`var` 创建函数作用域绑定；`let`、`const` 和 `class` 创建块级作用域绑定。理解这一点，是解释循环闭包差异的前提：`for (let i = 0; ...)` 的每轮迭代会获得独立的 `i` 绑定，而 `var` 不会。

## 查询失败：先区分“名字不存在”与“值不能用”

### RHS 找不到标识符

当引擎需要读取一个根本不存在的标识符，通常会抛出 `ReferenceError`：

```js
console.log(missingValue); // ReferenceError
```

TDZ 中的 `let`/`const` 绑定也会在访问时抛出 `ReferenceError`，虽然它已经属于当前作用域。

有一个常见例外：`typeof` 对未声明标识符不会抛错。

```js
typeof missingValue; // "undefined"
```

但对处于 TDZ 的变量，`typeof` 仍然会抛出 `ReferenceError`。

### LHS 找不到标识符

向未声明标识符赋值时，行为取决于是否为严格模式：

```js
function createLeak() {
  leakedValue = 99;
}

createLeak();
```

在传统的非严格脚本环境中，这种赋值可能在全局对象上创建属性，造成隐式全局变量。在严格模式、ES 模块中，这会直接抛出 `ReferenceError`：

```js
function createValue() {
  "use strict";
  localValue = 99; // ReferenceError
}
```

实际项目应始终先使用 `const` 或 `let` 声明变量，并启用 lint 规则阻止此类漏写。

### `ReferenceError` 与 `TypeError`

`ReferenceError` 说明标识符无法按当前规则被解析或访问；`TypeError` 则表示标识符已经得到一个值，但对该值执行的操作不合法。

```js
let handler;
handler(); // TypeError：handler 的值是 undefined，不能调用

const config = null;
config.enabled; // TypeError：不能读取 null 的属性
```

还有一种常见的 `TypeError`：给 `const` 绑定重新赋值。绑定存在，但它不允许被再次写入。

```js
const port = 3000;
port = 4000; // TypeError
```

## 练习：逐项标注查询

请分析下面的代码。先自己作答，再展开答案。

```js
function add(a) {
  const b = a;
  return a + b;
}

const result = add(2);
```

<details>
<summary>查看解析</summary>

运行时的主要查询如下：

- `add(2)`：读取 `add`，是 RHS；
- 调用 `add` 时，实参 `2` 写入形参 `a`，是 LHS；
- `const b = a`：读取 `a`，是 RHS；初始化 `b`，可按 LHS 模型理解；
- `return a + b`：读取 `a`、`b`，各一次 RHS；
- `const result = add(2)`：初始化 `result`，可按 LHS 模型理解。

按书中的简化计数，LHS 为 `result`、`a`、`b` 共 3 次；RHS 为 `add`、`a`、`a`、`b` 共 4 次。注意 `const` 的声明与初始化不能分开写，计数只是帮助理解读写方向，不应替代语言规范。

</details>

## 本章要点

1. JavaScript 在执行前会分析源码并建立声明相关信息；运行时才进行具体的读取和写入。
2. LHS/RHS 是理解标识符写入与读取的教学模型：LHS 找绑定写值，RHS 取绑定的值。
3. 复合赋值和自增自减同时包含 RHS 与 LHS。
4. 作用域查找从当前作用域向外进行；内层同名绑定会遮蔽外层绑定。
5. `var` 是函数作用域，`let`、`const`、`class` 是块级作用域；后者在初始化前受 TDZ 约束。
6. `ReferenceError` 关注名字能否被解析或访问，`TypeError` 关注已得到的值能否执行目标操作。
7. LHS/RHS 只讨论标识符的作用域查询；`this`、对象属性和原型链有各自独立的规则。

## 延伸阅读

- [《你不知道的 JavaScript（上卷）》](https://github.com/getify/You-Dont-Know-JS)
- [MDN：词法作用域](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures#%E8%AF%8D%E6%B3%95%E4%BD%9C%E7%94%A8%E5%9F%9F)
- [MDN：let 与暂时性死区](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let#%E6%9A%82%E6%97%B6%E6%80%A7%E6%AD%BB%E5%8C%BA)
