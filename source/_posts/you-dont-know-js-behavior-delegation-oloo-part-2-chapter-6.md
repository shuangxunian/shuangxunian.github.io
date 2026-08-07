---
title: "《你不知道的 JavaScript（上卷）》第二部分第六章：行为委托与 ES6 class"
excerpt: "以 OLOO（Objects Linked to Other Objects）理解原型委托，比较类式继承与行为对象设计，并厘清 ES6 class、extends、super 和静态方法的真实语义。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 原型, OLOO, class, 对象, 前端基础]
---

> 本文完成第二部分“`this` 和对象原型”，承接[原型](/2026/08/07/you-dont-know-js-prototypes-part-2-chapter-5/)。OLOO 是本书提出的一种直接使用对象关联的设计风格，不是现代 JavaScript 唯一正确的对象设计方式；附录部分会说明 ES6 `class` 的原型基础与额外语义。

## 从“继承复制”转向“行为委托”

本书希望读者警惕把传统类模型直接投射到 JavaScript 上。经典面向类语言通常以类声明实例结构与行为，子类复用和重写父类行为；具体运行时不一定真的逐份复制对象，但“实例属于类”的抽象很强。

JavaScript 的对象模型直接暴露了另一种机制：对象自身找不到属性时，可以通过 `[[Prototype]]` 向关联对象继续查找。这是**委托**，不是把原型对象的属性复制到当前对象。

```text
button instance -> Button behavior -> Widget behavior -> Object.prototype -> null
```

OLOO（Objects Linked to Other Objects）选择把这种链条作为主要设计语言：先定义行为对象，再用 `Object.create()` 建立关联并创建具体对象。它避免把行为对象称为“类”、把具体对象称为“类实例”，但底层仍是同一套原型链属性查找。

## 同一个控制器，两个表达方式

### 类式写法：`class`、`extends` 与 `super`

下面的类式写法可以清晰表达一个 `Button` 是 `Widget` 的专门化版本：

```js
class Widget {
  constructor(width) {
    this.width = width;
  }

  render() {
    console.log(`width: ${this.width}`);
  }
}

class Button extends Widget {
  constructor(width, text) {
    super(width);
    this.text = text;
  }

  render() {
    super.render();
    console.log(`button text: ${this.text}`);
  }
}

const button = new Button(200, "Submit");
button.render();
```

这里实例状态在构造阶段写到 `button` 自身；`render` 等实例方法则共享在 `Widget.prototype` 与 `Button.prototype` 上。`super(width)` 在派生构造函数中完成父类初始化，`super.render()` 调用按当前方法的词法 HomeObject 解析到的原型方法。

### OLOO 写法：行为对象与委托实例

OLOO 不使用构造调用，而是先创建可被委托的行为对象：

```js
const WidgetBehavior = {
  init(width) {
    this.width = width;
    return this;
  },

  render() {
    console.log(`width: ${this.width}`);
  },
};

const ButtonBehavior = Object.create(WidgetBehavior);

ButtonBehavior.init = function init(width, text) {
  WidgetBehavior.init.call(this, width);
  this.text = text;
  return this;
};

ButtonBehavior.render = function render() {
  WidgetBehavior.render.call(this);
  console.log(`button text: ${this.text}`);
};

const button = Object.create(ButtonBehavior).init(200, "Submit");
button.render();
```

`button` 没有自己的 `render`，读取时会先委托到 `ButtonBehavior`；`ButtonBehavior` 自身也没有从 `WidgetBehavior` 复制方法，而是通过原型关联继续委托。`init()` 显式把状态写到具体 `button` 上，避免将可变状态放入行为对象。

两种写法都依赖原型链。区别在于：`class` 用构造、类名和 `super` 组织类层级；OLOO 用对象和显式关联表达“谁向谁委托”。哪一种更清晰，取决于领域和团队，而不是语言底层是否存在原型链。

## 覆写行为：显式上游调用的代价

在上面的 OLOO 示例中，`ButtonBehavior.render` 覆写了同名行为。为了复用上游实现，它显式写出：

```js
WidgetBehavior.render.call(this);
```

优点是依赖对象一目了然；代价是来源名称被硬编码。重命名、交换原型或改变层级时，需要同步修改这些调用。OLOO 没有内建的“自动父方法调用”机制，不能把这类显式调用说成没有耦合的 `super` 替代品。

ES6 对象方法可在具有 HomeObject 的场景使用 `super`，但将其加入 OLOO 往往又引入类式覆写语义。多数情况下，更好的方向是减少深层覆写：把共享步骤拆成独立函数或协作者，通过组合来完成行为。

## 初始化对象而不是“构造实例”

OLOO 常用 `Object.create(behavior)` 创建具体对象，再调用 `init()` 写入状态。`init` 的名字强调它是普通方法，不是只可由 `new` 触发的构造函数。

这种风格有两个纪律：

1. 行为对象上的属性应尽量是稳定行为，不保存会被每个具体对象改写的可变状态；
2. 初始化方法应清楚地建立具体对象所需的不变量，必要时返回 `this` 以支持链式创建。

它并不自动带来封装或不可变性。行为对象仍是可访问对象，具体对象也能直接改写自己的属性；需要私有状态、验证或资源生命周期时，仍要用闭包、私有字段、模块或明确 API 设计解决。

## 内省：检查关联，而非猜测“类型”

OLOO 中可以直接检查对象关联：

```js
console.log(WidgetBehavior.isPrototypeOf(ButtonBehavior)); // true
console.log(ButtonBehavior.isPrototypeOf(button)); // true
console.log(Object.getPrototypeOf(button) === ButtonBehavior); // true
```

这些问题回答的是“一个对象是否出现在另一个对象的原型链上”。它们不需要构造函数。

`instanceof` 也不是只能搭配 `new` 使用；其默认语义是检查某个函数的 `prototype` 是否出现在目标对象的原型链中，因而很适合类/构造函数关联。但它不能直接回答“`WidgetBehavior` 是否是 `button` 的原型”，而且受跨 realm、原型替换和 `Symbol.hasInstance` 影响。

`obj.constructor` 同样不可靠：它常来自原型链，替换原型后可能失真，也不是对象的内建类型标签。业务逻辑应使用明确的能力接口、schema 或领域标记，而非依赖 `constructor` 猜测类型。

## `Object.create` 与对象字面量方法

`Object.create(proto)` 是建立 OLOO 关联的直接方式。之后用赋值添加行为，默认属性可写、可枚举、可配置，通常比直接使用描述符更符合普通对象预期：

```js
const ButtonBehavior = Object.create(WidgetBehavior);
ButtonBehavior.render = function render() {
  // ...
};
```

也可以在 `Object.create` 的第二个参数中一次定义属性描述符，但此 API 的布尔描述符默认是 `false`：

```js
const ButtonBehavior = Object.create(WidgetBehavior, {
  render: {
    value() {
      console.log("render");
    },
    writable: true,
    enumerable: true,
    configurable: true,
  },
});
```

漏写这些标记会得到不可写、不可枚举、不可配置的方法，不应为了“写得短”而忽略描述符语义。

## 附录 A 扩展阅读

ES6 `class` 的 `extends`、`super`、字段、私有字段及静态继承已在独立文章中详解：[《你不知道的 JavaScript（上卷）》第二部分附录 A：ES6 中的 class](/2026/08/07/you-dont-know-js-es6-class-appendix-part-2/)。

## 第二部分知识地图

1. `this` 是调用相关的接收者绑定，和词法作用域不同；普通函数按调用形式判断，箭头函数则词法捕获外层 `this`。
2. 对象以字符串/Symbol 键保存属性，读取与写入受到描述符和原型链影响。
3. 混入是属性复制，方法借用是临时调用，原型委托是不复制的关联；三者不能互换。
4. `[[Prototype]]` 决定对象缺失属性时向谁委托；`Object.create()` 可直接建立这种关联。
5. `class` 建立在原型机制上，同时提供严格模式、`super`、私有字段和静态继承等额外语义。
6. OLOO 是理解和使用对象委托的一种有价值视角，但组合与明确 API 通常比深层继承更稳健。

## 练习：OLOO 方法在哪里定义，`this` 又是谁

```js
const Speaker = {
  speak() {
    return this.message;
  },
};

const greeting = Object.create(Speaker);
greeting.message = "Hello";

console.log(Object.hasOwn(greeting, "speak"));
console.log(greeting.speak());
```

<details>
<summary>查看解析</summary>

输出为 `false` 和 `"Hello"`。`speak` 定义在 `Speaker` 上，`greeting` 通过原型链找到它；但调用表达式是 `greeting.speak()`，因此函数执行时的 `this` 是 `greeting`，可读取 `greeting.message`。

</details>

## 本章要点

1. OLOO 直接表达对象间的原型委托，不要求使用构造函数、`new` 或类名。
2. 类式继承与 OLOO 都依赖原型链，但采用不同的组织和表达方式。
3. 覆写行为时，OLOO 的上游调用需要显式指定来源，来源名称会形成耦合。
4. `isPrototypeOf` 与 `Object.getPrototypeOf` 适合检查对象关联；`instanceof` 和 `constructor` 不能替代领域类型校验。
5. `class` 的实例方法仍基于原型链，但 `extends`、`super`、严格模式、私有字段和静态继承使其具有独立语义。
6. OLOO、`class`、工厂和组合都是工具。选择应基于状态边界、协作方式和团队可读性，而不是范式口号。

## 延伸阅读

- [MDN：继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN：Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN：类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
- [MDN：super](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/super)
