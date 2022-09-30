---
title: 从.map(parseInt)讲起
excerpt: 简单讲一下map和parselnt
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- 面试
---

## 前言
看到一道笔试题:
```javascript
['1', '2', '3'].map(parseInt)
```
这道题目中涉及到 map 和 parseInt 函数的运用，如果对这两个函数的理解不充分的话，是很难思考出正确的结果的。

## 函数
```javascript
map((item, index, thisArr) => ( newArr ))
```
**参数解析**
- item: callback 的第一个参数，数组中正在处理的当前元素。
- index: callback 的第二个参数，数组中正在处理的当前元素的索引。
- thisArr: callback 的第三个参数，map 方法被调用的数组。

**返回**
- 一个新数组，每个元素都是执行回调函数的结果。

```javascript
parseInt(string, radix)
```
**参数解析**
- string: 必需。要被解析的字符串。
- radix: 可选。表示要解析的字符串的基数。该值介于 2 ~ 36 之间。如果省略该参数或其值为 0，则数字将以 10 为基础来解析。如果它以 “0x” 或 “0X” 开头，将以 16 为基数。如果该参数小于 2 或者大于 36，则 parseInt() 将返回 NaN。

**返回**
- 解析后的数字

**注意**
```javascript
// 1. 只有字符串中的第一个数字会被返回。 
parseInt(' 12abc!6')    //12
// 2. 开头和结尾的空格是允许的。 
parseInt(' 12x')        //12
// 3. 如果字符串的第一个字符不能被转换为数字，那么 parseInt() 会返回 NaN。     
parseInt('s90')         //NaN
// 4. radix 表示的是当前要解析的字符串的表示进制数，而不是解析的进制数。
parseInt('3', 2);        //表示当前的字符串'3' 是以二进制表示的（当然这是不合规则的，仅为说明问题），而不是将 3 用二进制作转换
```

**示例**
```javascript
parseInt("10"); // 10
parseInt("19",10); // 19 (10+9)
parseInt("11",2); // 3 (2+1)
parseInt("17",8); // 15 (8+7)
parseInt("1f",16); // 31 (16+15)
parseInt("010"); // 10 或 8
```

## ['1', '2', '3'].map(parseInt) 解析
通过上述对 map 和 parseInt 函数的分析可以知道，执行方法时，map给parseInt传递了三个参数，其中第三个参数会被 parseInt 忽略，因此会依次执行：
```javascript
parseInt('1', 0)
// radix 为 0，默认以十进制解析字符串，返回 1
parseInt('2', 1)
// radix 为 1，不在 2 ~ 36 之间，返回 NaN
parseInt('3', 2)
// radix 为 2， 字符串却为 3，超出二进制的表示范围，因此要解析的字符串和基数矛盾，返回 NaN

//return [1, NaN, NaN]
```

## 补充
一些看起来奇怪但实际上解释得通的例子：
```javascript
parseInt(false, 16)     // 250
parseInt(parseInt, 16)  // 15
parseInt('0x10', 16)    // 16
parseInt('103', 2)      // 2
parseInt(1/0, 19)       // 18
```

## 参考链接
[通过 ['1', '2', '3'].map(parseInt) 学习 map 和 parseInt 函数](https://www.cnblogs.com/wx1993/p/8417817.html)