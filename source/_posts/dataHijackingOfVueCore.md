---
title: Vue 核心之数据劫持
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
当前前端界空前繁荣，各种框架横空出世，包括各类mvvm框架横行霸道，比如Angular、Regular、Vue、React等等，它们最大的优点就是可以实现数据绑定，再也不需要手动进行DOM操作了，它们实现的原理也基本上是脏检查或数据劫持。那么本文就以Vue框架出发，探索作者运用Object.defineProperty来实现数据劫持的奥秘（本文所选取的相关代码源自于Vue v2.0.3版本的源码）。


## 回顾一下Object.defineProperty
语法
- Object.defineProperty(obj,prop,descriptor)

参数
- obj:目标对象
- prop:需要定义的属性或方法的名称
- descriptor:目标属性所拥有的特性

可供定义的特性列表
- value:属性的值
- writable:如果为false，属性的值就不能被重写。
- get: 一旦目标属性被访问就会调回此方法，并将此方法的运算结果返回用户。
- set:一旦目标属性被赋值，就会调回此方法。
- configurable:如果为false，则任何尝试删除目标属性或修改属性性以下特性（writable, configurable, enumerable）的行为将被无效化。
- enumerable:是否能在for...in循环中遍历出来或在Object.keys中列举出来。

## 什么是数据劫持
通过上面对Object.defineProperty的介绍，我们不难发现，当我们访问或设置对象的属性的时候，都会触发相对应的函数，然后在这个函数里返回或设置属性的值。既然如此，我们当然可以在触发函数的时候动一些手脚做点我们自己想做的事情，这也就是“劫持”操作。在Vue中其实就是通过Object.defineProperty来劫持对象属性的setter和getter操作，并“种下”一个监听器，当数据发生变化的时候发出通知。先简单的举个例子：
```javascript
var data = {
    name:'lhl'
}

Object.keys(data).forEach(function(key){
    Object.defineProperty(data,key,{
        enumerable:true,
        configurable:true,
        get:function(){
            console.log('get');
        },
        set:function(){
            console.log('监听到数据发生了变化');
        }
    })
})；
data.name //控制台会打印出 “get”
data.name = 'hxx' //控制台会打印出 "监听到数据发生了变化"
```
上面的这个例子可以看出，我们完全可以控制对象属性的设置和读取。在Vue中，作者在很多地方都非常巧妙的运用了Object.defineProperty这个方法，具体用在哪里并且它又解决了哪些问题，下面就做详细的介绍。

### 监听对象属性的变化
这个应该是Vue敲开数据绑定的前大门，它通过observe每个对象的属性，添加到订阅器dep中，当数据发生变化的时候发出一个notice。 相关源代码如下：（作者采用的是ES6+flow写的，代码在src/core/observer/index.js模块里面）
```javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  const dep = new Dep()//创建订阅对象

  const property = Object.getOwnPropertyDescriptor(obj, key)//获取obj对象的key属性的描述
  //属性的描述特性里面如果configurable为false则属性的任何修改将无效
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = observe(val)//创建一个观察者对象
  Object.defineProperty(obj, key, {
    enumerable: true,//可枚举
    configurable: true,//可修改
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val//先调用默认的get方法取值
      //这里就劫持了get方法，也是作者一个巧妙设计，在创建watcher实例的时候，通过调用对象的get方法往订阅器dep上添加这个创建的watcher实例
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          dependArray(value)
        }
      }
      return value//返回属性值
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val//先取旧值
      if (newVal === value) {
        return
      }
      //这个是用来判断生产环境的，可以无视
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = observe(newVal)//继续监听新的属性值
      dep.notify()//这个是真正劫持的目的，要对订阅者发通知了
    }
  })
}
```
以上是Vue监听对象属性的变化，那么问题来了，我们经常在传递数据的时候往往不是一个对象，很有可能是一个数组，那是不是就没有办法了呢，答案显然是否则的。那么下面就看看作者是如何监听数组的变化：

### 监听数组的变化
我们还看先看这段源码：
```javascript
const arrayProto = Array.prototype//原生Array的原型
export const arrayMethods = Object.create(arrayProto)

;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  const original = arrayProto[method]//缓存元素数组原型
  //这里重写了数组的几个原型方法
  def(arrayMethods, method, function mutator () {
    //这里备份一份参数应该是从性能方面的考虑
    let i = arguments.length
    const args = new Array(i)
    while (i--) {
      args[i] = arguments[i]
    }
    const result = original.apply(this, args)//原始方法求值
    const ob = this.__ob__//这里this.__ob__指向的是数据的Observer
    let inserted
    switch (method) {
      case 'push':
        inserted = args
        break
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})

...
//定义属性
function def (obj, key, val, enumerable) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  });
}
```
上面的代码主要是继承了Array本身的原型方法，然后又做了劫持修改，可以发出通知。Vue在observer数据阶段会判断如果是数组的话，则修改数组的原型，这样的话，后面对数组的任何操作都可以在劫持的过程中控制。结合Vue的思想，我简单的写个小demo方便更好的理解:
```javascript
var arrayMethod = Object.create(Array.prototype);
['push','shift'].forEach(function(method){
    Object.defineProperty(arrayMethod,method,{
        value:function(){
            var i = arguments.length
            var args = new Array(i)
            while (i--) {
              args[i] = arguments[i]
            }
            var original = Array.prototype[method];
            var result = original.apply(this,args);
            console.log("已经控制了，哈哈");
            return result;
        },
        enumerable: true,
        writable: true,
        configurable: true
    })
})
var bar = [1,2];
bar.__proto__ = arrayMethod;
bar.push(3);//控制台会打印出 “已经控制了，哈哈”;并且bar里面已经成功的添加了成员 ‘3’ 
```
整个过程看起来好像没有什么问题，似乎Vue已经做到了完美，其实不然，Vue还是不能检测到数据项和数组长度改变的变化，例如下面的调用：
```javascript
vm.items[index] = "xxx";
vm.items.length = 100;
```
我们尽量避免这样的调用方式，如果确实需要，作者也帮我们实现了一个$set操作，这里就不做介绍了。

### 实现对象属性代理
正常情况下我们是这样实例化一个Vue对象:
```javascript
var VM = new Vue({
    data:{
        name:'lhl'
    },
    el:'#id'
})
```
按理说我们操作数据的时候应该是VM.data.name = ‘hxx’才对，但是作者觉得这样不够简洁，所以又通过代理的方式实现了VM.name = ‘hxx’的可能。 相关代码如下：
```javascript
function proxy (vm, key) {
  if (!isReserved(key)) {
    Object.defineProperty(vm, key, {
      configurable: true,
      enumerable: true,
      get: function proxyGetter () {
        return vm._data[key]
      },
      set: function proxySetter (val) {
        vm._data[key] = val;
      }
    });
  }
}
```
表面上看起来我们是在操作VM.name，实际上还是通过Object.defineProperty()中的get和set方法劫持实现的。

## 总结
Vue框架很好的利用了Object.defineProperty()这个方法来实现了数据的双向绑定，同时也达到了很好的模块间解耦，在日常开发中，你也可以用好这个方法来优化对象获取和修改属性方式，或者自己实现一个MVVM的双向数据绑定等。

## 参考链接
[Vue 核心之数据劫持](https://juejin.cn/post/6844903463285948423)