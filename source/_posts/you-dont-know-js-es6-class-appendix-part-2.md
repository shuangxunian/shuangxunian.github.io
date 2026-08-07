---
title: "《你不知道的 JavaScript（上卷）》第二部分附录 A：ES6 中的 class"
excerpt: "从原型链基础理解 ES6 class，并梳理 constructor、extends、super、静态成员、字段与私有字段等 class 专有语义及其工程选型边界。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, class, 原型, super, ES6, 前端基础]
---

> 本文是第二部分的独立附录，配合[行为委托与 ES6 class](/2026/08/07/you-dont-know-js-behavior-delegation-oloo-part-2-chapter-6/)阅读。理解原型委托是理解 `class` 的基础，但不能因此把 `class` 简化为可以逐行替换的“纯语法糖”。

## `class` 的定位：原型基础上的类式语法与额外语义

`class` 不会把父类方法或属性复制到每个实例。实例方法仍定义在 `ClassName.prototype` 上，实例通过原型链查找并共享这些方法；`extends` 也会建立相应的原型关联。从这个角度看，原型机制仍是底层基础。

但 `class` 还定义了一组手写构造函数默认没有的语义：必须使用 `new`、类体严格模式、暂时性死区（TDZ）、方法默认不可枚举、`super` 的 HomeObject 规则、字段初始化、私有字段和静态继承等。因此，准确的说法是：**`class` 没有改变 JavaScript 以原型链查找公共行为的事实，但它不是可忽略语义的文本缩写。**

## 基础语法：构造器、原型方法与字段

```js
class Car {
  constructor(wheelCount) {
    this.wheelCount = wheelCount;
  }

  drive() {
    console.log(`${this.wheelCount} wheels moving`);
  }
}

const car = new Car(4);
car.drive();

console.log(Object.getPrototypeOf(car) === Car.prototype); // true
```

`constructor` 是类定义中最多只能出现一次的特殊方法。使用 `new Car(4)` 时，它会执行并初始化实例。基类省略 `constructor` 时有一个空的默认构造器；派生类省略时则具有类似 `constructor(...args) { super(...args); }` 的默认行为。

`drive` 定义在 `Car.prototype` 上而非每个实例自身，因此多个实例共享同一个方法函数。类方法默认不可枚举，这与对象字面量方法和普通赋值创建的可枚举属性不同。

### 类声明的 TDZ 与严格模式

类声明在其执行到声明前处于 TDZ：

```js
const car = new Car(); // ReferenceError

class Car {}
```

类体天然处于严格模式。类不能像普通函数一样直接调用：

```js
class Car {}

Car(); // TypeError
```

这些规则能避免一些构造调用误用，但也意味着 `class` 与函数声明的提升和调用习惯不同。

### 公共字段与私有字段

字段声明会在每个实例上创建相应属性：

```js
class Counter {
  count = 0;
  #step = 1;

  increment() {
    this.count += this.#step;
  }
}
```

`count` 是普通自有属性，可以由外部读取或修改。`#step` 是类私有元素，不是字符串或 `Symbol` 属性键，类外访问会产生语法错误。私有字段提供的封装不能用普通 OLOO 属性或原型属性等价模拟；闭包可以隐藏状态，但生命周期与 API 形态不同。

## 静态成员属于类构造器对象

使用 `static` 定义的成员位于类构造器对象自身，不在实例原型上：

```js
class Car {
  static category() {
    return "vehicle";
  }
}

console.log(Car.category()); // "vehicle"
console.log(typeof new Car().category); // "undefined"
```

静态方法适合工厂、注册表、解析器或与单个实例无关的工具行为。它们同样会沿“类构造器对象的原型链”参与静态继承，后文会展示。

## `extends` 建立两条关联

`class Child extends Parent` 不只是等价于 `Child.prototype = Object.create(Parent.prototype)`。它至少建立两条关联：

```text
child instance -> Child.prototype -> Parent.prototype -> Object.prototype
Child constructor -> Parent constructor -> Function.prototype
```

第一条让实例查找到父类实例方法，第二条让子类查找到父类静态方法。`extends` 还带来派生构造函数初始化规则与 `super` 语义。

```js
class Vehicle {
  constructor(engine) {
    this.engine = engine;
  }

  ignite() {
    return `${this.engine} engine started`;
  }

  static category() {
    return "vehicle";
  }
}

class Car extends Vehicle {
  constructor(engine, wheelCount) {
    super(engine);
    this.wheelCount = wheelCount;
  }

  drive() {
    return `${super.ignite()} on ${this.wheelCount} wheels`;
  }
}

const car = new Car("electric", 4);
console.log(car.drive());
console.log(Car.category()); // "vehicle"
```

## `super`：从 HomeObject 的原型开始查找

### 派生构造函数中的 `super()`

在派生类构造函数中，`super(args)` 调用父类构造逻辑，并取得当前实例的 `this` 绑定。访问 `this` 或返回 `this` 前必须先完成 `super()`，除非构造函数显式返回另一个对象：

```js
class Parent {}

class Child extends Parent {
  constructor() {
    super();
    this.ready = true;
  }
}
```

这不是单纯的 `Parent.call(this, ...args)` 文本替换。派生类构造与 `new.target`、父类可构造性和返回值处理有关；尤其当父类是内建类或返回对象时，手写模拟更容易遗漏语义。

### 方法中的 `super.property`

实例方法中的 `super.method()` 会从定义该方法时记录的 HomeObject 的原型开始查找方法，再以当前 `this` 作为接收者调用。它不是“编译期把父类函数名写死”：若 HomeObject 的原型被改动，后续 `super` 查找会反映新的原型链。

这也意味着 `super` 可以正确触发上游 getter/setter；它不总等价于 `Parent.prototype.method.call(this)`。静态方法中的 `super` 则从类构造器对象的原型链开始查找。

```js
class Base {
  static label() {
    return "base";
  }
}

class Derived extends Base {
  static label() {
    return `${super.label()} -> derived`;
  }
}

console.log(Derived.label()); // "base -> derived"
```

## `class` 的边界：不要把它当成继承万能药

### 没有多重继承

`extends` 只提供单继承。多个来源的行为组合仍需要组合对象、函数、依赖注入或经过严格冲突策略设计的 mixin；`class` 不会自动解决命名冲突、初始化顺序和可变状态共享问题。

### 原型共享并非自动陷阱

类方法放在原型上共享通常正是性能和语义上的优点。真正危险的是把可变实例状态放到原型对象：

```js
class BadList {
  // 不要把可变数组挂到 BadList.prototype 上
}

BadList.prototype.items = [];
```

应在构造器或实例字段中初始化可变数据：

```js
class GoodList {
  items = [];
}
```

这样每个实例都会得到自己的数组。

### 原型关联技术上可变，但不应依赖

`Object.setPrototypeOf(Car.prototype, Other.prototype)` 或 `Object.setPrototypeOf(Car, Other)` 可以在运行时改变关联，`super` 的查找也会随 HomeObject 当前原型变化。但这种操作会破坏读者的预期，并可能妨碍引擎优化；应将类层级视为创建后稳定的设计，而非运行时配置通道。

## `class`、OLOO 与组合：如何选择

| 关注点 | `class` | OLOO | 组合/工厂函数 |
| --- | --- | --- | --- |
| 主要表达 | 具有实例身份的类式模型 | 对象直接向行为对象委托 | 显式组装依赖与状态 |
| 初始化 | `new` + 构造器 / 字段 | `Object.create()` + 初始化方法 | 工厂函数参数 |
| 上游行为 | `super`，有 HomeObject 语义 | 常需显式引用委托对象 | 调用协作者或纯函数 |
| 私有状态 | `#private` 字段可用 | 通常借助闭包或约定 | 闭包或模块私有状态 |
| 适合场景 | 生命周期、实例 API、框架惯例清晰 | 稳定且扁平的行为对象 | 多来源能力与依赖关系复杂 |

没有一条规则要求项目“全面禁止 `class`”或“所有对象都必须 OLOO”。在需要私有字段、静态成员、框架接口或熟悉类模型的团队中，`class` 往往清晰；需要避免深层继承时，优先组合；需要直接表达少量对象委托关系时，OLOO 是很好的工具。

## 上卷收尾：从作用域到对象模型

至此，书中上卷的两个主题可以串联起来：

1. 第一部分说明词法作用域、提升、闭包如何决定标识符和状态跨执行时间的行为。
2. 第二部分说明 `this` 由调用形式决定，对象属性通过原型链委托，`class` 与 OLOO 都是在此基础上的组织方式。

这些内容是 JavaScript 运行时模型的基础，而不是“静态语言基础”。不同版本和译本的后续卷册编排有所差异，但通常会继续扩展类型与语法、异步控制流和性能等主题。

## 练习：方法在哪，状态在哪

```js
class Counter {
  count = 0;

  increment() {
    this.count += 1;
  }
}

const first = new Counter();
const second = new Counter();

first.increment();

console.log(first.count, second.count);
console.log(first.increment === second.increment);
```

<details>
<summary>查看解析</summary>

输出 `1 0` 和 `true`。`count` 是实例字段，每个实例各有一份；`increment` 是原型方法，两个实例通过原型链访问同一个函数对象。

</details>

## 本章要点

1. `class` 的公共方法复用仍基于原型链，不会复制父类成员到每个实例。
2. `class` 同时具有独立语义：必须 `new`、TDZ、严格模式、不可枚举方法、字段、私有字段、`extends` 与 `super`。
3. `extends` 同时建立实例方法与静态方法的继承关联；派生构造函数在使用 `this` 前必须执行 `super()`。
4. `super` 从 HomeObject 的原型开始查找，并以当前接收者访问属性；它不是简单硬编码的父方法调用。
5. `class` 只提供单继承。将可变状态初始化在实例上，并优先组合而非构造深层继承树。
6. OLOO、`class` 与工厂函数各有适用边界；理解原型机制后再按领域模型和团队约定选择。

## 延伸阅读

- [MDN：类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
- [MDN：extends](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/extends)
- [MDN：super](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/super)
- [MDN：私有元素](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/Private_elements)
