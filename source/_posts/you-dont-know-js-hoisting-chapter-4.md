---
title: "《你不知道的 JavaScript（上卷）》第四章：提升"
excerpt: "用“绑定创建与初始化时机”替代代码搬移的比喻，准确理解 var、函数声明、函数表达式、let、const 与块内函数声明。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 作用域, 提升, ES6, 前端基础]
---

> 本文承接前三章的作用域模型。提升（hoisting）是对“代码执行前已建立某些绑定”现象的通俗称呼，并不表示引擎真的把源码逐行移动到文件顶部。

## 提升的准确模型：先创建绑定，再执行语句

JavaScript 执行一个作用域中的语句前，会先处理其中的声明并创建相应绑定。不同声明的关键差异不在于“是否提升”，而在于**绑定何时创建、何时初始化、初始化为什么值**。

| 声明形式 | 绑定创建时机 | 声明语句前的状态 |
| --- | --- | --- |
| `var` | 进入函数或全局作用域时 | 已初始化为 `undefined` |
| 函数声明 | 进入所在作用域时 | 已初始化为函数对象 |
| `let` / `const` / `class` | 进入块作用域时 | 已创建但未初始化，处于 TDZ |

“把声明提到顶部、把赋值留在原地”是理解 `var` 的有用近似，但不要把它当作规范层面的真实源码改写。更准确的说法是：**运行语句前，绑定已经存在；赋值和其他表达式仍按原有顺序执行。**

## `var`：先得到 `undefined`，再执行赋值

```js
console.log(total); // undefined
var total = 2;
```

可以将它用于理解的等价过程写成：

```js
var total;
console.log(total);
total = 2;
```

这里没有 `ReferenceError`，因为 `total` 绑定在执行 `console.log` 前已经存在，并被初始化为 `undefined`。真正容易出错的是把 `undefined` 当作“变量已经有了正确值”。

### 重复 `var` 不会创建第二个绑定

```js
var count = 10;
var count = 20;

console.log(count); // 20
```

`var` 只依附于最近的函数作用域或传统脚本的全局作用域，不依附于普通的 `{}` 块：

```js
if (true) {
  var visibleOutsideBlock = "var";
}

console.log(visibleOutsideBlock); // "var"
```

这与 `let`/`const` 的块级绑定不同。浏览器传统脚本的顶层 `var` 还可能成为全局对象属性，而 ES 模块不会；不要将两者混为一谈。

同一作用域里只有一个 `count` 绑定。第二个 `var count` 不会新建变量，但 `= 20` 仍是一次正常的运行时赋值。这也是重复 `var` 容易隐藏覆盖问题的原因。

## 函数声明：函数对象在语句执行前可用

函数声明的名称和函数对象在作用域初始化时就已关联，因此可以在声明语句前调用：

```js
announce(); // "ready"

function announce() {
  console.log("ready");
}
```

这项能力是语言行为，不是要求把函数都写在文件末尾。为了减少读者依赖隐式时序，业务代码通常仍会让声明与使用保持合理接近，并避免跨很长的作用域依赖提升。

## 函数表达式：只有承载它的变量按自身规则初始化

函数表达式是在赋值表达式执行时才创建函数对象。若使用 `var` 承载它，变量会先是 `undefined`，因此提前调用得到 `TypeError`：

```js
runTask(); // TypeError: runTask is not a function

var runTask = function () {
  console.log("running");
};
```

如果换为 `const` 或 `let`，提前访问甚至发生在绑定初始化之前，会得到 `ReferenceError`：

```js
runTask(); // ReferenceError：TDZ

const runTask = () => {
  console.log("running");
};
```

所以“函数表达式不会提升”是不够精确的速记。应当说：**函数对象不会在赋值前创建；变量绑定的可访问性由 `var`、`let` 或 `const` 决定。**

## 同名 `function` 与 `var`：声明不覆盖函数，赋值会

在同一个函数或传统全局 `var` 作用域中，函数声明和同名 `var` 声明会复用同一个绑定。初始化阶段，函数声明提供的函数对象可用；单独的 `var name;` 不会把它改回 `undefined`。但后续赋值会覆盖该值：

```js
announce(); // "function value"

var announce = "later value";

function announce() {
  console.log("function value");
}

announce(); // TypeError: announce is not a function
```

用“函数声明优先于 `var`”可以帮助解释这个经典例子，但它不是鼓励依赖声明排序的编程方式。重复函数声明、块内函数声明，以及不同执行环境中的遗留 Web 兼容规则都会增加复杂度。最稳妥的规则是：**不要让函数和变量在同一作用域使用同一个名字。**

## 块内函数声明：有标准语义，也有遗留差异

“不要在块中声明函数”曾是为了规避早期浏览器兼容性问题的常见建议。现代 JavaScript 已定义了块内函数声明的词法作用域行为；在严格模式和 ES 模块中，它是块级绑定：

```js
"use strict";

if (true) {
  function onlyInside() {
    return "inside";
  }

  console.log(onlyInside()); // "inside"
}

console.log(onlyInside); // ReferenceError
```

不过非严格传统脚本仍可能应用 Annex B 的 Web 兼容语义，使块内函数额外表现得像外层 `var` 绑定。为了让代码在模块、严格模式和旧脚本中都容易理解，条件函数更推荐写成显式赋值：

```js
const formatter = condition
  ? (value) => value.trim()
  : (value) => value;
```

## `let`、`const` 与 TDZ：绑定已存在，但尚不可访问

`let`、`const` 和 `class` 会在进入其块作用域时创建绑定，但初始化语句执行前处于**暂时性死区**（Temporal Dead Zone，TDZ）。这就是它们在声明前访问时抛出 `ReferenceError` 的原因。

```js
{
  console.log(status); // ReferenceError：status 处于 TDZ
  let status = "ready";
}
```

TDZ 从进入当前作用域开始，而非只从声明所在行开始。内层声明会遮蔽外层同名变量，即使内层尚未初始化：

```js
const mode = "outer";

{
  console.log(mode); // ReferenceError：内层 mode 处于 TDZ
  const mode = "inner";
}
```

`typeof` 对完全未声明的标识符是个特例，但它不能绕过 TDZ：

```js
typeof missingName; // "undefined"

{
  typeof ready; // ReferenceError：ready 处于 TDZ
  const ready = true;
}
```

`class` 也使用同样的未初始化语义：在类声明语句之前访问该类名会抛出 `ReferenceError`。

因此，`let`/`const` 不是“完全没有提升”，而是“预先创建、延迟初始化、初始化前禁止访问”。这比 `var` 的 `undefined` 更早暴露了声明顺序或遮蔽错误。

## 读懂代码：关注初始化时机，而不是背诵优先级

分析提升问题时，可以按以下顺序判断：

1. 这个名字由哪种声明创建，属于哪个作用域？
2. 当前语句执行前，它是函数对象、`undefined`，还是 TDZ 中未初始化的绑定？
3. 是否有后续赋值、遮蔽或同名声明改变了当前值？

例如：

```js
console.log(value);

var value = 1;

function value() {}
```

输出是函数对象。函数声明先为同名绑定提供初始值，随后才执行 `var value = 1`；`console.log` 位于赋值之前。

## 练习：分别会发生什么

```js
console.log(first);
console.log(second);

var first = "var value";
let second = "let value";
```

<details>
<summary>查看解析</summary>

第一行输出 `undefined`：`first` 是 `var` 绑定，执行前已初始化为 `undefined`。

第二行抛出 `ReferenceError`：`second` 是 `let` 绑定，尚未执行初始化，仍处于 TDZ。异常会中断当前脚本，因此下面两条赋值语句不会执行。

</details>

## 开发实践

1. 新代码默认使用 `const`；确实需要重新赋值时使用 `let`；避免引入新的 `var`。
2. 声明应靠近首次使用处，并在声明时完成有意义的初始化，而不是机械地全部放在作用域顶部。
3. 不要依赖函数、`var` 与块内函数声明之间的同名和排序规则；使用唯一、语义明确的名称。
4. 对条件逻辑，使用条件表达式、明确赋值或分别定义的函数，而不是依赖非严格脚本中的块内函数遗留行为。
5. 启用 ESLint 的 `no-var`、`no-use-before-define` 和 `no-redeclare` 等规则，将容易误解的提升模式拦在提交前。

## 本章要点

1. 提升不是源码移动，而是执行语句前创建绑定和进行初始化的现象。
2. `var` 绑定预先初始化为 `undefined`；赋值仍在原语句位置执行。
3. 函数声明在作用域初始化时即可调用；函数表达式的函数对象只在赋值表达式执行时创建。
4. 同名的函数声明与 `var` 会共享绑定，`var` 声明本身不覆盖函数，赋值会覆盖。
5. `let`、`const`、`class` 绑定会预先创建，但在 TDZ 内不能访问。
6. 严格模式和模块中的块内函数是块级绑定；非严格旧脚本可能存在遗留 Web 兼容行为。
7. 写代码时优先让声明、初始化和使用顺序清晰，而不是依赖提升带来的偶然可用性。

## 延伸阅读

- [MDN：变量提升](https://developer.mozilla.org/zh-CN/docs/Glossary/Hoisting)
- [MDN：var](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var)
- [MDN：函数声明](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)
- [MDN：暂时性死区](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let#%E6%9A%82%E6%97%B6%E6%80%A7%E6%AD%BB%E5%8C%BA)
