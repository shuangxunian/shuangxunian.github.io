---
title: 关于js中的get、set方法
excerpt: 如题目~
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 什么是getter/setter？
getter 是一种获得属性值的方法，setter是一种设置属性值的方法。getter负责查询值，它不带任何参数，setter则负责设置键值，值是以参数的形式传递，在他的函数体中，一切的return都是无效的。和普通属性不同的是，存储器属性在只声明了get或set时，对于读和写是两者不可兼得的，当它只拥有了getter方法，那么它仅仅只读，同样的，当它只有setter方法，那么您读到的永远都是undefined。

## 如何定义？
一共有五种：
1. 通过对象初始化器定义
2. 使用 Object.create 方法
3. 使用 Object.defineProperty 方法
4. 使用 Object.defineProperties 方法
5. 使用 `Object.prototype.__defineGetter__` 以及 `Object.prototype.__defineSetter__` 方法

### 通过对象初始化器定义
通过对象初始化器在创建对象的时候指明（也可以称为通过字面值创建对象时声明）
```javascript
(function () {
    var o = {
        a : 7,
        get b(){return this.a +1;},//通过 get,set的 b,c方法间接性修改 a 属性
        set c(x){this.a = x/2}
    };
    console.log(o.a);
    console.log(o.b);
    o.c = 50;
    console.log(o.a);
})();
```
在 chrome 中调试视图如下：

![](https://api2.mubu.com/v3/document_image/6b6fe72a-7efc-4115-8bc3-26ecb0e4ac36-3807603.jpg)

可以看到对象下多了 get 属性以及 set 属性
输出结果如下：

![](https://api2.mubu.com/v3/document_image/bd098c8b-b45f-4b9c-a509-9f947c5e4b91-3807603.jpg)

当然 get 语句与 set 语句可以声明多次用来对应多个 getter 和 setter
使用这种方法的好处是可以在声明属性的时候同时声明对应的 getter 和 setter
这里就有人问了，能不能将o 对象的 get 及 set 方法的方法名都改成 “a”,这样就可以直接通过“.”来访问方法直接操作
```javascript
(function () {
    var o = {
        a : 7,
        get a(){return this.a +1;},//死循环
        set a(x){this.a = x/2}
    };
    console.log(o.a);
    console.log(o.b);
    o.c = 50;
    console.log(o.a);
})();
```
打开 chrome 查看创建后的视图如下：

![](https://api2.mubu.com/v3/document_image/a37164b9-d841-4930-a1d3-4bf594c8b44c-3807603.jpg)

可以看到这个时候的 get 与 set 方法已经和上面不同，但是是否真的能起作用呢，答案是否定的，当我们通过 o.a 调用的是 get语句 声明的 a方法，进入到该方法后遇到 this.a 方法继续调用该方法形成死循环最终导致死循环报内存溢出错误。

新语法(ES6)：暂时只有 firefox 支持，其他浏览器会报错
```javascript
(function () {
    var b = "bb";
    var c = "cc";
    var o = {
        a : 7,
        get [b](){return this.a +1;},
        set [c](x){this.a = x/2},
    };
    console.log(o.a);
    console.log(o[b]);
    o["cc"] = 50;
    console.log(o.a);
})();
```
打开 firefox 查看调试：

![](https://api2.mubu.com/v3/document_image/ce0feee3-8645-46e9-b727-2d97e981ce30-3807603.jpg)

输出结果如下：

![](https://api2.mubu.com/v3/document_image/d9b2c35c-ba73-4a8f-96c4-f9f6403cb762-3807603.jpg)

### 使用 Object.create 方法
引用 MDN：
- 概述
Object.create() 方法创建一个拥有指定原型和若干个指定属性的对象。
- 语法
Object.create(proto, [ propertiesObject ])

我们都知道使用 Object.create 方法传递一个参数的时候可以创建一个以该参数为原型的对象 
第二个参数是可选项，是一个匿名的参数对象，该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符（包扩数据描述符或存取描述符，具体解释看后面的内容 什么是属性描述符）。
通过属性描述符我们可以实现为新创建的对象添加 get 方法以及 set 方法
```javascript
(function () {
    var o = null;
    o = Object.create(Object.prototype,//指定原型为 Object.prototype
            {
                bar:{
                    get :function(){
                        return 10;
                    },
                    set : function (val) {
                        console.log("Setting `o.bar` to ",val);
                    }
                }
            }//第二个参数
        );
    console.log(o.bar);
    o.bar = 12;
})();
```
在 chrome 中调试试图如下：

![](https://api2.mubu.com/v3/document_image/abd7b0a4-d56b-4cd4-9421-1fae111ce75e-3807603.jpg)

可以看到新创建对象通用多了 get 以及 set 属性
输出结果如下：

![](https://api2.mubu.com/v3/document_image/14523fed-da76-4c78-8612-9fde21f7ec3f-3807603.jpg)

上面这个例子并没有用来针对的 get 方法以及 set 方法使用的属性
```javascript
(function () {
    var o = null;
    o = Object.create(Object.prototype,//指定原型为 Object.prototype
            {
                bar:{
                    get :function(){
                        return this.a;
                    },
                    set : function (val) {
                        console.log("Setting `o.bar` to ",val);
                        this.a = val;
                    },
                    configurable :true
                }
            }//第二个参数
        );
    o.a = 10;
    console.log(o.bar);
    o.bar = 12;
    console.log(o.bar);
})();
```
亦或：
```javascript
(function () {
    var o = {a:10};
    o = Object.create(o,//指定原型为 o 这里实际可以理解为继承
            {
                bar:{
                    get :function(){
                        return this.a;
                    },
                    set : function (val) {
                        console.log("Setting `o.bar` to ",val);
                        this.a = val;
                    },
                    configurable :true
                }
            }//第二个参数
        );
    console.log(o.bar);
    o.bar = 12;
    console.log(o.bar);
})();
```
输出结果如下：

![](https://api2.mubu.com/v3/document_image/4bb329de-371a-46cb-b684-3bd53c9dd7f4-3807603.jpg)

使用这种方式的好处是可配置性高，但初学者容易迷糊。

### 使用 Object.defineProperty 方法
引用 MDN：
- 概要
Object.defineProperty() 方法直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。
- 语法
Object.defineProperty(obj, prop, descriptor)
- 参数
    - obj
    需要定义属性的对象。
    - prop
    需被定义或修改的属性名。
    - descriptor
    需被定义或修改的属性的描述符。

```javascript
(function () {
    var o = { a : 1}//声明一个对象,包含一个 a 属性,值为1
    Object.defineProperty(o,"b",{
        get: function () {
            return this.a;
        },
        set : function (val) {
            this.a = val;
        },
        configurable : true
    });

    console.log(o.b);
    o.b = 2;
    console.log(o.b);
})();
```
这个方法与前面两种的区别是：使用前面两种只能在声明定义的时候指定 getter 与 setter，使用该方法可以随时的添加或修改。
如果说需要一次性批量添加 getter 与 setter 也是没问题的，使用如下方法：

### 使用 Object.defineProperties方法
引用 MDN：
- 概述
Object.defineProperties() 方法在一个对象上添加或修改一个或者多个自有属性，并返回该对象。
- 语法
Object.defineProperties(obj, props)
- 参数
    - obj
    将要被添加属性或修改属性的对象
    - props
    该对象的一个或多个键值对定义了将要为对象添加或修改的属性的具体配置

不难看出用法与 Object.defineProperty 方法类似
```javascript
(function () {
    var obj = {a:1,b:"string"};
    Object.defineProperties(obj,{
        "A":{
            get:function(){return this.a+1;},
            set:function(val){this.a = val;}
        },
        "B":{
            get:function(){return this.b+2;},
            set:function(val){this.b = val}
        }
    });

    console.log(obj.A);
    console.log(obj.B);
    obj.A = 3;
    obj.B = "hello";
    console.log(obj.A);
    console.log(obj.B);
})();
```
输出结果如下：

![](https://api2.mubu.com/v3/document_image/3f84aefa-3c7a-47f5-ac49-ba8a8cb3a2d1-3807603.jpg)

### 使用 Object.prototype.__defineGetter__ 以及 Object.prototype.__defineSetter__ 方法
```javascript
(function () {
    var o = {a:1};
    o.__defineGetter__("giveMeA", function () {
        return this.a;
    });
    o.__defineSetter__("setMeNew", function (val) {
        this.a  = val;
    })
    console.log(o.giveMeA);
    o.setMeNew = 2;
    console.log(o.giveMeA);
})();
输出结果为1和2
```
查看 MDN 有如下说明：

![](https://api2.mubu.com/v3/document_image/0d61e32b-d6f7-41f9-aaf8-4248e3926033-3807603.jpg)

## 什么是属性描述符
引用 MDN：
对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。

即：
1. 数据描述符是一个拥有可写或不可写值的属性。
2. 存取描述符是由一对 getter-setter 函数功能来描述的属性。
3. 描述符必须是两种形式之一；不能同时是两者。

数据描述符和存取描述符均具有以下可选键值：
- configurable
当且仅当这个属性描述符值为 true 时，该属性可能会改变，也可能会被从相应的对象删除。默认为 false。
- enumerable  
true 当且仅当该属性出现在相应的对象枚举属性中。默认为 false。

数据描述符同时具有以下可选键值：
- value 
与属性相关的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。
- writable 
true 当且仅当可能用 赋值运算符 改变与属性相关的值。默认为 false。

存取描述符同时具有以下可选键值：
- get 
一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。方法将返回用作属性的值。默认为 undefined。
- set
一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将收到作为唯一参数的新值分配给属性。默认为 undefined。

以上是摘自MDN的解释，看起来是很晦涩的，具体什么意思呢：
首先我们从以上解释知道该匿名参数对象有个很好听的名字叫属性描述符，属性描述符又分成两大块：数据描述符以及存取描述符（其实只是一个外号，给指定的属性集合起个外号）。

数据描述符包括两个属性 : value 属性以及 writable 属性，第一个属性用来声明当前欲修饰的属性的值，第二个属性用来声明当前对象是否可写即是否可以修改

存取描述符就包括 get 与 set 属性用来声明欲修饰的象属性的 getter 及 setter

属性描述符内部，数据描述符与存取描述符只能存在其中之一，但是不论使用哪个描述符都可以同时设置 configurable 属性以及enumerable 属性。
configurable属性用来声明欲修饰的属性是否能够配置，仅有当其值为 true 时，被修饰的属性才有可能能够被删除，或者重新配置。
enumerable 属性用来声明欲修饰属性是否可以被枚举。

知道了什么是属性描述符，我们就可以开始着手创建一些对象并开始配置其属性

## 创建属性不可配置不可枚举的对象
```javascript
//使用默认值配置
(function () {
    var obj = {};//声明一个空对象
    Object.defineProperty(obj,"key",{
        value:"static"
                        //没有设置 enumerable 使用默认值 false
                        //没有 configurable 使用默认值 false
                        //没有 writable 使用默认值 false
    });

    console.log(obj.key);           //输出 “static”
    obj.key = "new"                 //尝试修改其值,修改将失败,因为 writable 为 false
    console.log(obj.key);           //输出 “static”
    obj.a = 1;//动态添加一个属性
    for(var item in obj){ //遍历所有 obj 的可枚举属性
         console.log(item);
    }//只输出一个 “a” 因为 “key”的 enumerable为 false
})();
```
```javascript
//显示配置 等价于上面
(function () {
    var obj = {};
    Object.defineProperty(obj,"key",{
        enumerable : false,
        configurable : false,
        writable : false,
        value : "static"
    })
})();
```
```javascript
//等价配置
(function () {
    var o = {};
    o.a = 1;
    //等价于
    Object.defineProperty(o,"a",{value : 1,
                                writable : true,
                                configurable : true,
                                enumerable : true});
    
    Object.defineProperty(o,"a",{value :1});
    //等价于
    Object.defineProperty(o,"a",{value : 1,
                                writable : false,
                                configurable : false,
                                enumerable : false});
})();
```

## Enumerable 特性
属性特性 enumerable 决定属性是否能被 for...in 循环或 Object.keys 方法遍历得到
```javascript
(function () {
    var o = {};
    Object.defineProperty(o,"a",{value :1,enumerable :true});
    Object.defineProperty(o,"b",{value :2,enumerable :false});
    Object.defineProperty(o,"c",{value :2});//enumerable default to false
    o.d = 4;//如果直接赋值的方式创建对象的属性,则这个属性的 enumerable 为 true

    for(var item in o){ //遍历所有可枚举属性包括继承的属性
        console.log(item);
    }

    console.log(Object.keys(o));//获取 o 对象的所有可遍历属性不包括继承的属性

    console.log(o.propertyIsEnumerable('a'));//true
    console.log(o.propertyIsEnumerable('b'));//false
    console.log(o.propertyIsEnumerable('c'));//false
})();
```
输出结果如下：

![](https://api2.mubu.com/v3/document_image/fa4d422f-c613-48ab-93dd-4b7cf8194fb9-3807603.jpg)

## Configurable 特性
```javascript
(function () {
    var o = {};
    Object.defineProperty(o,"a",{get: function () {return 1;},
                                configurable : false} );
                                //enumerable 默认为 false,
                                //value 默认为 undefined,
                                //writable 默认为 false,
                                //set 默认为 undefined
                                  
    //抛出异常,因为最开始定义了 configurable 为 false,故后期无法对其进行再配置
    Object.defineProperty(o,"a",{configurable : true} );
    //抛出异常,因为最开始定义了 configurable 为 false,故后期无法对其进行再配置,enumerable 的原值为 false
    Object.defineProperty(o,"a",{enumerable : true} );
    //抛出异常,因为最开始定义了 configurable 为 false,set的原值为 undefined
    Object.defineProperty(o,"a",{set : function(val){}} );
    //抛出异常,因为最开始定义了 configurable 为 false,故无法进行覆盖,尽管想用一样的来覆盖
    Object.defineProperty(o,"a",{get : function(){return 1}});
    //抛出异常，因为最开始定义了 configurable 为 false,故无法将其进行重新配置把属性描述符从存取描述符改为数据描述符
    Object.defineProperty(o,"a",{value : 12});

    console.log(o.a);//输出1
    delete o.a;      //想要删除属性,将失败
    console.log(o.a);//输出1
    
})();
```

## 提高及扩展
1. 属性描述符中容易被误导的地方之 writable 与 configurable
```javascript
(function () {
    var o = {};
    Object.defineProperties(o,{
        "a": {
            value:1,
            writable:true,//可写
            configurable:false//不可配置
            //enumerable 默认为 false 不可枚举
        },
        "b":{
            get :function(){
                return this.a;
            },
            configurable:false
        }
    });
    console.log(o.a);   //1
    o.a = 2;            //修改值成功,writable 为 true
    console.log(o.a);   //2
    Object.defineProperty(o,"a",{value:3});//同样为修改值成功
    console.log(o.a);   //3

    //将其属性 b 的属性描述符从存取描述符重新配置为数据描述符
    Object.defineProperty(o,"b",{value:3});//抛出异常,因为 configurable 为 false
})();
```
2. 通过上面的学习，我们都知道传递属性描述符参数时，是定义一个匿名的对象，里面包含属性描述符内容，若每定义一次便要创建一个匿名对象传入，将会造成内存浪费。故优化如下：
```javascript
(function () {
    var obj = {};

    //回收同一对象,即减少内存浪费
    function withValue(value){
        var d = withValue.d ||(
            withValue.d = {
                enumerable : false,
                configurable : false,
                writable : false,
                value :null
            }
            );
        d.value = value;
        return d;
    }
    Object.defineProperty(obj,"key",withValue("static"))
})();
```

## 参考链接
[关于js中的get、set方法](https://blog.csdn.net/u010403387/article/details/44238539)
[浅谈 JS 对象添加 getter与 setter 的5种方法以及如何让对象属性不可配置或枚举](https://segmentfault.com/a/1190000003882976)