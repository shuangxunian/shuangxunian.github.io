---
title: JS数据类型
excerpt: JS的8种数据类型
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## JS数据类型：基础概念 
请注意：JS的数据类型有8种。

在ES5的时候，我们认知的数据类型确实是 6种：Number、String、Boolean、undefined、object、Null。

ES6 中新增了一种 Symbol 。这种类型的对象永不相等，即始创建的时候传入相同的值，可以解决属性名冲突的问题，做为标记。

谷歌67版本中还出现了一种 bigInt。是指安全存储、操作大整数。（但是很多人不把这个做为一个类型）。

## JS数据类型：JS 的数据类型有几种？
8种。Number、String、Boolean、Null、undefined、object、symbol、bigInt。

## JS数据类型：Object 中包含了哪几种类型？
其中包含了Data、function、Array等。这三种是常规用的。

## JS数据类型：JS的基本类型和引用类型有哪些呢？
基本类型（单类型）：除Object。 String、Number、boolean、null、undefined。
引用类型：object。里面包含的 function、Array、Date。

## JS数据类型：JS 中 typeof 输出分别是什么？

|           |           |                      |
| --------- | --------- | -------------------- |
| String    | String    | 字符串               |
| Number    | Number    | 数据类型             |
| Boolean   | Boolean   | 布尔型               |
| Undefined | Undefined | 没有初始化、定义的值 |
| Null      | Object    | 不存在的对象         |
| NaN       | Number    | Number 中的特殊数值  |
| Object    | Function  |                      |

##  Number('as') == NaN ?
肯定是false， Number(‘as’) 输出 NaN。
注：用 isNaN（） 检测是否是非数值型。 js 规定的NaN 不等于NaN。

## Number('123') == NaN ？
肯定是false，Number('123’) 输出 123。

## JS数据类型：如何判断数据类型？
1. typeof 操作符（通用：上面有内容有讲到）
2. toString()
    作用：其他类型转成 string 的方法
    支持：number、boolean、string、object 
    不支持：null 、undefined
3. toLocaleString
    把数组转成本地字符串
4. 检测数组类型的方法
    - instanceof  操作符
    - 对象的 constructor 属性
    - Array.isArray()检验值是否为数组

## JS数据类型：null 和 undefined 有什么区别？
Null 只有一个值，是 null。不存在的对象。
Undefined 只有一个值，是undefined。没有初始化。undefined 是从 null 中派生出来的。
简单理解就是：undefined 是没有定义的，null 是定义了但是为空。

## JS数据类型：null 不存在的原因是什么？如何解决？
不存在的原因是：
    1. 方法不存在
    2. 对象不存在
    3. 字符串变量不存在
    4. 接口类型对象没初始化 

解决方法：
    做判断处理的时候，放在设定值的最前面

## JS数据类型：== 和 === 有什么区别，什么场景下使用？
- ==表示相同。
    比较的是物理地址，相当于比较两个对象的 hashCode ，肯定不相等的。
    类型不同，值也可能相等。
- ===表示严格相同。
    例：同为 null／undefined ，相等。

简单理解就是 == 就是先比较数据类型是否一样。=== 类型不同直接就是 false。

## JS数据类型：对象可以比较地？
对象是可以比较，遍历比较key 和 value就行， Object.is(value1, value2)。

## JS数据类型：总结
1. Undefined 类型
    只有一个值。在使用var 声明变量但未对其加初始化时，这个变量就是undefined。
2. Null 类型
    只有一个值。null是表示一个空对象指针，这也是typeof操作符检测 null 值时会返回 object 的原因。
3. Boolean 类型
    使用最多的一个类型，有两个字面值，分别是true、false。true不一定等于1,false不一定等于0。
    boolean类型的字面值是区分大小写的。True和False是标识符
4. Number 类型
    数字类型，表示数据的整数和浮点数。某些语言中也称为“双精度值”。
    ```javascript
    var intNum = 55;    //十进制
    var num = 012;      //八进制
    var octalNum = 0x23;//十六进制
    ```
5. String 类型
    字符串可以有单引号、双引号表示。字符串是不可变的，一旦创建，值就不能改变
    要改变某个变量保存的字符串，首先要销毁原来的字符串，然后于用另一个包含的字符串填充该变量。
    注）toString()可以输出二进制、八进制、十进制，十六进制。null和undefined没有toString()方法，用String函数不返回这两个值的字面量。
6. Object 类型
    ECMAjavascript中的对象其实就是一组数据和功能的集合。对象可以通过执行new操作符后跟要创建的对象类型的名称来创建。创建object类型的实例并为其添加属性（或）方法，就可以自定义创建对象。如：var o = new Object( );
    object 的每个实例都有下列属性和方法：
    - constructor：保存着用于创建当前对象的函数。（构造函数)constructor就是object();
    - hasOwnProperty(propertyName):用于检查给定的当前属性在当前对象实例中）而不是在实例原型中）是否存在。其中，作为参数的属性名（propertyName)必须以字符串形式指定（例如：o.hasOwnProperty(“name”))。
    - isPrototypeOf(object):用于检查传入的对象是否是传入对象原型。
    - propertyIsEnumerable(propertyName):用于检查给定属性是否能够用for-in语句。与hasOwnProperty（）方法一样，作为参数的属性名必须以字符串形式指定。
    - toLocaleString( ):返回对象的字符串表示，该字符串与执行环境的地区对应。
    - toString( ):返回对象的字符串表示。
    - valueOf( ):返回对象的字符串、数值或者布尔值表示。通常与toString( )方法的返回值得相同。

    ECMAJS中object是所有对象的基础，因些所有对象都具有这些基本的属性和方法。
7. Symbol 类型
    Symbol 类型的对象永远不相等，即便创建的时候传入相同的值。因此，可以用解决属性名冲突的问题（适用于多少编码），做为标记。
    这是 es6 新增的数据类型。
8. BigInt 类型
    Javascript 中的任意精度整数，可以安全存储和操作大整数。即始超出 Number 能够表示的安全整数范围。是 chrome 67中的新功能。

## 参考链接
[JS基础篇1：数据类型（8种）](https://blog.csdn.net/u013592575/article/details/95087953)