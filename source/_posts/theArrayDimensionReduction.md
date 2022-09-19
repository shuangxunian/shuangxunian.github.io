---
title: js之数组降维
excerpt: 一道办公面试常常会遇到的问题
categories:
- 技术文章
tags:
- js
- 面试
---

## 前言
将多维数组（尤其是二维数组）转化为一维数组是业务开发中的常用逻辑，除了使用朴素的循环转换以外，我们还可以利用Javascript的语言特性和数据结构的思想实现更为简洁优雅的转换。

## 二维数组降维
二维数组只有两个维度，降维比较简单，也不用考虑太复杂的算法逻辑，我们看一下二维数组降维的几种方法；

### 遍历降维
此方法思路简单，利用双重循环遍历二维数组中的每个元素并放到新数组中。

```javascript
var arr = [
    ['h', 'e', 'l', 'l', 'o'],
    ['m', 'y'],
    ['w', 'o', 'r', 'l', 'd'],
    ['!']
];
var result = [];
for (var r = 0; r < arr.length; r++) {
    for (var c = 0; c < arr[r].length; c++) {
        result.push(arr[r][c]);
    }
}
console.log(result); //=>[ 'h', 'e', 'l', 'l', 'o', 'm', 'y', 'w', 'o', 'r', 'l', 'd', '!' ]
```

### 使用concat
利用concat方法，可以将双重循环简化为单重循环：

```javascript
var arr = [
    ['h', 'e', 'l', 'l', 'o'],
    ['m', 'y'],
    ['w', 'o', 'r', 'l', 'd'],
    ['!']
];
var result = [];
for (var r = 0, result = []; r < arr.length; r++) {
    result = result.concat(arr[r]);
}
console.log(result); //=>[ 'h', 'e', 'l', 'l', 'o', 'm', 'y', 'w', 'o', 'r', 'l', 'd', '!' ]
```

arr的每一个元素都是一个数组或参数，作为concat方法的参数，数组中的参数或每一个子元素又都会被独立插入进新数组。

**concat：**concat方法用于多个数组的合并。它将新数组的成员，添加到原数组成员的后部，然后返回一个新数组，原数组不变。

```javascript
['hello'].concat(['world'])
// ["hello", "world"]

['hello'].concat(['world'], ['!'])
// ["hello", "world", "!"]

[].concat({a: 1}, {b: 2})
// [{ a: 1 }, { b: 2 }]

[2].concat({a: 1})
// [2, {a: 1}]
```

除了数组作为参数，concat也接受其他类型的值作为参数，添加到目标数组尾部。

```javascript
[1, 2, 3].concat(4, 5, 6)
// [1, 2, 3, 4, 5, 6]
```

如果数组成员包括对象，concat方法返回当前数组的一个浅拷贝。所谓“浅拷贝”，指的是新数组拷贝的是对象的引用。

```javascript
var obj = { a: 1 };
var oldArray = [obj];

var newArray = oldArray.concat();

obj.a = 2;
newArray[0].a // 2
```

上面代码中，原数组包含一个对象，concat方法生成的新数组包含这个对象的引用。所以，改变原对象以后，新数组跟着改变。

### 使用apply+concat
利用apply方法，只需要一行代码就可以完成二维数组降维了。

```javascript
var arr = [
    ['h', 'e', 'l', 'l', 'o'],
    ['m', 'y'],
    ['w', 'o', 'r', 'l', 'd'],
    ['!']
];
var result = Array.prototype.concat.apply([], arr);
console.log(result); //=>[ 'h', 'e', 'l', 'l', 'o', 'm', 'y', 'w', 'o', 'r', 'l', 'd', '!' ]
```

**apply:**apply方法会调用一个函数，apply方法的第一个参数会作为被调用函数的this值，apply方法的第二个参数（一个数组，或类数组的对象）会作为被调用对象的arguments值，也就是说该数组的各个元素将会依次成为被调用函数的各个参数。

apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数，使用格式如下。

```javascript
func.apply(thisValue, [arg1, arg2, ...])
```

apply方法的第一个参数也是this所要指向的那个对象，如果设为null或undefined，则等同于指定全局对象。第二个参数则是一个数组，该数组的所有成员依次作为参数，传入原函数。原函数的参数，在call方法中必须一个个添加，但是在apply方法中，必须以数组形式添加。

```javascript
function f(x, y){
  console.log(x + y);
}

f.call(null, 1, 1) // 2
f.apply(null, [1, 1]) // 2
```

上面代码中，f函数本来接受两个参数，使用apply方法以后，就变成可以接受一个数组作为参数。

利用这一点，可以做一些有趣的应用。

**找出数组最大元素**
JavaScript 不提供找出数组最大元素的函数。结合使用apply方法和Math.max方法，就可以返回数组的最大元素。

```javascript
var a = [10, 2, 4, 15, 9];
Math.max.apply(null, a) // 15
```

**将数组的空元素变为undefined**
通过apply方法，利用Array构造函数将数组的空元素变成undefined。

```javascript
Array.apply(null, ['a', ,'b'])
// [ 'a', undefined, 'b' ]
```

空元素与undefined的差别在于，数组的forEach方法会跳过空元素，但是不会跳过undefined。因此，遍历内部元素的时候，会得到不同的结果。

```javascript
var a = ['a', , 'b'];

function print(i) {
  console.log(i);
}

a.forEach(print)
// a
// b

Array.apply(null, a).forEach(print)
// a
// undefined
// b
```

**转换类似数组的对象**
另外，利用数组对象的slice方法，可以将一个类似数组的对象（比如arguments对象）转为真正的数组。

```javascript
Array.prototype.slice.apply({0: 1, length: 1}) // [1]
Array.prototype.slice.apply({0: 1}) // []
Array.prototype.slice.apply({0: 1, length: 2}) // [1, undefined]
Array.prototype.slice.apply({length: 1}) // [undefined]
```

上面代码的apply方法的参数都是对象，但是返回结果都是数组，这就起到了将对象转成数组的目的。从上面代码可以看到，这个方法起作用的前提是，被处理的对象必须有length属性，以及相对应的数字键。

**绑定回调函数的对象**
举个按钮点击事件的例子，如下。

```javascript
var o = new Object();

o.f = function () {
  console.log(this === o);
}

var f = function (){
  o.f.apply(o);
  // 或者 o.f.call(o);
};

// jQuery 的写法
$('#button').on('click', f);
```

上面代码中，点击按钮以后，控制台将会显示true。由于apply()方法（或者call()方法）不仅绑定函数执行时所在的对象，还会立即执行函数，因此不得不把绑定语句写在一个函数体内。

## 多维数组降维
多维数组就如二维数组那么简单了，因为不确定数组的深度，所以也不能进行遍历来降维，只能通过递归或者栈方法来实现。

### 递归
通过递归的方法实现了多维数组的降维，在这里使用了原型链将方法封装进了Array原型中，可以直接在数组方法中调用。

```javascript
Array.prototype.deepFlatten = function() {
    var result = []; //定义保存结果的数组
    this.forEach(function(val, idx) { //遍历数组
        if (Array.isArray(val)) { //判断是否为子数组
            val.forEach(arguments.callee); //为子数组则递归执行
        } else {
            result.push(val); //不为子数组则将值存入结果数组中
        }
    });
    return result; //返回result数组
}
var arr = [2, 3, [2, 2],
    [3, 'f', ['w', 3]], { "name": 'Tom' }
];
console.log(arr.deepFlatten()); //=>[ 2, 3, 2, 2, 3, 'f', 'w', 3, { name: 'Tom' } ]
```

### 栈方法
通过栈方法，建立了一个栈，将数组的内容存进去，然后逐个取出来，如果取出来的是个数组，就将这个数组打散拼接进栈中，在出栈一个，这样循环。

```javascript
Array.prototype.deepFlatten = function() {
    var result = []; //定义保存结果的数组
    var stack = this; //将数组放入栈中
    while (stack.length !== 0) { //如果栈不为空，则循环遍历
        var val = stack.pop(); //取出最后一个值
        if (Array.isArray(val)) { //判断是不是数组
            stack = stack.concat(val); //如果是数组就将拼接入栈中
        } else {
            result.unshift(val); //如果不是数组就将其取出来放入结果数组中
        }
    }
    return result;
}
var arr = [2, 3, [2, 2],
    [3, 'f', ['w', 3]], { "name": 'Tom' }
];
console.log(arr.deepFlatten()); //=>[ 2, 3, 2, 2, 3, 'f', 'w', 3, { name: 'Tom' } ]
```

多维数组降维的方法也可以降维二维数组，但是有点大材小用，还是用对的方法做对的事才是最好的！

## 参考链接
[JavaScript 之数组降维](https://juejin.im/entry/6844903456277266445)
[JavaScript 教程](https://wangdoc.com/javascript/index.html)