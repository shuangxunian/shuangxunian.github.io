---
title: "《你不知道的 JavaScript（上卷）》第二部分第五章：原型"
excerpt: "理解对象的 [[Prototype]] 委托链、属性屏蔽、函数 prototype 与 constructor 的区别，以及 Object.create、instanceof 和无原型字典的边界。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 原型, 对象, instanceof, 前端基础]
---

> 本文承接[混合对象“类”](/2026/08/07/you-dont-know-js-mixins-and-classes-part-2-chapter-4/)。原型不是对象的“父类副本”，而是一条属性查找时可委托给其他对象的关联。下一章将以 OLOO（Objects Linked to Other Objects）进一步讨论如何直接设计这种关联。

## `[[Prototype]]`：对象间的委托关联

普通对象具有内部的 `[[Prototype]]` 链接，它要么引用另一个对象，要么为 `null`。当对象自身没有某个属性时，属性读取可以沿该链接继续查找，这便形成原型链。

```js
const parent = { value: 2 };
const child = Object.create(parent);

console.log(Object.getPrototypeOf(child) === parent); // true
console.log(child.value); // 2
```

`Object.getPrototypeOf(obj)` 是读取直接原型的标准 API；`Object.create(proto)` 则创建一个以 `proto` 为原型的新对象。应优先使用这两个 API。

`obj.__proto__` 是历史遗留访问器，在现代环境中存在规范化的兼容定义，但不适合作为业务代码的原型操作 API。它容易与对象字面量中的特殊 `__proto__` 语法混淆，也不适用于所有无原型对象。更不要频繁调用 `Object.setPrototypeOf()` 修改已创建对象的原型，这通常会损害运行时优化；应在创建阶段建立关联。

### 原型链如何读取属性

读取 `obj.key` 可用下面的简化过程理解：

1. 在 `obj` 自身属性中查找 `key`；
2. 若不存在，继续查 `Object.getPrototypeOf(obj)`；
3. 重复直到原型为 `null`；
4. 仍未找到时，结果为 `undefined`。

多数普通对象的链最终会经过 `Object.prototype`，其中包含 `toString`、`valueOf`、`hasOwnProperty`、`isPrototypeOf` 等通用方法。但这不是所有对象的强制终点：`Object.create(null)` 的原型直接为 `null`，不同 realm 的内建对象也拥有各自的原型对象。

原型链只参与属性查找，不会复制属性。`child.value` 得到 `parent.value`，并不表示 `child` 获得了一个自己的 `value`。

## 属性屏蔽：写入为何不总是改原型

当原型链上已有同名属性，对当前对象赋值时会发生不同结果。最常见的一种是创建自身同名属性，从而遮蔽（shadow）原型属性：

```js
const parent = { value: 1 };
const child = Object.create(parent);

child.value = 99;

console.log(child.value); // 99
console.log(parent.value); // 1
console.log(Object.hasOwn(child, "value")); // true
```

这个结果成立的前提是：自身没有阻止写入的属性，原型上的同名数据属性可写，且 `child` 可扩展。属性赋值实际对应规范中的 `[[Set]]`，还要考虑访问器和完整性限制。

| 原型上的同名属性 | `child.key = value` 的结果 |
| --- | --- |
| 可写数据属性 | 通常在 `child` 上创建可写自身属性，遮蔽原型 |
| 不可写数据属性 | 赋值失败；严格模式抛出 `TypeError` |
| 带 setter 的访问器 | 调用 setter，通常不创建同名自身属性 |
| 只有 getter 的访问器 | 赋值失败；严格模式抛出 `TypeError` |

```js
const parent = {
  set score(value) {
    this._score = value;
  },
};
const child = Object.create(parent);

child.score = 10;
console.log(Object.hasOwn(child, "score")); // false
console.log(child._score); // 10
```

setter 以接收者 `child` 作为 `this` 执行，因此上例创建的是 `child._score`，不是 `child.score`。若确实要绕过继承而定义自身属性，可以使用 `Object.defineProperty()`，前提是目标对象可扩展；但不应把它当作对原型 API 设计的常规修补手段。

### 自增会先读后写

```js
const parent = { count: 2 };
const child = Object.create(parent);

child.count += 1;

console.log(child.count); // 3
console.log(parent.count); // 2
```

`child.count += 1` 会先沿原型链读取到 `2`，再尝试把 `3` 写回 `child.count`。在本例中写入创建自身属性，于是产生遮蔽。这不是“修改了原型再复制”，而是一次读取委托加一次自身写入。

## 函数的 `prototype` 与对象的 `[[Prototype]]`

这两个名称很像，却不是同一个东西：

- **对象的 `[[Prototype]]`**：该对象属性查找时要委托给谁；
- **构造函数的 `prototype` 属性**：使用 `new Fn()` 创建对象时，新对象应关联到哪个原型对象。

```js
function Foo() {}

const instance = new Foo();

console.log(Object.getPrototypeOf(instance) === Foo.prototype); // true
console.log(Object.getPrototypeOf(Foo) === Function.prototype); // true
```

`Foo.prototype` 不是 `Foo` 自己的原型，`Object.getPrototypeOf(Foo)` 才是。并且不是所有函数值都可构造：箭头函数和方法简写不能作为 `new` 的目标，因此不能把“每个函数都有可供实例使用的 `prototype`”当成通用规则。

## `constructor` 是约定性属性，不是可靠类型标签

默认创建的 `Foo.prototype` 通常有一个 `constructor` 属性指回 `Foo`：

```js
function Foo() {}
const instance = new Foo();

console.log(instance.constructor === Foo); // true
```

实例通常没有自己的 `constructor`，上例是沿原型链找到的。若替换原型对象，这个约定很容易丢失或变成不符合预期的值：

```js
function Foo() {}
Foo.prototype = { greet() {} };

const instance = new Foo();
console.log(instance.constructor === Foo); // false
```

因此，不要用 `obj.constructor` 做运行时类型验证。`Object.prototype.toString.call(value)` 对内建值有时有帮助，但可受 `Symbol.toStringTag` 和跨 realm 影响，也不是万能“真实类型”判断。对数组使用 `Array.isArray()`；对业务对象使用显式标签、schema 校验或可靠的领域不变量。

## 构造调用与原型关联

对可构造函数执行 `new Foo(args)` 时，可以将过程理解为：

1. 创建一个新对象，并将新对象的 `[[Prototype]]` 设为当时的 `Foo.prototype`（若它是对象）；
2. 以新对象为 `this` 执行 `Foo`；
3. 若 `Foo` 显式返回对象或函数，使用该返回值；否则返回新对象。

这解释了为什么后续替换 `Foo.prototype` 不会改变已创建实例的原型链接：每次 `new` 只在创建当时读取一次 `Foo.prototype`。

## 类式原型继承的正确起点

若维护使用构造函数的旧代码，建立原型关联时应避免执行父构造函数：

```js
function Parent(name) {
  this.name = name;
}

Parent.prototype.say = function say() {
  return this.name;
};

function Child(name, level) {
  Parent.call(this, name); // 初始化当前实例的自有状态
  this.level = level;
}

Child.prototype = Object.create(Parent.prototype);
Object.defineProperty(Child.prototype, "constructor", {
  value: Child,
  writable: true,
  configurable: true,
});

const child = new Child("Ada", 2);
console.log(child.say()); // "Ada"
```

`Child.prototype = new Parent()` 会在建立原型链时运行父构造函数，并将父实例属性放入 `Child.prototype`。对象或数组等可变属性会被所有未遮蔽它们的子实例共享，构造函数副作用也会提前发生。`Object.create(Parent.prototype)` 只建立所需的委托关系。

这类“继承”模式仍是用构造函数模拟类层级。它有实际用途，但不是唯一设计方案；下一章的 OLOO 会直接从对象委托关系组织行为。

## 无原型对象与原型判断工具

`Object.create(null)` 创建的对象的 `[[Prototype]]` 就是 `null`：

```js
const dictionary = Object.create(null);
dictionary.scope = "lexical";

console.log(Object.getPrototypeOf(dictionary)); // null
console.log(Object.hasOwn(dictionary, "scope")); // true
```

它适合需要避免原型键干扰的纯字典。没有原型不代表没有属性，只是属性查找不会继续委托到 `Object.prototype`；直接调用 `dictionary.hasOwnProperty` 会失败。

常见原型相关工具的含义如下：

| API / 运算符 | 用途与边界 |
| --- | --- |
| `Object.getPrototypeOf(obj)` | 返回直接原型或 `null` |
| `proto.isPrototypeOf(obj)` | 判断 `proto` 是否出现在 `obj` 的原型链上；无原型对象需通过 `Object.prototype.isPrototypeOf.call` 使用 |
| `obj instanceof Fn` | 判断 `Fn.prototype` 是否出现在 `obj` 的原型链上 |
| `Object.hasOwn(obj, key)` | 只判断自身是否有属性，不用于原型关系判断 |

`instanceof` 很适合判断同一 realm 中由特定构造函数/类创建的实例，但不能作为通用类型系统：跨 iframe/realm 时会失败，且类可自定义 `Symbol.hasInstance`，构造函数的 `prototype` 也可能被替换。

## 练习：读取来自哪里，写入发生在哪里

```js
const base = { role: "reader" };
const account = Object.create(base);

console.log(account.role);
account.role = "author";

console.log(base.role);
console.log(Object.hasOwn(account, "role"));
```

<details>
<summary>查看解析</summary>

依次输出 `"reader"`、`"reader"`、`true`。

第一次读取沿原型链从 `base` 找到 `role`。随后赋值在 `account` 上创建自身 `role` 属性，遮蔽原型值，不会修改 `base.role`。

</details>

## 本章要点

1. `[[Prototype]]` 是对象属性查找时的委托链接，值可以是对象或 `null`；`Object.create` 是建立关联的标准方式。
2. 读取会沿原型链向上查找，属性不会因此被复制到当前对象。
3. 写入受自身属性、原型描述符、setter、严格模式和对象可扩展性共同影响；可写继承属性常被自身属性遮蔽。
4. `Foo.prototype` 指定 `new Foo()` 实例的原型，不是 `Foo` 自己的原型；并非所有函数都可构造。
5. `constructor` 是易丢失的约定属性，`instanceof` 只回答原型链关联问题，二者都不应被当作万能类型判断。
6. 构造函数继承应使用 `Object.create(Parent.prototype)`，避免用父实例作为子原型。
7. `Object.create(null)` 适合无原型字典；存在性检查优先使用 `Object.hasOwn`。

## 延伸阅读

- [MDN：继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN：Object.getPrototypeOf()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf)
- [MDN：Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN：instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)
