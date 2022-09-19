---
title: Javscript数组快速填充数据的8种方法
excerpt: 如何快速造数据
categories:
- 技术文章
tags:
- js
---

## 前言
日常开发过程中经常会遇到模拟数据填充的问题。也就是造一些假数据，方便自己调试和开发。由此，整理了常用的数据填充的方法。

## fill()
fill() 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。
```javascript
let arr  = Array(10).fill('1')
//[ "1", "1", "1", "1", "1", "1", "1", "1", "1", "1" ]

let obj = Array(2).fill({name:'叫我詹躲躲',age:18})
//[{ name: "叫我詹躲躲", age: 18 },{ name: "叫我詹躲躲", age: 18 }]
```

## copyWithin()
copyWithin() 方法浅复制数组的一部分到同一数组中的另一个位置，并返回它，不会改变原数组的长度。填充的都是undefined.
**语法**
array.copyWithin(target, start, end)
**参数**

| 参数     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| *target* | 必需。复制到指定目标索引位置。                               |
| *start*  | 可选。元素复制的起始位置。                                   |
| *end*    | 可选。停止复制的索引位置 (默认为 *array*.length)。如果为负值，表示倒数。 |

```javascript
 const arr = [...Array(10)].copyWithin(0)
//[ undefined, undefined, undefined, undefined, undefined ]
```
还可以根据索引复制数组的值到另一个数组
```javascript
let arr = ['a', 'b', 'c', 'd', 'e'];
console.log(arr.copyWithin(0, 3, 4));
[ "d", "b", "c", "d", "e" ]
```

## keys()
keys() 方法用于从数组创建一个包含数组键的可迭代对象。 如果对象是数组返回 true，否则返回 false。
```javascript
//填充数组
const arr = [...Array(10).keys()]
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

## Array.from()
Array.from() 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
```javascript
Array.from(arrayLike[, mapFn[, thisArg]])
```
**参数**
- arrayLike想要转换成数组的伪数组对象或可迭代对象。
- mapFn 可选 如果指定了该参数，新数组中的每个元素会执行该回调函数。
- thisArg 可选 可选参数，执行回调函数 mapFn 时 this 对象。

**四种填充顺序数据方法（其他数据亦可）**
```javascript
const arr = Array.from(Array(10)).map((item,index)=>index)
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]

const arr = Array.from(Array(10), (item, index)=>index)
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]

const arr = Array.apply(null, Array(10)).map((item, index)=>index)
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]

const arr = Array.from({length:10},(item,index)=>index)
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

## map()
一个Map对象在迭代时会根据对象中元素的插入顺序来进行 — 一个 for...of 循环在每次迭代后会返回一个形式为[key，value]的数组。 使用map填充顺序数据
```javascript
const arr =[...Array(10)].map((item,index)=>index)
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

## Array.of()
Array.of() 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型
```javascript
const arr = Array.of(1, 2, 3);   
// [1, 2, 3]
Array.of() 和 Array 构造函数之间的区别在于处理整数参数：Array.of(7) 创建一个具有单个元素 7 的数组，而 Array(7) 创建一个长度为7的空数组（注意：这是指一个有7个空位(empty)的数组，而不是由7个undefined组成的数组）。

const arr = Array(7);          
// [ , , , , , , ]
const arr1 = Array(1, 2, 3);    
// [1, 2, 3]
```

## ArrayBuffer()
ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区。 有时候还会建立固定长度的原始二进制数据缓冲区。可以使用ArrayBuffer，它是一个字节数组。
```javascript
const buffer = new ArrayBuffer(8);

console.log(buffer.byteLength);
//8
console.log(buffer);
//{ byteLength: 8 }
```

## Object.keys()
Object.keys() 方法会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致 。
```javascript
let arr = Object.keys(Array.apply(null, {length:10})).map((item=>+item));
//[ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
创建值为String的数组

let arr = Object.keys([...Array(10)]);
//[ "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" ]
可以创建临时数据进行展示

let yData = Array.from({ length: 5 }, v => Math.random() * 10240)
 //[ 7513.437978671786, 5167.373983274039, 3814.0122504839223, 981.9482320596001, 4330.3850800180335 ]
 
let xData = Array.from({ length: 5 }, (v, w) => '叫我詹躲躲' + w)
//[ "叫我詹躲躲0", "叫我詹躲躲1", "叫我詹躲躲2", "叫我詹躲躲3", "叫我詹躲躲4" ]
```

## 参考链接
[Javscript数组快速填充数据的8种方法](https://zhuanlan.zhihu.com/p/267587176)
