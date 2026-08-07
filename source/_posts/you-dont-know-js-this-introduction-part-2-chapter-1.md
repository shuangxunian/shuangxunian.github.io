---
title: "《你不知道的 JavaScript（上卷）》第二部分第一章：关于 this"
excerpt: "建立 this 的正确心智模型：它不是函数自身，也不是词法作用域；普通函数的 this 由调用形式决定，箭头函数则是例外。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, this, 对象, 前端基础]
---

> 第一部分“作用域与闭包”已结束。本章进入第二部分“`this` 和对象原型”，先建立一个前提：`this` 与词法作用域是两套独立机制。后续章节会系统分析具体绑定规则与对象原型。

## `this` 解决的是什么问题

`this` 让同一个函数可以根据调用时提供的接收者（receiver）操作不同对象，而不需要为每个对象复制一份函数。

```js
function identify() {
  return this.name.toUpperCase();
}

const author = { name: "Kyle" };
const reader = { name: "Reader" };

console.log(identify.call(author)); // "KYLE"
console.log(identify.call(reader)); // "READER"
```

`identify` 是同一个函数值；`call()` 为这两次调用显式指定了不同的 `this` 值。把函数写成对象方法时，也可以由调用形式提供接收者：

```js
const user = {
  name: "Ada",
  identify,
};

console.log(user.identify()); // "ADA"
```

这是一种表达“由谁来执行这个行为”的方式。它并不总比显式参数更好：无状态工具函数、数据转换和跨对象计算通常使用参数更直观。

```js
function identifyByName(user) {
  return user.name.toUpperCase();
}

identifyByName(author); // "KYLE"
```

选择标准不是代码是否更短，而是调用方是否确实是行为的自然接收者，以及函数被提取、组合或异步传递后是否仍能保持清晰。

## 误区一：`this` 不是函数自身

函数也是对象，可以拥有属性，但普通调用不会自动让 `this` 指向当前函数：

```js
function trackCall() {
  trackCall.count += 1;
}

trackCall.count = 0;
trackCall();
console.log(trackCall.count); // 1
```

这里能计数，是因为函数体通过词法标识符 `trackCall` 访问了函数对象属性；不是因为 `this` 指向 `trackCall`。下面这种写法不可靠：

```js
function trackCall() {
  this.count += 1;
}
```

当以 `trackCall()` 普通调用时，严格模式中的 `this` 为 `undefined`，访问 `this.count` 会抛出 `TypeError`；非严格传统脚本中它通常指向全局对象，反而可能污染全局状态。

也不要使用已废弃、严格模式中禁止的 `arguments.callee` 来引用当前函数。若状态确实属于函数的某个实例或流程，通常应使用闭包、对象字段或 `class` 私有字段表达所有权：

```js
function createTracker() {
  let count = 0;

  return () => ++count;
}
```

## 误区二：`this` 不是函数的词法作用域

词法作用域通过标识符名称查找，取决于代码写在哪里；`this` 是函数调用时得到的特殊绑定。两者不能互相替代。

```js
function readValue() {
  const value = 2;
  return this.value;
}

console.log(readValue.call({ value: 99 })); // 99
```

`this.value` 是读取调用者提供对象的属性，和局部变量 `value` 没有关系。若要读取局部变量，直接写 `value`：

```js
function readLocalValue() {
  const value = 2;
  return value;
}
```

同样，嵌套函数也不会因为写在外层函数中，就自动把 `this` 指向外层调用的 `this`。普通嵌套函数有自己的调用方式；箭头函数才会词法地继承外层 `this`。

## 函数不“属于”对象

下面的对象字面量把一个函数值放在 `user.identify` 属性中，但函数本身并没有永久绑定到 `user`：

```js
const user = {
  name: "Ada",
  identify() {
    return this.name;
  },
};

const detachedIdentify = user.identify;
console.log(detachedIdentify.call({ name: "Grace" })); // "Grace"
```

`user.identify()` 的调用形式会让普通函数的 `this` 指向 `user`；一旦提取为 `detachedIdentify`，再怎样调用它就决定了新的 `this`。这解释了回调中常见的 “`this` 丢失” 问题：不是 `this` 消失，而是调用形式变化了。

## 普通函数的 `this` 取决于调用形式

对普通函数而言，`this` 绑定在每次调用时确定。下一章会逐一展开，先记住这张导航表：

| 调用形式 | 普通函数的典型 `this` |
| --- | --- |
| `fn()` | 严格模式为 `undefined`；非严格传统脚本通常为全局对象 |
| `obj.fn()` | `obj` |
| `fn.call(context)` / `fn.apply(context)` | `context` |
| `new Fn()` | 新创建的实例对象（除非构造函数显式返回其他对象） |
| 已用 `fn.bind(context)` 创建的函数 | 绑定时指定的 `context` |

表中的“典型”很重要：可选链调用、代理、宿主 API、严格模式和绑定函数都可能引入细节。分析时先看**函数如何被调用**，而不是看它在哪里声明、谁的属性名引用了它。

## 箭头函数是词法 `this` 的例外

箭头函数没有自己的 `this`。它从创建位置的外层作用域取得 `this`，之后不能由 `call`、`apply` 或 `bind` 改写。

```js
function createLogger() {
  return () => this.name;
}

const logger = createLogger.call({ name: "Ada" });
console.log(logger.call({ name: "Grace" })); // "Ada"
```

这里 `createLogger` 通过 `call` 得到 `{ name: "Ada" }`；返回的箭头函数词法捕获该 `this`。第二次 `call` 不会改变箭头函数的结果。

箭头函数很适合需要保留外层 `this` 的回调，但不适合需要调用者动态提供 `this` 的对象方法、原型方法或部分 DOM 事件处理器。把所有函数都改成箭头函数，和把所有函数都 `bind` 一样，会掩盖真实的接收者语义。

## 一个可重复的分析步骤

遇到 `this` 问题时，按下面顺序判断：

1. 当前执行的是普通函数、箭头函数，还是已绑定函数？
2. 对普通函数，调用表达式是什么：独立调用、属性调用、`call`/`apply`、`bind` 还是 `new`？
3. 函数是否被提取成回调，从而改变了调用形式？
4. 代码是否处于严格模式或 ES 模块中？这会影响默认绑定。

不要从函数“属于谁”、定义在哪个对象里，或外层有哪些局部变量开始猜测。

## 练习：提取方法后会怎样

```js
const profile = {
  name: "Ada",
  getName() {
    return this.name;
  },
};

const getName = profile.getName;

console.log(profile.getName());
console.log(getName.call({ name: "Grace" }));
```

<details>
<summary>查看解析</summary>

依次输出 `"Ada"` 和 `"Grace"`。

第一次是 `profile.getName()` 属性调用，普通函数的 `this` 为 `profile`。第二次通过 `call()` 显式指定了 `{ name: "Grace" }`。函数值相同，区别完全在调用形式。

</details>

## 本章要点

1. `this` 让同一个行为函数可以面向不同接收者复用，但显式参数在无状态场景常更清晰。
2. `this` 不是函数自身；函数属性和函数的 `this` 绑定是两件事。
3. `this` 不是词法作用域，不能用来读取局部变量或调用者的局部变量。
4. 普通函数的 `this` 通常由调用形式决定；函数从对象属性中被提取后，调用形式会改变。
5. 箭头函数没有自己的 `this`，会词法继承创建位置外层的 `this`。
6. 分析 `this` 时，先识别函数类型和调用表达式，再考虑严格模式等边界条件。

## 延伸阅读

- [MDN：this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
- [MDN：箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN：Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
