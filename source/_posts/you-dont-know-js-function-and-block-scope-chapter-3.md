---
title: "《你不知道的 JavaScript（上卷）》第三章：函数作用域与块作用域"
excerpt: "理解函数体、IIFE、var、let 与 const 如何划定变量可见范围，并将最小暴露原则应用到循环、模块和资源管理。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 作用域, IIFE, ES6, 前端基础]
---

> 本文承接[第一章：作用域是什么](/2026/08/07/you-dont-know-js-scope-chapter-1/)与[第二章：词法作用域](/2026/08/07/you-dont-know-js-lexical-scope-chapter-2/)。本章关注如何用函数与代码块主动缩小变量的可见范围。

## 最小暴露：作用域设计的目标

作用域不仅是语言规则，也是设计工具。一个标识符只应在真正需要它的代码中可见，这可以称为**最小暴露原则**：

- 降低名称冲突与意外覆盖的风险；
- 避免实现细节被外部代码直接依赖或修改；
- 让读代码的人更快判断变量可能在哪里被使用。

“私有”在 JavaScript 中有不同层次：闭包和模块可以隐藏模块内部状态，类的 `#private` 字段可以限制实例字段访问。函数/块作用域解决的是第一步问题：先把局部名字限制在最小合理范围。

## 函数体形成作用域边界

函数定义时会确定函数体的词法外层环境；每次调用函数时，函数体拥有自己的局部绑定。函数内的形参、局部声明和嵌套函数通常不能从函数外通过未限定名称访问。

```js
function doWork() {
  const innerValue = 10;
  return innerValue * 2;
}

doWork();
console.log(innerValue); // ReferenceError
```

`innerValue` 的可见范围是 `doWork` 的函数体。外部不需要知道它如何参与计算，只依赖 `doWork()` 的返回结果。这既隐藏了实现细节，也允许以后重构内部变量而不影响调用方。

函数作用域也隔离了同名局部变量：

```js
function formatTitle(title) {
  const prefix = "[title]";
  return `${prefix} ${title}`;
}

function formatWarning(message) {
  const prefix = "[warning]";
  return `${prefix} ${message}`;
}
```

两个 `prefix` 分属不同函数调用的局部环境，互不冲突。

## IIFE：模块出现前的隔离方式

IIFE 是 Immediately Invoked Function Expression（立即调用函数表达式）。它创建一个函数表达式，并立刻执行，以获得一次性的私有函数作用域。

```js
(function initialize() {
  const cacheKey = "article-cache";
  console.log(cacheKey);
})();
```

`initialize` 是**具名函数表达式**的内部名字，不会在外层作用域创建同名绑定。也可以省略名称：

```js
(function () {
  const cacheKey = "article-cache";
}());
```

两种调用写法都有效：`(function () {})()` 和 `(function () {}())`。第一种更常见；选择团队约定的一种即可。

### 声明与表达式的区别

下面这句是函数声明，`boot` 的绑定位于其所在的外层作用域：

```js
function boot() {}
```

而在表达式上下文中，函数可以不向外暴露名字。包上一层括号会使 `function` 按表达式解析，因此适合 IIFE：

```js
(function () {
  // 独立作用域
}());
```

IIFE 也可以显式传入依赖。在浏览器环境中，不要假设 `window` 总是存在；跨环境代码可使用 `globalThis`，或更好地直接传入所需能力。

```js
(function configure(storage) {
  storage.setItem("theme", "dark");
}(globalThis.localStorage));
```

上例仅适合确定存在 `localStorage` 的浏览器场景。库代码应先检查能力，或由调用者传入适配对象。

### 为什么偏向具名函数表达式

匿名函数并非天然不可调试，现代工具常能推断名称；但明确命名通常能让堆栈、性能分析和错误报告更清楚，也能在递归时稳定地引用函数自身。

```js
const factorial = function factorial(value) {
  return value <= 1 ? 1 : value * factorial(value - 1);
};
```

事件解绑依赖的是**同一个函数对象引用**，不是函数名。无论函数是否具名，都应该保存回调引用；不要使用已废弃且在严格模式中禁止的 `arguments.callee`。

### IIFE 的今天

IIFE 曾是浏览器脚本隔离变量、实现早期模块模式的主力。现代应用优先使用 ES 模块：模块天然拥有顶层作用域，并通过 `export` 明确公开 API。IIFE 仍适合一次性的隔离初始化，或维护不支持模块的旧脚本，但不必为了“私有变量”而在所有代码外包一层 IIFE。

## 块作用域：让变量只活在需要的代码块中

代码块通常由 `{}` 形成，例如 `if`、`for`、`while`、`switch` 的分支或单独的 `{}`。`let`、`const`、`class` 和 `catch` 的异常参数在这些位置创建块级绑定。

```js
if (response.ok) {
  const payload = readPayload(response);
  render(payload);
}

// payload 在这里不可见
```

### `var` 与块的关系

`var` 只有函数作用域或全局作用域，没有块级作用域：

```js
if (true) {
  var legacyValue = "outside too";
  const blockValue = "inside only";
}

console.log(legacyValue); // "outside too"
console.log(blockValue); // ReferenceError
```

因此，在新代码中默认使用 `const`；只有需要重新赋值时改用 `let`。`var` 仍是合法语法，在维护旧代码、理解函数提升或兼容遗留环境时需要认识它，但通常不应成为新增业务代码的首选。

### `let`：块级绑定与暂时性死区

`let` 的绑定在进入块作用域时就已建立，但初始化语句执行前处于暂时性死区（TDZ），不能读取或写入。它并非“完全没有提升”，而是不能像 `var` 一样在声明前得到 `undefined`。

```js
{
  console.log(status); // ReferenceError：TDZ
  let status = "ready";
}

console.log(status); // ReferenceError：块外不可见
```

### `const`：绑定不可重新赋值

`const` 同样是块级绑定，声明时必须初始化，之后不能给该绑定重新赋值：

```js
const settings = { theme: "light" };
settings.theme = "dark"; // 合法：修改对象内容
settings = {}; // TypeError：不能重新赋值给 settings
```

`const` 保护的是绑定关系，而不是对象、数组等引用值的深层内容。需要不可变数据时，应采用复制更新、冻结策略或合适的状态管理约束，而不是只依赖 `const`。

## 循环中的每轮绑定

`for` 循环头部的 `let`/`const` 具有每轮迭代绑定的语义。这使异步回调能够得到对应轮次的值：

```js
for (let index = 0; index < 3; index += 1) {
  setTimeout(() => console.log(index), 1000);
}

// 约 1 秒后依次输出：0、1、2
```

若改成 `var index`，三个回调共享同一个函数作用域中的 `index`，在计时器运行前循环已经结束，通常都会输出 `3`。以前常用 IIFE 为每轮参数创建一个局部副本；在现代 JavaScript 中，优先使用 `let`。

## 块、闭包与内存：不要许下过度承诺

缩小作用域会减少不必要的引用范围，常常有利于代码生命周期管理：当变量不再可达时，垃圾回收器才有机会回收它引用的对象。但“执行离开一个块”不等于对象一定立即释放。

```js
let readLater;

{
  const largeData = loadLargeData();
  readLater = () => largeData.length;
}
```

虽然块已经结束，`readLater` 的闭包仍引用 `largeData`，它就必须继续保留。实际内存回收时机由引擎决定，也受闭包、异步任务、DOM 引用和缓存等因素影响。块作用域是减少意外持有的好工具，但内存问题仍应通过堆快照等工具分析。

## `catch` 的作用域边界

`catch` 参数只在对应的异常处理块内有效，适合避免错误变量泄漏到后续逻辑：

```js
try {
  saveArticle();
} catch (error) {
  reportError(error);
}

// error 在这里不可见
```

现代 JavaScript 允许在不需要错误对象时省略绑定：

```js
try {
  refreshCache();
} catch {
  showOfflineState();
}
```

## 练习：选出正确输出

```js
const tasks = [];

for (let index = 0; index < 3; index += 1) {
  tasks.push(() => index);
}

console.log(tasks.map((task) => task()));
```

<details>
<summary>查看解析</summary>

输出为 `[0, 1, 2]`。循环的每次迭代都获得独立的 `index` 绑定，每个回调闭包捕获的是自己那一轮的绑定。

若将 `let index` 改成 `var index`，三个回调共享同一个绑定，循环结束后值为 `3`，输出会变为 `[3, 3, 3]`。

</details>

## 本章要点

1. 函数体建立局部作用域边界，适合隐藏实现细节并避免命名冲突。
2. IIFE 是模块出现前常用的隔离方案；现代项目优先采用 ES 模块的显式导入和导出。
3. 函数表达式通常可避免在外层创建函数名绑定；具名函数表达式能改善调试和递归可读性。
4. `var` 没有块级作用域；`let`、`const`、`class` 和 `catch` 参数具有块级绑定。
5. `let`/`const` 进入作用域后先经历 TDZ，不应表述为“没有提升”。
6. `const` 禁止重新赋值绑定，并不使对象内容自动不可变。
7. `for (let ...)` 为每轮迭代创建独立绑定，是解决循环异步回调问题的标准写法。
8. 缩小作用域有助于减少不必要引用，但对象能否回收取决于是否仍可达，而非单纯是否离开代码块。
9. 作用域是封装与减少误用的工具，不是安全边界；不可信代码不应被授予执行权限。

## 延伸阅读

- [MDN：函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/function)
- [MDN：let](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)
- [MDN：const](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)
- [MDN：闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures)
