---
title: JavaScript实现类与继承的方法
excerpt: 关于js类与继承方法实现的文章
categories:
- 技术文章
tags:
- js
---

## JavaScript定义类的4种方法
### 工厂方法
```javascript
function creatPerson(name, age) {
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sayName = function() {
        window.alert(this.name);
    };
    return obj;
}
```

### 构造函数方法
```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.sayName = function() {
        window.alert(this.name);
    };
}
```

### 原型方法
```javascript
function Person() {}
Person.prototype = {
    constructor : Person,
    name : "Ning",
    age : "23",
    sayName : function() {
        window.alert(this.name);
    }
};
```
大家可以看到这种方法有缺陷，类里属性的值都是在原型里给定的。

### 组合使用构造函数和原型方法（使用最广）
```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
}      
Person.prototype = {
    constructor : Person, 
    sayName : function() {
        window.alert(this.name);
    }
};
```
将构造函数方法和原型方法结合使用是目前最常用的定义类的方法。这种方法的好处是实现了属性定义和方法定义的分离。比如我可以创建两个对象person1和person2，它们分别传入各自的name值和age值，但sayName()方法可以同时使用原型里定义的。

## JavaScript实现继承的3种方法
### 借用构造函数法（又叫经典继承）
```javascript
function SuperType(name) {
    this.name = name;
    this.sayName = function() {
        window.alert(this.name);
    };
}
function SubType(name, age) {
    SuperType.call(this, name); //在这里借用了父类的构造函数
    this.age = age;
}
```

### 对象冒充
```javascript
function SuperType(name) {
    this.name = name;
    this.sayName = function() {
        window.alert(this.name);
    };
}
function SubType(name, age) {
    this.supertype = SuperType; //在这里使用了对象冒充
    this.supertype(name);
    this.age = age;
}
```

### 组合继承（最常用）
```javascript
function SuperType(name) {
    this.name = name;
}
SuperType.prototype = {
    sayName : function() {
        window.alert(this.name);
    }
};
function SubType(name, age) {
    SuperType.call(this, name); //在这里继承属性
    this.age = age;
}
SubType.prototype = new SuperType(); //这里继承方法
```
组合继承的方法是对应着我们用‘组合使用构造函数和原型方法’定义父类的一种继承方法。同样的，我们的属性和方法是分开继承的。

## 参考链接
[JavaScript实现类与继承的方法（全面整理）](https://segmentfault.com/a/1190000013253890)