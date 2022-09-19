---
title: undefined与null
excerpt: 解释一下undefined与null的区别
categories:
- 技术文章
tags:
- js
---

## 前言
Undefined 和 Null 是 Javascript 中两种特殊的原始数据类型(Primary Type)，它们都只有一个值，分别对应 undefined 和 null ，下面就来详细说说两者的异同。

## undefined 
undefined 的字面意思就是：未定义的值 。这个值的语义是，希望表示一个变量最原始的状态，而非人为操作的结果 。 这种原始状态会在以下 4 种场景中出现：

### 声明了一个变量，但没有赋值

```javascript
var foo;
console.log(foo); // undefined
```

访问 foo，返回了 undefined，表示这个变量自从声明了以后，就从来没有使用过，也没有定义过任何有效的值，即处于一种原始而不可用的状态。

### 访问对象上不存在的属性

```javascript
console.log(Object.foo); // undefined
var arr = [];
console.log([0]); // undefined
```

访问 Object 对象上的 foo 属性，返回 undefined ， 表示Object 上不存在或者没有定义名为 foo 的属性。数组中的元素在内部也属于对象属性，访问下标就等于访问这个属性，返回 undefined ，就表示数组中不存在这个元素。

### 函数定义了形参，但没有传递实参

```javascript
//函数定义了形参 a
function fn(a) {
    console.log(a); // undefined
}
fn(); //未传递实参
```
函数 fn 定义了形参 a， 但 fn 被调用时没有传递参数，因此，fn 运行时的参数 a 就是一个原始的、未被赋值的变量。

### 使用 void 对表达式求值

```javascript
void 0 ; // undefined
void false; // undefined
void []; // undefined
void null; // undefined
void function fn(){} ; // undefined
```

ECMAScript 明确规定 void 操作符 对任何表达式求值都返回 undefined ，这和函数执行操作后没有返回值的作用是一样的，JavaScript 中的函数都有返回值，当没有 return 操作时，就默认返回一个原始的状态值，这个值就是 undefined，表明函数的返回值未被定义。

因此，undefined 一般都来自于某个表达式最原始的状态值，不是人为操作的结果。当然，你也可以手动给一个变量赋值 undefined，但这样做没有意义，因为一个变量不赋值就是 undefined 。

## null
null 的字面意思是：空值。这个值的语义是，希望表示一个对象被人为的重置为空对象，而非一个变量最原始的状态。在内存里的表示就是，栈中的变量没有指向堆中的内存对象，即：

![](https://api2.mubu.com/v3/document_image/d3f2bd56-fb43-4b9f-aaa3-0635e94a0525-3807603.jpg)

当一个对象被赋值了null 以后，原来的对象在内存中就处于游离状态，GC 会择机回收该对象并释放内存。因此，如果需要释放某个对象，就将变量设置为 null，即表示该对象已经被清空，目前无效状态。试想一下，如果此处把 null 换成 undefined 会不会感到别扭? 显然语义不通，其操作不能正确的表达其想要的行为。

与 null 相关的另外一个问题需要解释一下：

```javascript
typeof null== 'object' 
```

null 有属于自己的类型 Null，而不属于Object类型，typeof 之所以会判定为 Object 类型，是因为JavaScript 数据类型在底层都是以二进制的形式表示的，二进制的前三位为 0 会被 typeof 判断为对象类型，而 null 的二进制位恰好都是 0 ，因此，null 被误判断为 Object 类型。

```
000 - 对象，数据是对象的应用
1 - 整型，数据是31位带符号整数
010 - 双精度类型，数据是双精度数字
100 - 字符串，数据是字符串
110 - 布尔类型，数据是布尔值
```

其实，我们可以通过另一种方法获取 null 的真实类型：

```javascript
Object.prototype.toString.call(null) ; // [object Null]
```

通过 Object 原型上的toString() 方法可以获取到JavaScript 中对象的真实数据类型，当然 undefined 类型也可以通过这种方式来获取：

```javascript
Object.prototype.toString.call(undefined) ; // [object Undefined]
```

## 相似性
虽然 undefined 和 null 的语义和场景不同，但总而言之，它们都表示的是一个**无效的值**。 因此，在JS中对这类值访问属性时，都会得到异常的结果：

```javascript
null.toString(); // Cannot read property 'toString' of null
undefined.toString(); // Cannot read property 'toString' of undefined　　
```

ECMAScript 规范认为，既然 null 和  undefined 的行为很相似，并且都表示 一个无效的值，那么它们所表示的内容也具有相似性，即有

```javascript
undefined == null; // true
```

不要试图通过转换数据类型来解释这个结论，因为：

```javascript
Number(null); // 0
Number(undefined); // NaN
// 在比较相等性之前，null 没有被转换为其他类型
null== 0 ; //false
```

但 === 会返回 false ，因为全等操作 === 在比较相等性的时候，不会主动转换分项的数据类型，而两者又不属于同一种类型：

```javascript
undefined === null; // false，类型不相同
undefined !== null;  // true, 类型不相同
```

## 总结
用一句话总结两者的区别就是：undefined 表示一个变量自然的、最原始的状态值，而 null 则表示一个变量被人为的设置为空对象，而不是原始状态。所以，在实际使用过程中，为了保证变量所代表的语义，不要对一个变量显式的赋值 undefined，当需要释放一个对象时，直接赋值为 null 即可。

## 参考链接
[详解 undefined 与 null 的区别](https://www.cnblogs.com/onepixel/p/7337248.html)