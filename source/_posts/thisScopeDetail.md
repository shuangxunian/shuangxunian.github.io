---
title: javascript中的this作用域详解
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---

## 前言
Javascript中this的指向一直是困扰我很久的问题，在使用中出错的机率也非常大。在面向对象语言中，它代表了当前对象的一个引用，而在js中却经常让我觉得混乱，它不是固定不变的，而是随着它的执行环境的改变而改变。
在Javascript中this总是指向调用它所在方法的对象。因为this是在函数运行时，自动生成的一个内部对象，只能在函数内部使用。

## 分析
下面我们分几种情况深入分析this的用法：
1. 全局的函数调用
    ```javascript
    function globalTest() {
        this.name = "global this";
        console.log(this.name);
    }
    globalTest(); //global this
    ```
    以上代码中，globalTest()是全局性的方法，属于全局性调用，因此this就代表全局对象window。为了充分证明this是window,对代码做如下更改：
    ```javascript
    var name = "global this";

    function globalTest() {
        console.log(this.name);
    }
    globalTest(); //global this
    ```
    name作为一个全局变量，运行结果仍然是“global this”，说明this指向的是window。在方法体中我们尝试更改全局name，再次调用方法输出“rename global this”， 说明全局的name在方法内部被更改。代码如下：
    ```javascript
    var name = "global this";

    function globalTest() {
        this.name = "rename global this"
        console.log(this.name);
    }
    globalTest(); //rename global this
    ```
    根据以上三段代码，我们得出结论：对于全局的方法调用，this指向的是全局对象window,即调用方法所在的对象。
2. 对象方法的调用
    如果函数作为对象的方法调用，this指向的是这个上级对象，即调用方法的对象。 在以下代码中，this指向的是obj对象。
    ```javascript
    function showName() {
        console.log(this.name);
    }
    var obj = {};
    obj.name = "ooo";
    obj.show = showName;
    obj.show(); //ooo
    ```
3. 构造函数的调用
    构造函数中的this指向新创建的对象本身。
    ```javascript
    function showName() {
        this.name = "showName function";
    }
    var obj = new showName();
    console.log(obj.name); //showName function
    ```
    上述代码中，我们通过new关键字创建一个对象的实例，new关键字可以改变this的指向，将这个this指向对象obj。
    我们再增加一个全局的name,用以证明this指向的不是global:
    ```javascript
    var name = "global name";

    function showName() {
        this.name = "showName function";
    }
    var obj = new showName();

    console.log(obj.name); //showName function
    console.log(name); //global name
    ```
    在构造函数的内部，我们对this.name进行赋值，但并没有改变全局变量name。
4. apply/call调用时的this
    apply和call都是为了改变函数体内部的this指向。 其具体的定义如下：
    **call方法:**
    语法：call(thisObj，Object)
    定义：调用一个对象的一个方法，以另一个对象替换当前对象。
    说明：
    - call 方法可以用来代替另一个对象调用一个方法。call 方法可将一个函数的对象上下文从初始的上下文改变为由 thisObj 指定的新对象。
    - 如果没有提供 thisObj 参数，那么 Global 对象被用作 thisObj。

    **apply方法:**
    语法：apply(thisObj，[argArray])
    定义：应用某一对象的一个方法，用另一个对象替换当前对象。
    说明：
    - 如果 argArray 不是一个有效的数组或者不是 arguments 对象，那么将导致一个 TypeError。
    - 如果没有提供 argArray 和 thisObj 任何一个参数，那么 Global 对象将被用作 thisObj， 并且无法被传递任何参数。

    ```javascript
    var value = "Global value";

    function FunA() {
        this.value = "AAA";
    }

    function FunB() {
        console.log(this.value);
    }
    FunB(); //Global value 因为是在全局中调用的FunB(),this.value指向全局的value
    FunB.call(window); //Global value,this指向window对象，因此this.value指向全局的value
    FunB.call(new FunA()); //AAA, this指向参数new FunA()，即FunA对象

    FunB.apply(window); //Global value
    FunB.apply(new FunA()); //AAA
    ```
    在上述代码中，this的指向在call和apply中是一致的，只不过是调用参数的形式不一样。call是一个一个调用参数，而apply是调用一个数组。

## 参考链接
[javascript中的this作用域详解](https://www.cnblogs.com/lsgxeva/p/7975669.html)