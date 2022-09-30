---
title: ES6 - 箭头函数、箭头函数与普通函数的区别
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
这篇文章我们来了解一下ES6中的箭头函数。首先会介绍一下箭头函数的基本语法，因为基本语法比较好理解，我们用示例做简单介绍即可。之后，我们重点来讨论一下箭头函数与普通函数之间的区别。

## 基本语法
ES6中允许使用箭头=>来定义箭头函数，具体语法，我们来看一个简单的例子：
```javascript
// 箭头函数
let fun = (name) => {
    // 函数体
    return `Hello ${name} !`;
};

// 等同于
let fun = function (name) {
    // 函数体
    return `Hello ${name} !`;
};
```
可以看出，定义箭头函在数语法上要比普通函数简洁得多。箭头函数省去了function关键字，采用箭头=>来定义函数。函数的参数放在=>前面的括号中，函数体跟在=>后的花括号中。
关于箭头函数的参数：
1. 如果箭头函数没有参数，直接写一个空括号即可。
2. 如果箭头函数的参数只有一个，也可以省去包裹参数的括号。
3. 如果箭头函数有多个参数，将参数依次用逗号(,)分隔，包裹在括号中即可。

```javascript
// 没有参数
let fun1 = () => {
    console.log(111);
};

// 只有一个参数，可以省去参数括号
let fun2 = name => {
    console.log(`Hello ${name} !`)
};

// 有多个参数
let fun3 = (val1, val2, val3) => {
    return [val1, val2, val3];
};
```

关于箭头函数的函数体：
1. 如果箭头函数的函数体只有一句代码，就是简单返回某个变量或者返回一个简单的JS表达式，可以省去函数体的大括号{ }。
```javascript
let f = val => val;
// 等同于
let f = function (val) { return val };

let sum = (num1, num2) => num1 + num2;
// 等同于
let sum = function(num1, num2) {
  return num1 + num2;
};
let f = val => val;
// 等同于
let f = function (val) { return val };

let sum = (num1, num2) => num1 + num2;
// 等同于
let sum = function(num1, num2) {
  return num1 + num2;
};
```
2. 如果箭头函数的函数体只有一句代码，就是返回一个对象，可以像下面这样写：
```javascript
// 用小括号包裹要返回的对象，不报错
let getTempItem = id => ({ id: id, name: "Temp" });

// 但绝不能这样写，会报错。
// 因为对象的大括号会被解释为函数体的大括号
let getTempItem = id => { id: id, name: "Temp" };

```
3. 如果箭头函数的函数体只有一条语句并且不需要返回值（最常见是调用一个函数），可以给这条语句前面加一个void关键字
```javascript
let fn = () => void doesNotReturn();
```
箭头函数最常见的用处就是简化回调函数。
```javascript
// 例子一
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);

// 例子二
// 正常函数写法
var result = [2, 5, 1, 4, 3].sort(function (a, b) {
  return a - b;
});

// 箭头函数写法
var result = [2, 5, 1, 4, 3].sort((a, b) => a - b);

```

## 箭头函数与普通函数的区别
### 语法更加简洁、清晰
从上面的基本语法示例中可以看出，箭头函数的定义要比普通函数定义简洁、清晰得多，很快捷。

### 箭头函数不会创建自己的this
箭头函数没有自己的this，它会捕获自己在定义时（注意，是定义时，不是调用时）所处的外层执行环境的this，并继承这个this值。所以，箭头函数中this的指向在它被定义的时候就已经确定了，之后永远不会改变。
来看个例子：
```javascript
var id = 'Global';

function fun1() {
    // setTimeout中使用普通函数
    setTimeout(function(){
        console.log(this.id);
    }, 2000);
}

function fun2() {
    // setTimeout中使用箭头函数
    setTimeout(() => {
        console.log(this.id);
    }, 2000)
}

fun1.call({id: 'Obj'});     // 'Global'

fun2.call({id: 'Obj'});     // 'Obj'

```
上面这个例子，函数fun1中的setTimeout中使用普通函数，2秒后函数执行时，这时函数其实是在全局作用域执行的，所以this指向Window对象，this.id就指向全局变量id，所以输出'Global'。
但是函数fun2中的setTimeout中使用的是箭头函数，这个箭头函数的this在定义时就确定了，它继承了它外层fun2的执行环境中的this，而fun2调用时this被call方法改变到了对象{id: 'Obj'}中，所以输出'Obj'。
再来看另一个例子：
```javascript
var id = 'GLOBAL';
var obj = {
  id: 'OBJ',
  a: function(){
    console.log(this.id);
  },
  b: () => {
    console.log(this.id);
  }
};

obj.a();    // 'OBJ'
obj.b();    // 'GLOBAL'

```
上面这个例子，对象obj的方法a使用普通函数定义的，普通函数作为对象的方法调用时，this指向它所属的对象。所以，this.id就是obj.id，所以输出'OBJ'。
但是方法b是使用箭头函数定义的，箭头函数中的this实际是继承的它定义时所处的全局执行环境中的this，所以指向Window对象，所以输出'GLOBAL'。（这里要注意，定义对象的大括号{}是无法形成一个单独的执行环境的，它依旧是处于全局执行环境中！！）

### 箭头函数继承而来的this指向永远不变
上面的例子，就完全可以说明箭头函数继承而来的this指向永远不变。对象obj的方法b是使用箭头函数定义的，这个函数中的this就永远指向它定义时所处的全局执行环境中的this，即便这个函数是作为对象obj的方法调用，this依旧指向Window对象。

### .call()/.apply()/.bind()无法改变箭头函数中this的指向
.call()/.apply()/.bind()方法可以用来动态修改函数执行时this的指向，但由于箭头函数的this定义时就已经确定且永远不会改变。所以使用这些方法永远也改变不了箭头函数this的指向，虽然这么做代码不会报错。
```javascript
var id = 'Global';
// 箭头函数定义在全局作用域
let fun1 = () => {
    console.log(this.id)
};

fun1();     // 'Global'
// this的指向不会改变，永远指向Window对象
fun1.call({id: 'Obj'});     // 'Global'
fun1.apply({id: 'Obj'});    // 'Global'
fun1.bind({id: 'Obj'})();   // 'Global'

```

### 箭头函数不能作为构造函数使用
我们先了解一下构造函数的new都做了些什么？简单来说，分为四步：
1. JS内部首先会先生成一个对象；
2. 再把函数中的this指向该对象；
3. 然后执行构造函数中的语句；
4. 最终返回该对象实例。

但是！！因为箭头函数没有自己的this，它的this其实是继承了外层执行环境中的this，且this指向永远不会随在哪里调用、被谁调用而改变，所以箭头函数不能作为构造函数使用，或者说构造函数不能定义成箭头函数，否则用new调用时会报错！
```javascript
let Fun = (name, age) => {
    this.name = name;
    this.age = age;
};

// 报错
let p = new Fun('cao', 24);

```

### 箭头函数没有自己的arguments
箭头函数没有自己的arguments对象。在箭头函数中访问arguments实际上获得的是外层局部（函数）执行环境中的值。
```javascript
// 例子一
let fun = (val) => {
    console.log(val);   // 111
    // 下面一行会报错
    // Uncaught ReferenceError: arguments is not defined
    // 因为外层全局环境没有arguments对象
    console.log(arguments); 
};
fun(111);

// 例子二
function outer(val1, val2) {
    let argOut = arguments;
    console.log(argOut);    // 1
    let fun = () => {
        let argIn = arguments;
        console.log(argIn);     // 2
        console.log(argOut === argIn);  // 3
    };
    fun();
}
outer(111, 222);

```
上面例子二，1.2.3处的输出结果如下：

![]()

很明显，普通函数outer内部的箭头函数fun中的arguments对象，其实是沿作用域链向上访问的外层outer函数的arguments对象。
可以在箭头函数中使用rest参数代替arguments对象，来访问箭头函数的参数列表！！

### 箭头函数没有原型prototype
```javascript
let sayHi = () => {
    console.log('Hello World !')
};
console.log(sayHi.prototype); // undefined
```

### 不可以使用 yield 命令
不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

## 参考链接
[ES6 - 箭头函数、箭头函数与普通函数的区别](https://juejin.im/post/6844903805960585224)