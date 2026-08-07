---
title: "《你不知道的 JavaScript（上卷）》第二部分第三章：对象"
excerpt: "系统梳理 JavaScript 对象的创建、属性键、描述符、复制、完整性、遍历，以及原型链上的读取与赋值行为。"
date: 2026-08-07
categories:
- 开发技术
tags: [JavaScript, 对象, 原型, ES6, 前端基础]
---

> 本文承接[第二部分第二章：this 全面解析](/2026/08/07/you-dont-know-js-this-binding-rules-part-2-chapter-2/)。对象既是属性的容器，也是 JavaScript 原型委托与 `this` 方法调用的基础。下一章会继续讨论“类”式设计，本章先建立对象本身的模型。

## 创建对象：字面量是默认选择

创建普通对象最直接的方式是对象字面量：

```js
const article = {
  title: "Objects",
  published: true,
};
```

也可以使用构造形式：

```js
const article = new Object();
article.title = "Objects";
article.published = true;
```

对不带参数的普通对象而言，两者通常得到相同原型的普通对象；但字面量更紧凑，可在创建时清楚展示结构，因此是日常首选。`Object` 构造器还具有把某些传入值包装或直接返回对象的特殊行为，不能把它简单理解为“只能逐个加属性”。

需要没有原型链的字典对象时，可显式创建：

```js
const dictionary = Object.create(null);
dictionary.word = "scope";
```

这种对象没有 `Object.prototype`，因此没有 `toString` 或 `hasOwnProperty` 等继承方法；存在性检查应使用 `Object.hasOwn(dictionary, key)`。

## 值类型、包装对象与属性键

### 原始值不是对象

现代 JavaScript 有 7 种原始值类型：`string`、`number`、`bigint`、`boolean`、`symbol`、`undefined`、`null`；除此之外，所有非原始值都是对象。函数是可调用对象，`typeof` 会为它返回特殊结果 `"function"`。

字符串、数字和布尔值等原始值在进行属性访问时会临时包装，因此可以调用原型方法：

```js
const text = "abc";
console.log(text.toUpperCase()); // "ABC"
console.log(text.length); // 3
```

这不表示 `text` 变成了可长期存放自有属性的对象。`null` 和 `undefined` 没有可供包装的对象，访问属性会抛出 `TypeError`。`typeof null === "object"` 是历史遗留行为，不能用来判断 `null` 是否为对象。

通常应避免显式包装对象：

```js
const primitive = "abc";
const wrapper = new String("abc");

console.log(typeof primitive); // "string"
console.log(typeof wrapper); // "object"
console.log(primitive === wrapper); // false
```

显式包装会带来比较、序列化和类型判断上的额外复杂度，字面量通常更合适。

### 属性键是字符串或 Symbol

普通对象的属性键只有两类：字符串键和 `Symbol` 键。数字下标会转换为字符串：

```js
const record = {};
record[3] = "three";

console.log(record["3"]); // "three"
```

`Symbol` 用于避免命名冲突或定义协议行为：

```js
const internalId = Symbol("id");
const user = { [internalId]: 42 };

console.log(user[internalId]); // 42
```

类的 `#private` 字段不属于普通对象属性键体系，不能通过 `obj["#field"]` 访问。

## 访问属性：`.`、`[]` 与函数属性

点操作符适用于静态、合法标识符形式的属性名；方括号适用于任意字符串键、`Symbol` 键和运行时计算出的键：

```js
const key = "display name";
const profile = {
  [key]: "Ada",
  age: 36,
};

console.log(profile.age); // 36
console.log(profile[key]); // "Ada"
```

JavaScript 没有独立于属性的“方法”类型。函数只是属性值；只有以 `obj.fn()` 形式调用时，普通函数才会获得 `obj` 作为 `this`：

```js
const counter = {
  value: 0,
  increment() {
    this.value += 1;
  },
};

const increment = counter.increment;
increment.call(counter);
```

## 数组是特殊对象，不是普通字典

数组也是对象，数组下标最终也是属性键。但符合数组索引条件的非负整数键会参与 `length` 计算：

```js
const items = ["a", "b", "c"];
items[5] = "f";

console.log(items.length); // 6
```

非索引自定义属性不会改变 `length`：

```js
items.label = "letters";
console.log(items.length); // 6
```

稀疏数组、混合值类型和把数组当成键值字典都可能损害可读性与运行时优化。顺序集合用数组；动态键值集合用对象或 `Map`。

## 复制对象：先明确需要什么语义

“深拷贝”不是单一的标准行为。选择前应先问：要复制哪些类型、是否允许循环引用、是否需要保留原型/描述符、是否希望触发 getter/setter？

### `Object.assign` 与对象展开：浅复制

```js
const source = {
  title: "Objects",
  options: { theme: "dark" },
};

const assigned = Object.assign({}, source);
const spread = { ...source };

assigned.options.theme = "light";
console.log(source.options.theme); // "light"
```

两种写法都只复制一层的自有可枚举字符串键和 `Symbol` 键；嵌套对象仍共享引用。它们读取源对象的 getter，复制得到的是值而不是原描述符。`Object.assign` 还会对目标对象执行普通赋值，因此可能触发目标的 setter。

### `structuredClone`：适合许多数据对象的深复制

```js
const original = {
  tags: ["js"],
  createdAt: new Date(),
};

const cloned = structuredClone(original);
cloned.tags.push("objects");

console.log(original.tags); // ["js"]
```

`structuredClone` 支持循环引用以及许多内建数据类型，例如 `Date`、`Map`、`Set`、`ArrayBuffer`。但函数、DOM 节点、`WeakMap`、`WeakSet` 等不可按结构化克隆算法复制；自定义类实例的原型和属性描述符也不会按“原样深拷贝”保留。使用前应验证目标运行时支持和所需类型。

### JSON 序列化只适合 JSON 数据

```js
const copy = JSON.parse(JSON.stringify({ name: "Ada", active: true }));
```

它只能处理 JSON 可表达的数据：函数、`undefined`、`Symbol` 会被忽略或转换，循环引用会抛错，`Date` 会变为字符串，`NaN` 和 `Infinity` 也不能保留原值。不要把它当成通用深拷贝方案。

## 属性描述符：数据属性与访问器属性

每个自有属性都有描述符。可通过 `Object.getOwnPropertyDescriptor()` 查看：

```js
const user = { name: "Ada" };
console.log(Object.getOwnPropertyDescriptor(user, "name"));
// { value: "Ada", writable: true, enumerable: true, configurable: true }
```

数据属性包含 `value`、`writable`、`enumerable`、`configurable`。通过对象字面量或普通赋值创建的属性，后三项默认都为 `true`；通过 `Object.defineProperty()` 新建属性时，未指定的布尔描述符默认是 `false`。

```js
Object.defineProperty(user, "id", {
  value: 1,
  enumerable: true,
});

console.log(user.id); // 1
```

上例的 `id` 不可写、不可配置。描述符可用来表达受控状态，但过度使用会增加维护成本。

### `writable`、`configurable` 与 `enumerable`

- `writable: false`：数据属性不能通过赋值修改；严格模式赋值会抛出 `TypeError`，非严格模式通常静默失败。
- `configurable: false`：不能删除该属性，也不能将其改为可配置或切换为访问器属性。若它仍可写，可修改值，并可将 `writable` 单向改为 `false`。
- `enumerable: false`：不会出现在 `for...in` 和 `Object.keys()` 中，但仍可直接访问，也可通过存在性检查发现。

访问器属性以 `get` / `set` 取代 `value` / `writable`：

```js
const temperature = {
  _celsius: 0,
  get fahrenheit() {
    return this._celsius * 1.8 + 32;
  },
  set fahrenheit(value) {
    this._celsius = (value - 32) / 1.8;
  },
};

temperature.fahrenheit = 68;
console.log(temperature._celsius); // 20
```

读取 getter、写入 setter 都是函数调用，可能有副作用或抛出异常；不应把它们当作普通字段读取。

## 对象完整性：防扩展、密封与冻结

这三个 API 约束的是对象**自身属性的结构**，都不是深冻结，也不是并发或安全机制。

| API | 新增属性 | 删除/重配属性 | 写入现有数据属性 |
| --- | --- | --- | --- |
| `Object.preventExtensions(obj)` | 禁止 | 允许 | 允许 |
| `Object.seal(obj)` | 禁止 | 禁止 | 允许（若原属性可写） |
| `Object.freeze(obj)` | 禁止 | 禁止 | 禁止（数据属性） |

```js
const settings = Object.freeze({
  theme: { name: "dark" },
});

settings.theme.name = "light"; // 仍可修改嵌套对象
```

`Object.freeze` 会使自有数据属性不可写、所有自有属性不可配置；访问器的 setter 仍可能改变它所引用的外部状态。需要深层不可变时，必须针对数据结构设计递归策略，并处理循环引用、特殊对象和性能成本。

## 属性存在性与遍历范围

属性读取结果为 `undefined` 不等于属性不存在：属性可能存在且值恰好是 `undefined`。需要判断存在性时，先决定是否包含原型链。

```js
const parent = { inherited: true };
const child = Object.create(parent);
child.own = undefined;

console.log("inherited" in child); // true，包含原型链
console.log(Object.hasOwn(child, "inherited")); // false
console.log(Object.hasOwn(child, "own")); // true
```

`Object.hasOwn()` 是现代首选。为了兼容旧环境或无原型对象，可使用：

```js
Object.prototype.hasOwnProperty.call(child, "own");
```

### 常用遍历 API

| API | 自有属性 | 可枚举 | 返回内容 |
| --- | --- | --- | --- |
| `for...in` | 否，含可枚举继承属性 | 是 | 字符串键 |
| `Object.keys(obj)` | 是 | 是 | 字符串键数组 |
| `Object.values(obj)` | 是 | 是 | 值数组 |
| `Object.entries(obj)` | 是 | 是 | `[key, value]` 数组 |
| `Object.getOwnPropertyNames(obj)` | 是 | 否，全部字符串键 | 字符串键数组 |
| `Object.getOwnPropertySymbols(obj)` | 是 | 否，全部 Symbol 键 | Symbol 键数组 |
| `Reflect.ownKeys(obj)` | 是 | 否，全部键 | 字符串与 Symbol 键数组 |

现代规范为自有属性键定义了稳定顺序：整数索引键升序、其余字符串按创建顺序、Symbol 键按创建顺序。`for...in` 还涉及原型链，不适合依赖枚举顺序；遍历对象数据通常优先使用 `Object.entries()`。

`for...of` 消费的是可迭代对象，不是“所有对象”的遍历语法。数组、字符串、`Map`、`Set` 等原生可迭代；普通对象默认没有 `Symbol.iterator`：

```js
for (const [key, value] of Object.entries({ name: "Ada" })) {
  console.log(key, value);
}
```

## 属性读取与赋值：原型链上的 `[[Get]]` / `[[Set]]`

规范使用内部操作描述属性访问。读取 `obj.key` 时，可以将 `[[Get]]` 简化为：先检查自身属性，未找到则沿 `[[Prototype]]` 向上；整条链都没有时得到 `undefined`。若找到 getter，会以接收者对象执行 getter。

写入 `obj.key = value` 对应更复杂的 `[[Set]]`。以下情形最值得掌握：

| 原型链与自身状态 | 赋值结果 |
| --- | --- |
| 自身存在可写数据属性 | 修改自身属性值 |
| 自身不存在，原型也没有该属性 | 在自身创建属性 |
| 原型有可写数据属性 | 通常在自身创建同名属性，遮蔽原型属性 |
| 原型有不可写数据属性 | 赋值失败；严格模式抛出 `TypeError` |
| 原型有 setter | 调用 setter，通常不会自动创建自身属性 |
| 原型只有 getter | 赋值失败；严格模式抛出 `TypeError` |

```js
const prototype = { role: "reader" };
const account = Object.create(prototype);

account.role = "author";

console.log(account.role); // "author"
console.log(prototype.role); // "reader"
```

这不是复制原型属性，而是在 `account` 上创建同名自身属性，称为**遮蔽**（shadowing）。后续原型章节会进一步讨论委托、遮蔽与构造函数的误区。

## 练习：复制与冻结能保证什么

```js
const source = {
  options: { theme: "dark" },
};

const copy = Object.assign({}, source);
const frozen = Object.freeze(source);

copy.options.theme = "light";
console.log(frozen.options.theme);
```

<details>
<summary>查看解析</summary>

输出 `"light"`。`Object.assign` 是浅复制，`copy.options` 与 `source.options` 指向同一个嵌套对象。`Object.freeze(source)` 只冻结了外层 `source`，没有冻结它引用的 `options` 对象，因此嵌套属性仍可被修改。

</details>

## 本章要点

1. 对象字面量是创建普通对象的默认选择；无原型字典可用 `Object.create(null)`。
2. 原始值有 7 种；属性键是字符串或 `Symbol`，数字键会转换为字符串。
3. 函数是对象属性值，是否获得对象 `this` 取决于调用形式。
4. `Object.assign`、对象展开都是浅复制；`structuredClone` 适合许多数据类型，JSON 只适合 JSON 数据。
5. 数据属性与访问器属性使用不同描述符；`Object.defineProperty` 的默认描述符通常为 `false`。
6. 防扩展、密封、冻结都只约束自身属性，且默认是浅层约束。
7. `Object.hasOwn` 只检查自身属性，`in` 还会查原型链；遍历前应先选择属性范围与键类型。
8. 属性读取沿原型链委托，属性写入会受自身描述符、原型数据属性和 setter 共同影响。

## 延伸阅读

- [MDN：对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN：Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [MDN：structuredClone()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/structuredClone)
- [MDN：属性描述符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
