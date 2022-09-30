---
title: 实现js的clone函数
excerpt: 实现一个函数clone，使JavaScript中的5种主要的数据类型（包括Number、String、Object、Array、Boolean）进行值复制
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- 面试
---

## 要求
实现一个函数clone，可以对JavaScript中的5种主要的数据类型（包括Number、String、Object、Array、Boolean）进行值复制。

## code

```javascript
/** 对象克隆 
 * 支持基本数据类型及对象 
 * 递归方法 */
function clone(obj) {
    var o;
    switch (typeof obj) {
        case "undefined":
            break;
        case "string":
            o = obj + "";
            break;
        case "number":
            o = obj - 0;
            break;
        case "boolean":
            o = obj;
            break;
        case "object": // object 分为两种情况 对象（Object）或数组（Array） 
            if (obj === null) {
                o = null;
            } else {
                if (Object.prototype.toString.call(obj).slice(8, -1) === "Array") {
                    o = [];
                    for (var i = 0; i < obj.length; i++) {　　
                        o.push(clone(obj[i]));
                    }
                } else {
                    o = {};
                    for (var k in obj) {
                        o[k] = clone(obj[k]);
                    }
                }
            }
            break;
        default:
            o = obj;
            break;
    }
    return o;
}

var m1 = clone([1, 2, 3]);
var m2 = clone({ 1: '1', 'hello': 32 });
console.log(m1); //[ 1, 2, 3 ]
console.log(m2); //{ '1': '1', hello: 32 }
```

## 解析
**Object.prototype.toString.call(obj).slice(8,-1) === "Array" 是什么意思？**
**语法：**
array.slice(start, end)

**参数：**
- **start** 可选。规定从何处开始选取。如果是负数，那么它规定从数组尾部开始算起的位置。也就是说，-1 指最后一个元素，-2 指倒数第二个元素，以此类推。
- **end**   可选。规定从何处结束选取。该参数是数组片断结束处的数组下标。如果没有指定该参数，那么切分的数组包含从 start 到数组结束的所有元素。如果这个参数是负数，那么它规定的是从数组尾部开始算起的元素。

**返回值：**
- **Array** 返回一个新的数组，包含从 start 到 end （不包括该元素）的 arrayObject 中的元素。

即，如果最后一个为数组，则会返回Array

## 参考链接
[实现一个函数clone，使JavaScript中的5种主要的数据类型（包括Number、String、Object、Array、Boolean）进行值复制](https://www.cnblogs.com/guorange/p/7214726.html)
[JavaScript slice() 方法](https://www.runoob.com/jsref/jsref-slice-array.html)