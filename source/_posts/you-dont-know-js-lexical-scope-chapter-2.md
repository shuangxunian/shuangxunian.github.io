---
title: "《你不知道的 JavaScript（上卷）》第二章：词法作用域"
excerpt: "从代码的书写位置理解词法作用域、作用域链和遮蔽；厘清 eval、with、new Function 与字符串计时器的真实边界。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 作用域, 词法作用域, 前端基础]
---

> 本文承接[第一章：作用域是什么](/2026/08/07/you-dont-know-js-scope-chapter-1/)。本章讨论的是标识符的词法作用域，不是 `this` 的动态绑定，也不是对象的原型链查找。

## 一句话理解词法作用域

**函数或块能访问哪些标识符，主要由它在源码中的书写位置决定，而不是由它在何处被调用决定。** 这套规则称为词法作用域（lexical scope），也常被称为静态作用域。

“词法”指的是源码被分析时的结构。函数定义在哪里，它的外层作用域是谁，通常在代码写下时已经确定。调用函数只会创建新的执行过程，不会把调用者所在的局部变量变成被调用函数的词法外层变量。

```js
const label = "global";

function showLabel() {
  console.log(label);
}

function run() {
  const label = "run";
  showLabel(); // "global"
}

run();
```

`showLabel` 定义在全局作用域，因此它读取的是全局 `label`。即使 `run` 调用它，`run` 内的 `label` 也不会参与 `showLabel` 的词法查找。

## 从一个例子看作用域链

```js
function foo(a) {
  const b = a * 2;

  function bar(c) {
    console.log(a, b, c);
  }

  bar(b * 3);
}

foo(2); // 2 4 12
```

可以把这段源码看成三个嵌套的作用域：

```text
全局作用域
└── foo 作用域：a、b、bar
    └── bar 作用域：c
```

在 `bar` 中访问标识符时，查找顺序如下：

| 标识符 | 查找路径 | 结果 |
| --- | --- | --- |
| `c` | `bar` | 找到形参 `c` |
| `b` | `bar` -> `foo` | 找到 `foo` 内的 `b` |
| `a` | `bar` -> `foo` | 找到 `foo` 的形参 `a` |

查找只会从当前作用域向外进行，不能向内层、同级或调用者的局部作用域查找。若一直到全局作用域仍未找到，读取通常会抛出 `ReferenceError`。

## 遮蔽：同名时选择最近的绑定

内层作用域中声明同名标识符，会让该标识符在内层优先解析到新绑定。这叫**遮蔽**（shadowing）。

```js
const theme = "light";

function preview() {
  const theme = "dark";
  return theme;
}

preview(); // "dark"
```

在 `preview` 内，直接写 `theme` 只能得到内层的 `"dark"`。对普通外层局部变量，没有语法可以绕过遮蔽、直接按名字访问被遮蔽的绑定；更好的设计是避免无意义的同名，或把需要共享的值通过参数、返回值或对象引用显式传递。

### 全局变量不是总能通过 `window` 访问

浏览器中的传统脚本里，顶层 `var` 声明往往会成为 `window` 的属性，因此有时能看到 `window.someName`。但这不是通用的“绕过作用域”技巧：

- ES 模块的顶层声明不挂到 `window`；
- 顶层 `let`、`const`、`class` 也不会成为全局对象属性；
- 非浏览器环境根本没有 `window`。

若确实需要访问全局对象，应使用跨环境名称 `globalThis`，并把这种依赖限制在初始化或平台适配层。不要用全局对象解决遮蔽问题。

## 块级作用域让边界更精确

函数体会创建函数作用域；`let`、`const` 和 `class` 在块中创建块级绑定。`catch` 的异常参数也只在对应的 `catch` 块中可见。`var` 则不具备块级作用域：它仍归属最近的函数作用域或全局作用域。

```js
if (true) {
  var legacy = "visible outside";
  const scoped = "only inside";
}

console.log(legacy); // "visible outside"
console.log(scoped); // ReferenceError
```

因此，现代代码通常优先使用 `const`，只有确实需要重新赋值时才使用 `let`。这不是风格偏好而已，它能把变量的可见范围缩小到真正需要的位置。

ES 模块还会形成自己的顶层作用域，并天然采用严格模式。模块中的顶层变量既不会自动成为 `globalThis` 属性，也不会泄漏到其他模块。

## `eval`：运行字符串代码，但不要依赖它改变作用域

`eval` 会把字符串当作 JavaScript 代码执行。直接调用的非严格 `eval` 有历史遗留行为：其中的 `var` 声明可以进入调用者的变量环境。

```js
function readValue(source) {
  eval(source);
  return value;
}

readValue("var value = 3;"); // 3
```

这段代码难以静态理解：只看函数体，无法知道 `value` 是否会出现。严格模式下，直接 `eval` 有自己的变量环境，`var value` 不会泄漏到 `readValue`；不过它仍可读取或修改外层已有的可访问绑定。

```js
function updateValue(source) {
  "use strict";
  let value = 1;
  eval(source);
  return value;
}

updateValue("value = 3;"); // 3
```

更重要的是安全性：把外部或可变输入传给 `eval` 相当于在当前权限下执行任意代码。业务代码不应使用它。需要处理数据时使用 `JSON.parse`；需要表达式或规则时，定义受限语法并编写解析器，或选择经过审计的专用库。

### `new Function` 和字符串计时器不是同一回事

它们同样会执行字符串代码，因此都应避免，但不等价于“把代码插入当前词法作用域”。

```js
function createReader() {
  const secret = "inside";
  return new Function("return typeof secret;");
}

createReader()(); // "undefined"
```

`Function` 构造器创建的函数不会捕获创建位置的局部作用域，它的外层环境是全局环境。浏览器中的 `setTimeout("...")` / `setInterval("...")` 也会在全局环境中执行字符串，而不是捕获当前函数的局部变量。请始终传递函数：

```js
setTimeout(() => render(), 100);
```

## `with`：已废弃的动态作用域干扰源

`with` 会把一个对象临时加入标识符解析环境：对象上存在的属性可以像局部标识符一样被解析。

```js
function updateCount(target) {
  with (target) {
    count += 1;
  }
}

const state = { count: 2 };
updateCount(state);
console.log(state.count); // 3
```

这会使代码含义依赖运行时对象是否具有某个属性，静态分析和重构都变得不可靠。对象也可以通过 `Symbol.unscopables` 排除部分属性，使判断更不直观。

若 `with` 块中赋值的名字既不在对象上、也不在任何外层作用域中，非严格模式下可能创建隐式全局属性；若外层已有同名绑定，写入的则可能是那个绑定。因此它不是“对象缺少属性就必然创建全局变量”，但两种结果都不应依赖。严格模式和 ES 模块会在解析阶段禁止 `with`，使用它会产生 `SyntaxError`。

```js
function updateCountSafely(target) {
  target.count += 1;
}
```

显式属性访问不仅可读，也便于 TypeScript、lint 和压缩工具检查。

## 性能：可预测性比微优化更重要

词法作用域让引擎能在编译和运行时采用多种优化策略，但不能简单地说只要出现 `eval` 或 `with`，引擎就会“放弃全部优化”或每次都完整遍历作用域链。具体策略、退优化条件和收益都由引擎版本及运行时数据决定。

实际结论依然明确：`eval`、`with` 和字符串代码执行会破坏静态可分析性，增加安全审计与维护成本，也可能妨碍优化。禁止它们的首要理由是**安全性与可维护性**；性能是额外收益。内容安全策略（CSP）通常也会限制 `eval` 一类动态执行。项目中可启用 ESLint 的 `no-eval`、`no-with` 与 `no-implied-eval` 规则，在提交前阻止这些模式。对真正的性能问题，应以真实场景的性能分析结果为准。

## 练习：调用位置不会改变词法作用域

```js
const message = "outer";

function printMessage() {
  console.log(message);
}

function caller() {
  const message = "caller";
  printMessage();
}

caller();
```

<details>
<summary>查看解析</summary>

输出是 `"outer"`。`printMessage` 在全局作用域定义，它的外层作用域是全局作用域，调用者 `caller` 不在它的作用域链上。

若要输出 `"caller"`，应把值明确作为参数传入：

```js
function printMessage(value) {
  console.log(value);
}

function caller() {
  const message = "caller";
  printMessage(message);
}
```

</details>

## 本章要点

1. JavaScript 的标识符作用域以词法作用域为主：函数或块的书写嵌套关系决定查找路径。
2. 查找从当前作用域向外，最近的同名绑定优先，形成遮蔽。
3. 函数在哪里调用不会改变它定义时的词法外层；这正是闭包能够可靠访问定义处变量的基础。
4. `let`、`const`、`class` 与 `catch` 参数提供块级绑定，`var` 只有函数或全局作用域；ES 模块还有独立的顶层作用域。
5. `eval`、`new Function`、字符串计时器都会引入动态代码执行风险；只有非严格的直接 `eval` 具有向调用者变量环境添加 `var` 的遗留行为。
6. `with` 会动态干扰名称解析，严格模式已禁止它；使用显式对象属性访问替代。
7. 不要以 `window` 或性能传闻为由保留这些特性。选择可预测、可分析的代码结构。

## 延伸阅读

- [《你不知道的 JavaScript》：Scope & Closures](https://github.com/getify/You-Dont-Know-JS)
- [MDN：闭包与词法环境](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures)
- [MDN：eval()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval)
- [MDN：with](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)
