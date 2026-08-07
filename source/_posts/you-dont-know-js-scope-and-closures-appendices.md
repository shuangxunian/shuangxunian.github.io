---
title: "《你不知道的 JavaScript（上卷）》附录：动态作用域、块作用域与箭头函数"
excerpt: "梳理作用域与闭包部分的四个附录：动态作用域的对照、旧环境的块作用域替代方案、箭头函数的词法 this，以及第一部分的现代实践总览。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 作用域, 闭包, 箭头函数, 前端基础]
---

> 本文为《你不知道的 JavaScript（上卷）》第一部分“作用域与闭包”的附录收束。书中部分工具和兼容方案产生于 ES6 普及之前，本文会标明其历史背景，并给出现代 JavaScript 的对应做法。

## 附录 A：动态作用域与词法作用域

### 两种查找依据

JavaScript 的普通标识符查找采用**词法作用域**：函数可以访问哪些外层绑定，由函数在源码中的嵌套位置决定，而不是由谁调用它决定。

动态作用域（dynamic scope）采用另一套规则：查找会沿当前调用栈向上，调用者的局部变量可能影响被调用函数的变量解析。一些语言或语言特性提供过这种机制，但 JavaScript 的普通变量查找不采用它。

```js
const value = 2;

function readValue() {
  console.log(value);
}

function caller() {
  const value = 3;
  readValue();
}

caller(); // 2
```

`readValue` 定义在全局作用域，其词法外层只包含全局环境，因此输出 `2`。若这是动态作用域语言，`readValue()` 可能沿调用栈找到 `caller` 中的 `value` 并输出 `3`。

### `this` 不等于动态作用域

`this` 的值在普通函数中通常由调用形式决定，表面上与动态作用域一样都和“调用位置”有关。但它不是标识符作用域链的一环，也不会让函数读取调用者的局部变量。

```js
function show() {
  console.log(this.name);
}

const user = { name: "Ada", show };
user.show(); // "Ada"
```

这里 `this` 由 `user.show()` 的调用形式确定；而 `show` 中的普通变量依然按其定义位置的词法作用域查找。把 `this` 叫作“动态作用域”会混淆两个独立的语言机制。

## 附录 B：块作用域的历史替代方案

在 `let`/`const` 成为通用能力以前，开发者确实使用过 `catch` 参数或 IIFE 来缩小标识符可见范围。这些写法今天主要用于阅读旧代码，而不是新增代码。

### `catch` 参数的局部可见性

`catch` 的异常绑定只在对应的 `catch` 块中可用：

```js
try {
  throw new Error("failed");
} catch (error) {
  console.log(error.message); // "failed"
}

console.log(error); // ReferenceError
```

早期代码有时会刻意 `throw` 一个值，只为借用这个局部绑定作为“块作用域”。这种技巧会把控制流和变量声明混在一起，不适合业务代码。现代代码直接使用 `let` 和 `const`。

### IIFE 不是块作用域的等价替身

IIFE 能创建函数作用域：

```js
(function () {
  var temporary = "inside";
  console.log(temporary);
}());

console.log(temporary); // ReferenceError
```

但函数边界与代码块边界并不相同。IIFE 会创建新的 `this`、`arguments` 和 `return` 边界，也不能让 `break`/`continue` 穿过函数边界去控制外部循环。因此它不能无差别替换块作用域。

### 旧工具与未标准化语法

Traceur 等早期转译器曾探索以辅助函数、闭包或其他代码生成策略实现 ES6 特性；不同版本和不同语法的输出并不相同，不能概括为“`let` 一定被编译成 `try/catch`”。

书中提到的 `let (x = 2) { ... }` 属于历史上的非标准/提案语法，没有进入现代 ECMAScript。不要在新代码中使用，也不需要引入专门工具支持它。对于需要兼容旧运行时的项目，应由当前构建工具根据目标浏览器生成适当的转换代码，并以真实兼容性测试为准。

## 附录 C：箭头函数与词法 `this`

箭头函数没有自己的 `this` 绑定。它会从创建位置的外层作用域取得 `this`，这通常被称为**词法 `this`**。

```js
function Timer() {
  this.seconds = 0;

  setInterval(() => {
    this.seconds += 1;
  }, 1000);
}
```

箭头回调中的 `this` 继承自 `Timer` 调用所创建的 `this`，不再需要过去常见的 `const self = this` 或 `function () {}.bind(this)` 写法。

### 箭头函数没有自己的调用者绑定

`call`、`apply`、`bind` 无法改变箭头函数已经捕获的 `this`。箭头函数也没有自己的 `arguments`、`super` 或 `new.target` 绑定，且不能作为构造函数使用：

```js
const readThis = () => this;

readThis.call({ name: "ignored" }); // 仍返回创建位置的 this
new readThis(); // TypeError
```

若需要访问外层函数的实参，箭头函数可以读取外层的 `arguments`，但更清晰的现代写法通常是使用剩余参数：

```js
const sum = (...numbers) => numbers.reduce((total, number) => total + number, 0);
```

### 什么时候不该使用箭头函数

箭头函数不是“更短的普通函数”。以下场景通常应使用普通函数或方法简写，因为它们需要动态接收者或构造能力：

- 对象方法需要由调用者决定 `this`；
- 原型方法需要使用实例 `this`；
- DOM 事件处理器需要框架/浏览器提供的处理器 `this`；
- 构造函数，或需要 `arguments` 的旧接口实现。

```js
const counter = {
  value: 0,
  increment() {
    this.value += 1;
  },
};

counter.increment();
console.log(counter.value); // 1
```

大型项目不应简单规定“全部使用箭头函数”或“全部使用 `bind`”。应根据函数是否需要自己的 `this` 选择形式，并让团队通过 lint 规则和代码审查保持一致。

## 附录 D：致谢

致谢部分记录作者对社区、读者、译者和开源贡献者的感谢，没有需要额外展开的技术概念。它也提醒我们：语言特性、浏览器实现与工程实践，始终由社区长期协作推进。

## 第一部分总纲：作用域与闭包

### 1. 词法作用域是根基

- 标识符的查找路径由代码书写时的嵌套关系确定，从当前作用域逐层向外查找。
- 内层同名绑定会遮蔽外层绑定；`this`、对象属性和原型链遵循各自的规则。
- `eval`、`with` 和字符串代码执行会破坏静态可分析性，并带来安全与维护风险，应避免使用。

### 2. 提升关注绑定的创建与初始化

- `var` 绑定在执行前已初始化为 `undefined`；函数声明在执行前已关联函数对象。
- `let`、`const`、`class` 绑定也会预先创建，但在初始化前处于暂时性死区（TDZ）。
- 与其记忆“声明谁优先”，不如分析当前绑定属于哪个作用域、何时初始化、是否被后续赋值覆盖。

### 3. LHS/RHS 是读写方向的心智模型

- RHS 读取标识符当前的值，LHS 查找可写入的绑定。
- 未解析的读取通常产生 `ReferenceError`；取得值后却执行非法操作通常产生 `TypeError`。
- 严格模式与 ES 模块会阻止未声明赋值形成隐式全局变量。

### 4. 闭包让状态跨越执行时间

- 函数可在定义位置的执行结束后继续访问外层绑定。
- 回调、异步任务、节流、防抖、记忆化缓存和模块封装都依赖这一能力。
- `for (let ...)` 的每轮独立绑定解决了 `var` 循环回调共享单一变量的问题。

### 5. 用现代模块划分边界

- ES 模块提供文件级作用域、静态依赖关系和明确的公开 API，是当前模块化首选。
- 经典 IIFE/工厂模块仍有助于理解闭包，也可用于维护遗留脚本。
- 作用域封装不是安全沙箱；API 设计、输入校验和权限控制仍不可替代。

## 工程实践清单

1. 新代码默认 `const`，仅在需要重新赋值时使用 `let`；避免新的 `var`。
2. 声明靠近首次使用处，保持变量生命周期与可见范围尽量小。
3. 禁用 `eval`、`with` 和字符串形式的计时器；通过 ESLint 的 `no-eval`、`no-with`、`no-implied-eval` 等规则持续检查。
4. 使用 ES 模块并明确 `export` API；不要用全局变量或 IIFE 代替清晰的模块边界。
5. 回调中需要外层 `this` 时使用箭头函数；需要动态 `this` 时使用普通函数或方法简写。
6. 对长期事件监听、计时器、订阅和缓存设计显式清理路径，避免闭包意外保留不再需要的数据。

## 延伸阅读

- [MDN：闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures)
- [MDN：箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN：JavaScript 模块](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)
- [MDN：严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
