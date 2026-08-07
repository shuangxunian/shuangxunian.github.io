---
title: "《你不知道的 JavaScript（上卷）》第二部分第二章：this 全面解析"
excerpt: "从调用位置判断普通函数的默认、隐式、显式与 new 绑定，厘清 bind、null、间接引用、箭头函数和严格模式的边界。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, this, 箭头函数, 面试, 前端基础]
---

> 本文承接[第二部分第一章：关于 this](/2026/08/07/you-dont-know-js-this-introduction-part-2-chapter-1/)。分析 `this` 时，先确认当前执行的是普通函数还是箭头函数；四大绑定规则只适用于普通函数。

## 从调用位置开始，而不是从定义位置猜测

普通函数的 `this` 在调用时根据调用表达式建立。所谓调用位置（call site），就是当前函数调用表达式出现的位置，而不是函数被声明、赋值或传递的位置。

```js
function logName() {
  console.log(this.name);
}

const user = { name: "Ada", logName };
user.logName();
```

这里的调用位置是 `user.logName()`，因此需要从这个表达式判断 `this`。复杂问题可以在目标函数内部写入 `debugger;`，再通过浏览器开发者工具的调用栈确认是谁、以什么形式调用了它。调用栈能帮助定位调用位置，但 `this` 的规则本身由当前调用表达式决定。

## 默认绑定：独立调用普通函数

当普通函数以 `fn()` 形式独立调用，没有其他绑定规则介入时，使用默认绑定。严格模式与非严格函数代码的行为不同：

```js
function strictThis() {
  "use strict";
  return this;
}

console.log(strictThis()); // undefined
```

```js
function sloppyThis() {
  return this;
}

// 在传统非严格脚本环境中，通常为 globalThis
console.log(sloppyThis() === globalThis);
```

严格性取决于**被调用函数自身所属的代码**：函数体中的 `"use strict"`、外层严格代码或 ES 模块都会让该函数以严格模式运行。仅仅从严格调用者调用一个非严格函数，不会反过来改变该函数的模式。

因此，不要依赖非严格默认绑定访问浏览器全局变量。ES 模块顶层 `this` 为 `undefined`，Node.js 等环境也有不同的顶层包装行为。需要全局对象时明确使用 `globalThis`，需要状态时明确传参或使用对象方法。

## 隐式绑定：通过对象属性调用

当调用表达式的函数引用前有对象属性访问时，普通函数的 `this` 会隐式绑定为该属性访问的基对象：

```js
function showValue() {
  "use strict";
  return this.value;
}

const record = { value: 2, showValue };
console.log(record.showValue()); // 2
```

对于属性链，取最后一次属性访问的基对象：

```js
const outer = {
  value: "outer",
  inner: {
    value: "inner",
    showValue,
  },
};

console.log(outer.inner.showValue()); // "inner"
```

函数并没有被永久绑定到 `record` 或 `inner`。属性访问只影响这一次普通函数调用。

### 隐式绑定丢失

把方法提取为普通函数值后，调用表达式不再包含原对象，隐式绑定自然消失：

```js
const detached = record.showValue;
console.log(detached()); // TypeError：严格模式下 this 是 undefined
```

作为回调传递也是同类问题。不要假设 `setTimeout(record.showValue, 100)` 会以 `record` 为 `this` 调用；回调的接收者由宿主 API 决定，通常不是原对象。要保留对象接收者，可使用包装函数或 `bind`：

```js
setTimeout(() => record.showValue(), 100);
setTimeout(record.showValue.bind(record), 100);
```

## 显式绑定：`call`、`apply` 与 `bind`

`call` 和 `apply` 可以为**本次调用**直接指定普通函数的 `this`：

```js
function describe(prefix, suffix) {
  return `${prefix}${this.name}${suffix}`;
}

const author = { name: "Ada" };

console.log(describe.call(author, "<", ">"));
console.log(describe.apply(author, ["[", "]"]));
```

`call` 接收逐个参数，`apply` 接收类数组对象。现代代码中，展开语法通常更易读，并可用于可迭代对象：`describe.call(author, ...parts)`。

### 原始值与 `null` / `undefined`

显式绑定的细节同样受函数是否严格影响：

```js
function strictType() {
  "use strict";
  return typeof this;
}

function sloppyType() {
  return typeof this;
}

console.log(strictType.call(7)); // "number"
console.log(sloppyType.call(7)); // "object"，非严格模式会装箱
console.log(strictType.call(null)); // "object"，this 就是 null
```

在非严格函数中，`call(null)` 或 `call(undefined)` 通常会把 `this` 替换为全局对象；严格函数中则保留 `null` 或 `undefined`。因此，“传入 `null` 一定退化为默认绑定”并不成立。

如果必须在非严格遗留代码中调用一个会写入 `this` 的函数，又不希望污染全局对象，可显式传入一个隔离对象，例如 `Object.create(null)`。现代代码更应避免依赖这种模式，改用严格模式、模块和清晰的参数设计。

### 硬绑定：`bind`

`bind` 不会立即调用原函数，而是返回一个新的绑定函数。它固定普通调用时的 `this`，并可预设部分参数：

```js
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const reader = { name: "Reader" };
const greetReader = greet.bind(reader, "Hello");

console.log(greetReader("!")); // "Hello, Reader!"
console.log(greetReader.call({ name: "Ignored" }, "?")); // "Hello, Reader?"
```

后一次 `call` 不能重写绑定函数的 `this`。但有一个重要例外：若被绑定的目标函数可构造，使用 `new` 调用绑定函数时，`new` 创建的新实例会作为原函数的 `this`，预绑定的 `this` 会被忽略，预设参数仍会保留。

## `new` 绑定：构造调用优先

`new` 不是“调用某个类”，而是对可构造函数的一种调用形式。大致会发生以下步骤：

1. 创建一个新对象，并将其原型关联到构造函数的 `prototype`；
2. 用新对象作为函数执行时的 `this`；
3. 执行构造函数体；
4. 若构造函数返回对象或函数，则使用这个显式返回值；否则返回新对象。

```js
function User(name) {
  this.name = name;
}

const ada = new User("Ada");
console.log(ada.name); // "Ada"
```

`new` 调用不能同时让 `call` 或 `apply` 指定一个既有对象作为构造函数的 `this`，因为 `new` 会创建并使用自己的新对象。箭头函数没有构造能力，`new (() => {})` 会抛出 `TypeError`。

绑定函数的构造行为展示了 `new` 的优先级：

```js
function Person(name) {
  this.name = name;
}

const BoundPerson = Person.bind({ name: "ignored" }, "Ada");
const person = new BoundPerson();

console.log(person.name); // "Ada"
```

## 优先级：解决多个规则可能同时出现的情况

常用判断顺序如下：

1. 是否为箭头函数？若是，使用创建位置的词法 `this`，四大规则不适用。
2. 是否通过 `new` 调用可构造函数或绑定函数？若是，使用 `new` 绑定。
3. 是否为已绑定函数，或通过 `call` / `apply` 显式指定 `this`？若是，使用显式绑定。
4. 是否以 `obj.fn()` 形式通过属性调用？若是，使用隐式绑定。
5. 否则使用默认绑定，并根据函数严格性确定 `undefined` 或全局对象等结果。

教学中常写为 `new` > 显式绑定 > 隐式绑定 > 默认绑定。`bind` 属于显式绑定的持久形式；当 `new` 与 `bind` 同时出现时，`new` 胜出。不要把 `call`、`apply`、`bind` 当作三个总能相互竞争的独立优先级层级。

## 间接引用与软绑定

赋值表达式会产生函数值而非属性引用，因此下例不保留 `source` 的隐式绑定：

```js
function show() {
  "use strict";
  return this && this.value;
}

const source = { value: 10, show };
const target = { value: 20 };

console.log((target.show = source.show)()); // undefined
```

赋值结果是 `show` 函数值，外层 `()` 是独立调用；`target` 不会成为 `this`。需要强调的是，这不是一种神秘的“绑定例外”，而是调用表达式已经不再是属性调用。

书中介绍的 softBind（软绑定）是一个历史辅助模式：在默认绑定时提供一个兜底对象，隐式或显式调用时仍允许其他接收者生效。JavaScript 没有原生 `softBind` API。现代项目通常以包装函数、默认参数或明确的对象 API 表达这种需求，避免修改 `Function.prototype`。

## 箭头函数：没有自身 `this`

箭头函数在创建时从外层词法环境取得 `this`，不参与默认、隐式、显式或 `new` 绑定：

```js
function createLogger() {
  setTimeout(() => {
    console.log(this.name);
  }, 100);
}

createLogger.call({ name: "Ada" }); // "Ada"
```

`call` 在这里绑定的是外层普通函数 `createLogger`；箭头回调读取它词法捕获的 `this`。对箭头函数本身使用 `call`、`apply` 或 `bind` 不能改变这个值；它也没有自己的 `arguments`，不能用作构造函数。

箭头函数适合需要保留外层 `this` 的回调。若函数应由调用者、实例或宿主 API 提供 `this`，则使用普通函数或方法简写。两种函数形式可以在同一项目中并存，关键是每处都让接收者语义明确。

## 练习：按优先级判断

```js
function Label(name) {
  this.name = name;
}

const BoundLabel = Label.bind({ name: "ignored" }, "Ada");
const label = new BoundLabel();

console.log(label.name);
```

<details>
<summary>查看解析</summary>

输出 `"Ada"`。`BoundLabel` 是绑定函数，预设参数 `"Ada"` 会传入 `Label`；但它通过 `new` 调用，因此 `new` 创建的新对象成为 `Label` 内的 `this`，并覆盖预绑定的 `{ name: "ignored" }`。

</details>

## 本章要点

1. 对普通函数，先定位调用表达式，再判断 `this`；不要从函数定义位置推断。
2. 默认绑定在严格函数中为 `undefined`，非严格传统脚本中通常会指向全局对象。
3. `obj.fn()` 使用隐式绑定；提取方法或传递回调会改变调用形式，可能失去原对象。
4. `call`/`apply` 只影响一次调用，`bind` 返回带预设 `this` 与参数的新函数。
5. 严格函数保留显式传入的原始值、`null` 和 `undefined`；非严格函数才可能装箱或替换为全局对象。
6. `new` 会创建构造调用所需的 `this`，并优先于绑定函数预设的 `this`。
7. 箭头函数没有自己的 `this`，应按词法外层而非四大绑定规则分析。

## 延伸阅读

- [MDN：this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
- [MDN：Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN：new 运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
- [MDN：严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)
