---
title: "《你不知道的 JavaScript（上卷）》第二部分第四章：混合对象“类”"
excerpt: "区分传统类模型、构造函数与原型委托、显式混入和方法借用，理解为什么属性复制并不等同于继承，以及何时应优先组合。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 原型, mixin, class, 对象, 前端基础]
---

> 本章延续[对象](/2026/08/07/you-dont-know-js-objects-part-2-chapter-3/)的原型与属性模型。书中批评的是把 Java/C++ 风格的“类复制”心智模型机械搬到 JavaScript；现代 JavaScript 已有 `class` 语法和额外语义，但对象行为查找仍建立在原型委托之上。

## 先区分三种“复用”

讨论“类”“继承”和混入前，先分清它们实际在做什么：

| 方式 | 核心动作 | 典型结果 |
| --- | --- | --- |
| 原型委托 | 建立对象之间的原型关联 | 查找行为时向原型对象委托 |
| 显式混入 | 将属性描述符复制到目标 | 目标获得自己的同名属性 |
| 方法借用 | 用 `call`/`apply` 以另一对象为 `this` 调用函数 | 不复制属性，只复用一次行为 |

这些方式都能实现某种复用，但它们对状态、覆盖、重构和调试的影响完全不同。混淆它们，正是许多“JavaScript 类继承”代码难以维护的起点。

## 传统类模型与 JavaScript 的对象模型

在经典面向类语言中，类通常描述实例的字段和行为，子类可继承并重写父类行为，多态调用依据实例的运行时类型选择实现。具体语言的对象内存布局和继承实现并不一定真的“复制整份父类”，但“实例属于某个类”的心智模型很强。

JavaScript 更直接地从对象出发。对象可以通过 `[[Prototype]]` 链在属性缺失时向另一个对象查找；没有必要先定义一个类再创建对象。`new` 也不是复制一个模板，而是创建新对象、建立原型关联并以新对象作为构造调用的 `this`。

ES2015 的 `class` 不是完全无意义的“纯文本糖”：它引入了必须用 `new` 调用、类体严格模式、方法不可枚举、`super`、私有字段等语义。但实例方法的共享和查找依然通过原型链完成，因此理解原型委托仍然必要。

## 构造函数与原型：实例状态和共享行为

任何普通可构造函数都能配合 `new` 使用；“构造函数”只是这种调用角色，不是独立的函数类型。约定上使用 PascalCase 命名以提醒调用方使用 `new`。

```js
function Car(wheelCount) {
  this.wheelCount = wheelCount; // 每个实例各自拥有的状态
}

Car.prototype.drive = function drive() {
  console.log(`${this.wheelCount} wheels moving`);
};

const first = new Car(4);
const second = new Car(4);

console.log(first.drive === second.drive); // true
```

`drive` 是 `Car.prototype` 上的一个函数值，两个实例都通过原型链查找到同一个函数，不会为每个实例复制函数。执行 `first.drive()` 时，`this` 仍由调用表达式绑定为 `first`。

## 原型式“继承”：避免用父实例当子原型

下面是容易出现问题的旧写法：

```js
Child.prototype = new Parent();
```

它会执行 `Parent` 构造函数，并把父构造函数创建的实例状态放到 `Child.prototype` 上。若该状态是数组或对象，所有子实例可能意外共享它；构造函数还有副作用或需要参数时，问题更明显。

更明确的原型关联方式是 `Object.create()`：

```js
function Vehicle(engine) {
  this.engine = engine;
}

Vehicle.prototype.ignite = function ignite() {
  return `${this.engine} engine started`;
};

function Car(engine, wheelCount) {
  Vehicle.call(this, engine); // 初始化这个实例自己的状态
  this.wheelCount = wheelCount;
}

Car.prototype = Object.create(Vehicle.prototype);
Object.defineProperty(Car.prototype, "constructor", {
  value: Car,
  writable: true,
  configurable: true,
});

Car.prototype.drive = function drive() {
  return `${this.ignite()} with ${this.wheelCount} wheels`;
};

const car = new Car("electric", 4);
console.log(car.drive());
```

这里 `Vehicle.call(this, engine)` 初始化的是 `car` 的自有状态；`Car.prototype` 只委托给 `Vehicle.prototype`，不会携带某个父实例的可变数据。修复 `constructor` 属性有助于反射和调试，但它不是 `new` 的工作依据，也不是运行时类型检查的可靠工具。

这种模式仍有成本：子构造函数必须显式调用父构造函数，重写方法时调用“父方法”往往要硬编码 `Vehicle.prototype.ignite.call(this)`。这正是书中质疑类式模拟的原因之一。

## 显式混入：复制属性，而不是建立委托

显式混入（mixin）将一个来源对象的属性复制到目标对象，以组合多个来源的能力。最简单的写法通常是：

```js
const canLog = {
  log() {
    console.log(this.name);
  },
};

const user = Object.assign({ name: "Ada" }, canLog);
user.log(); // "Ada"
```

这不是原型继承：`user` 得到了自己的 `log` 属性。`Object.assign` 只复制自有可枚举属性，会读取来源 getter、执行目标 setter，且不保留描述符。若确实需要复制所有自有键与描述符，应显式处理冲突：

```js
function copyMixin(target, source) {
  for (const key of Reflect.ownKeys(source)) {
    if (key === "constructor" || Object.hasOwn(target, key)) {
      continue;
    }

    const descriptor = Object.getOwnPropertyDescriptor(source, key);
    Object.defineProperty(target, key, descriptor);
  }

  return target;
}
```

这段工具刻意跳过既有属性，避免无声覆盖；实际项目还应根据领域决定冲突是拒绝、覆盖还是组合。不要使用 `for...in` 作为通用混入实现，因为它会遍历来源的可枚举继承属性。

### 复制仍可能共享可变引用

混入复制的是属性值或描述符，而不是任意对象图的深拷贝：

```js
const defaults = {
  options: { retry: 3 },
};

const serviceA = Object.assign({}, defaults);
const serviceB = Object.assign({}, defaults);

serviceA.options.retry = 1;
console.log(serviceB.options.retry); // 1
```

两个目标对象的 `options` 指向同一个对象。若需要独立状态，应在创建实例时创建新对象、使用工厂函数，或针对数据结构采用合适的复制策略。

### “伪多态”的耦合

当混入后的方法需要调用来源对象的同名实现时，常见写法是：

```js
Vehicle.prototype.ignite.call(this);
```

它把来源对象名硬编码进实现。重命名、替换来源或增加多层组合时，依赖关系变得隐蔽；这不是 `super` 的通用替代品。优先让共享能力保持小而独立，通过明确函数调用、组合对象或更清晰的接口协作。

## 方法借用：书中的“隐式混入”

书中将下面一类模式称为隐式混入：

```js
const logger = {
  log() {
    console.log(this.name);
  },
};

const user = {
  name: "Ada",
  print() {
    logger.log.call(this);
  },
};

user.print(); // "Ada"
```

这里没有复制 `logger.log`，而是以 `user` 为 `this` 临时调用它。更准确的工程术语通常是**方法借用**。它不仅能操作状态，也会隐式要求接收者满足 `logger.log` 所需的字段和不变量；这种隐藏约定会使重构变难。

若只是复用纯计算逻辑，直接写成显式参数函数更清晰：

```js
function formatName(name) {
  return name.toUpperCase();
}
```

## 寄生式工厂：创建后再增强对象

寄生继承（parasitic inheritance）通常指一种工厂式写法：先创建一个对象，再为它添加或包装行为后返回。它可以避开 `Child.prototype = new Parent()` 的共享实例状态问题，但会引入包装函数和显式“父方法”调用的复杂度。

```js
const vehicleProto = {
  drive() {
    return `${this.engine} moving`;
  },
};

function createCar(engine) {
  const car = Object.create(vehicleProto);
  car.engine = engine;
  car.wheelCount = 4;

  const inheritedDrive = car.drive;
  car.drive = function drive() {
    return `${inheritedDrive.call(this)} on ${this.wheelCount} wheels`;
  };

  return car;
}

console.log(createCar("electric").drive());
```

这种方式本质上更接近工厂和对象组合。它没有自动解决命名冲突、深拷贝或封装问题，应在确实能让领域模型更清晰时使用。

## 为什么混入常常不如组合

混入仍可用于小型、无状态、约定清晰的横切能力，例如为测试对象添加一个受控的序列化函数。但它有明显限制：

- 复制属性会制造来源与目标之间的重复定义，后续修改不会自动同步；
- 可变引用可能被多个目标共享；
- 同名冲突、初始化顺序和“父方法”调用难以标准化；
- 复制出的公开属性并不自动私有，但函数仍可以通过闭包保存私有状态。

与其构造复杂的“多重继承”层级，通常更推荐**组合**：把所需能力作为独立对象、函数或服务显式注入，再由对象协调调用。JavaScript 本身不提供 `class` 的多重继承，`class` 语法也不会消除混入固有的冲突处理问题。

## `class extends`：更清晰的类式语法，不改变原型基础

现代 `class` 可以更直接地表达单继承与 `super` 调用：

```js
class VehicleClass {
  constructor(engine) {
    this.engine = engine;
  }

  ignite() {
    return `${this.engine} engine started`;
  }
}

class CarClass extends VehicleClass {
  constructor(engine, wheelCount) {
    super(engine);
    this.wheelCount = wheelCount;
  }

  drive() {
    return `${super.ignite()} with ${this.wheelCount} wheels`;
  }
}
```

`extends` 建立原型关系，实例方法仍放在原型上共享；`super` 让方法覆写更清晰。它也带来前文提到的类特有语义，因此不应简单说它与手写构造函数“完全相同”。选择 `class`、工厂、组合或后续介绍的 OLOO，应以团队约定和领域复杂度为准，而不是假设其中某一种天然最优。

## 练习：这是复制还是委托

```js
const prototype = {
  greet() {
    return `Hello, ${this.name}`;
  },
};

const user = Object.create(prototype);
user.name = "Ada";

console.log(Object.hasOwn(user, "greet"));
console.log(user.greet());
```

<details>
<summary>查看解析</summary>

依次输出 `false` 和 `"Hello, Ada"`。`user` 没有自己的 `greet` 属性，读取时会沿原型链在 `prototype` 上找到函数。调用表达式仍是 `user.greet()`，因此函数中的 `this` 是 `user`。

</details>

## 本章要点

1. 原型委托、属性复制和方法借用都是复用手段，但不是同一种机制。
2. `new` 创建新对象并建立原型关联，不会复制一个“类模板”。
3. 构造函数的实例状态应放在实例自身，共享行为放在原型；建立原型关系优先使用 `Object.create(Parent.prototype)`。
4. 显式混入复制属性，不会自动处理描述符、冲突、可变引用或来源更新。
5. 书中的“隐式混入”本质是方法借用，会引入对接收者状态的隐式要求。
6. 组合通常比模拟多重继承更容易维护；OLOO 是下一章将介绍的另一种对象关联设计。
7. `class extends` 提供更清晰的单继承与 `super` 语法，但实例行为查找仍基于原型链。

## 延伸阅读

- [MDN：继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN：Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN：类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
- [MDN：Object.assign()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
