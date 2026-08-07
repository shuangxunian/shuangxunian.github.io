---
title: "《你不知道的 JavaScript（上卷）》第五章：作用域闭包"
excerpt: "从函数定义时的词法环境理解闭包，掌握回调、循环、模块模式与内存生命周期，并区分经典模块模式和 ES 模块。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 闭包, 作用域, 模块化, 前端基础]
---

> 本文承接前四章的词法作用域与提升模型。闭包不是一个额外的语法关键字，而是函数与其定义位置的词法环境共同产生的能力。

## 闭包是什么

可以把闭包理解为：**函数创建时会关联其词法环境，因此函数在之后执行时，仍能访问定义位置可见的外层绑定。**

当内部函数被返回、传给回调或存入其他长期存活的位置，并在外层函数返回后仍读取外层变量时，这个特性最直观。函数“记住”的不是某个变量值的快照，而是对外层**绑定**的访问能力；若该绑定后来被修改，闭包会读到更新后的值。

```js
function createReader() {
  let value = 2;

  return function readValue() {
    return value;
  };
}

const read = createReader();
console.log(read()); // 2
```

`createReader()` 已经返回，但 `readValue` 仍能访问 `value`。引擎会保留仍被可达闭包需要的环境信息。不要把它理解为“整个外层作用域永远不会销毁”：只有仍可能被观察到的绑定及其引用对象需要继续存活，具体存储与优化方式由引擎决定。

## 闭包记住的是绑定，不是固定数值

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count += 1;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
```

`increment` 与 `getCount` 共享同一个 `count` 绑定。每次调用 `increment` 都写回这个绑定，之后 `getCount` 读取到的自然是新值。不同工厂调用会创建不同的绑定实例：

```js
const first = createCounter();
const second = createCounter();

first.increment();
console.log(first.getCount()); // 1
console.log(second.getCount()); // 0
```

## 闭包常出现在哪里

闭包常见于函数在定义位置之外、或定义位置的执行结束之后才被调用的场景：

- 返回内部函数或将函数保存到对象中；
- `setTimeout`、`setInterval` 等异步回调；
- DOM 事件监听器；
- Promise、网络请求或其他异步 API 的回调；
- 高阶函数接收或返回函数；
- ES 模块中导出的函数访问模块私有绑定。

但不要把“传了一个函数”机械等同于“它一定持有有用的外层数据”。所有函数都有定义时的词法环境关联；只有函数实际引用了外层绑定时，才会形成值得讨论的捕获关系。

```js
function wait(message) {
  setTimeout(function timer() {
    console.log(message);
  }, 1000);
}

wait("hello");
```

回调 `timer` 在 `wait` 返回后执行，仍能读取参数 `message`。这就是异步回调中最常见的闭包形式。

### IIFE 是否算闭包

从广义的语言模型看，函数值会关联其词法环境，因此不必武断地说 IIFE “不算闭包”。但下面的 IIFE 没有把内部函数或数据带到外部，外层调用结束后也没有可观察的延续访问：

```js
(function initialize() {
  const version = "1.0";
  console.log(version);
}());
```

它展示的是普通的词法查找与函数作用域，而不是闭包最有价值的“函数离开创建环境后仍访问外层绑定”的用法。把术语争论转化为实际问题更有帮助：是否存在一个存活函数继续使用外层状态？

## 循环与闭包：关键是“共享绑定”

经典问题不是“闭包失效”，而是多个回调捕获了同一个 `var` 绑定：

```js
for (var index = 1; index <= 5; index += 1) {
  setTimeout(() => console.log(index), index * 1000);
}

// 每个回调执行时，通常都会输出 6
```

`var index` 属于外层函数或脚本作用域。循环结束时它已变为 `6`，所有回调读取的都是这个同一绑定。

### 使用 IIFE 为每一轮创建绑定

```js
for (var index = 1; index <= 5; index += 1) {
  (function schedule(value) {
    setTimeout(() => console.log(value), value * 1000);
  }(index));
}
```

每次 IIFE 调用都会创建独立的参数绑定 `value`，对应的定时器回调各自捕获自己的 `value`。

### 使用 `let` 的每轮迭代绑定

现代 JavaScript 优先使用 `let`：

```js
for (let index = 1; index <= 5; index += 1) {
  setTimeout(() => console.log(index), index * 1000);
}
```

`for (let ...)` 为每轮迭代提供独立的 `index` 绑定，因此五个回调分别读取 `1` 到 `5`。这不是定时器的特殊规则，而是循环的词法绑定语义。

## 经典模块模式：私有状态与公开 API

在 ES 模块普及前，IIFE 加闭包常用来创建单例模块：将状态放在封闭函数中，只返回明确的 API。

```js
const coolModule = (function createCoolModule() {
  const message = "private data";

  function printMessage() {
    console.log(message);
  }

  return { printMessage };
}());

coolModule.printMessage(); // "private data"
console.log(coolModule.message); // undefined
```

外部拿不到名为 `message` 的属性，但 `printMessage` 的闭包可以访问该绑定。这里的“私有”是封装与防误用，不是安全边界：只要把能读取或修改私有状态的方法暴露出去，调用方仍可通过它们影响状态。

### 工厂模块：每次调用一份独立状态

```js
function createModule(id) {
  const innerId = id;

  return {
    log() {
      console.log(innerId);
    },
  };
}

const firstModule = createModule(1);
const secondModule = createModule(2);

firstModule.log(); // 1
secondModule.log(); // 2
```

每次 `createModule` 调用都有自己独立的 `innerId` 绑定。模块模式不要求必须使用 IIFE 或返回单个函数；关键是外部持有的 API 通过闭包访问封闭状态。

### 公开 API 可以是可变对象

如果返回的是普通对象，公开 API 当然可以被内部或外部在运行时替换：

```js
const moduleApi = (function createModule() {
  const publicApi = {
    print() {
      console.log("old output");
    },
    change() {
      publicApi.print = () => console.log("new output");
    },
  };

  return publicApi;
}());

moduleApi.print(); // "old output"
moduleApi.change();
moduleApi.print(); // "new output"
```

是否允许这种可变 API 是设计选择。大多数业务模块更容易维护的做法是导出稳定函数，并把状态变化限制在模块内部。

## ES 模块：现代的文件级封装

ES 模块在以模块方式加载时拥有独立的顶层作用域，并使用静态的 `import`/`export` 语法声明依赖和公开接口。浏览器中通常通过 `<script type="module">` 加载；构建工具则会根据项目配置解析模块：

```js
// counter.js
let count = 0;

export function increment() {
  count += 1;
}

export function getCount() {
  return count;
}
```

```js
// app.js
import { getCount, increment } from "./counter.js";

increment();
console.log(getCount()); // 1
```

导出的函数通过模块词法环境访问 `count`，效果上与模块模式中的闭包一致，但依赖关系可由工具静态分析。导出名称由模块源代码确定，不能在运行时随意新增；不过导出的绑定是实时绑定（live binding），导出对象的内部内容也仍然可能变化。导入方不能给导入绑定重新赋值。

动态 `import()` 可以按运行时条件加载模块，但不会改变模块内部仍有独立作用域、静态声明导出的事实。

## 闭包与内存：管理生命周期，而不是回避闭包

闭包本身不是内存泄漏。问题发生在不再需要的函数仍被长生命周期对象持有，从而间接保留了大对象、DOM 节点或缓存数据。

```js
function subscribe(button, records) {
  const onClick = () => console.log(records.length);
  button.addEventListener("click", onClick);

  return () => button.removeEventListener("click", onClick);
}
```

调用方在组件卸载或功能结束时执行返回的清理函数，就能移除事件监听器，使不再需要的引用有机会被回收。类似地，应在不需要时 `clearInterval` / `clearTimeout`，取消订阅，并清理全局缓存中的回调引用。

节流、防抖、记忆化缓存等模式也依赖闭包保存状态。它们很实用，但缓存大小、失效策略和清理时机需要按业务生命周期设计。

## 练习：两个函数会共享计数吗

```js
function createIncrementer() {
  let value = 0;

  return () => ++value;
}

const increaseA = createIncrementer();
const increaseB = createIncrementer();

console.log(increaseA());
console.log(increaseA());
console.log(increaseB());
```

<details>
<summary>查看解析</summary>

依次输出 `1`、`2`、`1`。`increaseA` 和 `increaseB` 分别来自两次 `createIncrementer()` 调用，因此各自捕获不同的 `value` 绑定。前两次调用修改的是第一个绑定，第三次调用读取和修改的是第二个绑定。

</details>

## 本章要点

1. 闭包让函数在之后执行时仍可访问其定义位置的词法环境。
2. 闭包访问的是外层绑定而非数值快照，多个闭包可以共享同一个可变状态。
3. 定时器、事件、Promise 回调、高阶函数和模块都经常使用闭包。
4. `var` 循环问题来自多个回调共享同一绑定；IIFE 参数或 `for (let ...)` 可以提供每轮独立绑定。
5. 经典模块模式通过闭包隐藏状态并公开有限 API；现代项目优先使用 ES 模块。
6. ES 模块导出是静态声明的实时绑定，导入绑定不能由导入方重新赋值。
7. 闭包不是泄漏；及时移除监听器、清理计时器和取消订阅，才能避免不必要的长期引用。

## 延伸阅读

- [MDN：闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures)
- [MDN：模块指南](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)
- [MDN：addEventListener()](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)
