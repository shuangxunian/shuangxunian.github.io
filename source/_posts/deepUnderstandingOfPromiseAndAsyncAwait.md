---
title: 深入理解 promise、generator+co、async/await 用法
excerpt: 如题目~
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
回调函数因为涉及的内容多而杂，并且在项目中也不怎么使用，所以在这里就先不说了，
本章重点讲解一下 Promise、generator + co、async/await
因为里面内容会有点多，并且还有好多代码示例。所以需要静下心慢慢看，相信看完之后，你肯定会对这三种方法涉及的异步问题的理解更上一层楼

## Promise
Promise简单的说就是一个容器，里面保存着某个未来才会结束的时间(通常是一个异步操作)的结果。从语法上说，Promise就是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法处理。
如何理解：
- 没有异步就不需要promise
- promise本身不是异步，只是我们去编写异步代码的一种方式

### promise中所谓的 4 3 2 1
**4大术语**
一定要结合异步操作来理解
既然是异步，这个操作需要有个等待的过程，从操作开始，到获取结果，有一个过程的
- 解决（fulfill）指一个 promise 成功时进行的一系列操作，如状态的改变、回调的执行。虽然规范中用 fulfill 来表示解决，但在后世的 promise 实现多以 resolve 来指代之
- 拒绝（reject）指一个 promise 失败时进行的一系列操作
- 终值（eventual value）所谓终值，指的是 promise 被解决时传递给解决回调的值，由于 promise 有一次性的特征，因此当这个值被传递时，标志着 promise 等待态的结束，故称之终值，有时也直接简称为值（value）
- 据因（reason）也就是拒绝原因，指在 promise 被拒绝时传递给拒绝回调的值

**3种状态**
在异步操作中，当操作发出时，需要处于等待状态
当操作完成时，就有相应的结果，结果有两种：
- 成功了
- 失败了

一共是3种状态，如下：
- 等待态（Pending （也叫进行态）
- 执行态（Fulfilled）（也叫成功态）
- 拒绝态（Rejected） （也叫失败态）

![](https://api2.mubu.com/v3/document_image/65462d3e-2c55-4f26-9879-35600f4086e3-3807603.jpg)

*针对每一种状态，有一些规范：*
1. 等待态（Pending）
处于等待态时，promise 需满足以下条件：
- 可以迁移至执行态或拒绝态
2. 执行态（Fulfilled）
处于执行态时，promise 需满足以下条件：
- 不能迁移至其他任何状态
- 必须拥有一个不可变的终值
3. 拒绝态（Rejected）
处于拒绝态时，promise 需满足以下条件：
- 不能迁移至其他任何状态
- 必须拥有一个不可变的据因

**2种事件**
针对3种状态，只有如下两种转换方向：
- pending –> fulfilled
- pendeing –> rejected

在状态转换的时候，就会触发事件：
- 如果是pending –> fulfiied，就会触发onFulFilled事件
- 如果是pendeing –> rejected，就会触发onRejected事件

在调用resolve方法或者reject方法的时候，就一定会触发事件
需要注册onFulFilled事件 和  onRejected事件
针对事件的注册，Promise对象提供了then方法，如下：
promise.then(onFulFilled,onRejected)
针对 onFulFilled，会自动提供一个参数，作为终值（value）
针对 onRejected，会自动提供一个参数，作为据因（reason）

**1个对象**
promise

**注：**只有异步操作的结果，可以决定当前是哪一种状态，任何其他的操作都无法改变这个状态

### 基本使用
当我们创建 promise 时，会默认的处于 Pending 状态，并且在创建的时候，promise 中一定要有一个执行器，并且这个执行器会立即执行
```javascript
// ()=>{} 叫执行器，会立即执行
let p = new Promise(()=>{ })
// 刚创建的Promise默认处理Pending状态
console.log(p)  // Promise { <pending> }
```
在 promise 的执行器中需要传入两个参数，分别是 resolve 和 reject ,在内部调用时，就分别代表状态由 pending=>fulfilled(成功) pending=>rejected(失败)
并且一旦 promise 状态发生变化之后，之后状态就不会再变了。比如：调用 resolve 之后，状态就变为 fulfilled，之后再调用 reject,状态也不会变化
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("有钱了....")  // 现在promise就处理成功态
})
console.log(p)  // Promise { '有钱了....' }
//失败态就不演示了
```
切记状态发生变化之后，之后状态就不会再变了
```javascript
// 一个promise的状态只能从等待到成功，或从等待到失败
let p = new Promise((resolve,reject)=>{
    resolve("有钱了...")  // 成功了....
    reject("没钱了...")  // 失败了....
})
p.then(()=>{
    console.log("成功了....")
},()=>{
    console.log("失败了...")
})
//只能输出  成功了...
```

### then方法
上面代码已经看到了，在使用时可以通过promise对象的内置方法then进行调用，then有两个函数参数，分别表示promise对象中调用resolve和reject时执行的函数
```javascript
let p = new Promise((resolve,reject)=>{
    // resolve("有钱了...")  // 成功了....
    reject("没钱了...")  //失败了....
})
// 在then方法中，有两个参数
// 第一个参数表示从等待状到成功态，会调用第1个参数
// 第二个参数表示从等待状到失败态，会调用第2个参数
p.then(()=>{
    console.log("有钱了....")
},()=>{
    console.log("没钱了...")
})
//输出结果 没钱了...
```
在执行完后，成功肯定有一个成功的结果 失败肯定有一个失败的原因，那么如何得到成功的结果 ？ 如何得到失败原因呢？
```javascript
let p = new Promise((resolve,reject)=>{
    // 调用reolve时，可以把成功的结果传递下去
    // resolve("有钱了...")  // 成功了...
    // 调用reject时，可以把失败的原因传递下去
    reject("没钱了...")  // 失败了....
})
p.then((suc)=>{
    console.log(suc)
},(err)=>{
    console.log(err)
})
//输出结果  没钱了...
```
当我们在执行失败处理时，也可以用 throw ,就是抛出一个错误对象，也是失败的
如下：
```javascript
let p = new Promise((resolve,reject)=>{
    // throw 一个错误对象  也是失败的
    throw new Error("没钱了...")
})
p.then((suc)=>{
    console.log(suc)
},(err)=>{
    console.log(err)
})
```
throw的定义：throw语句用来抛出一个用户自定义的异常。当前函数的执行将被停止（throw之后的语句将不会执行），并且控制将被传递到调用堆栈中的第一个catch块。如果调用者函数中没有catch块，程序将会终止。
```javascript
//尝试一下
function getRectArea(width, height) {
  if (isNaN(width) || isNaN(height)) {
    throw "Parameter is not a number!";
  }
}
try {
  getRectArea(3, 'A');
}
catch(e) {
  console.log(e);
  // expected output: "Parameter is not a number!"
}
```
promise本身是同步的
```javascript
// promise本身是同步的
console.log("start")
let p = new Promise(()=>{
    console.log("哈哈")  // 哈哈
})
console.log("end")  
//输出顺序   start  哈哈   end
```
并且在执行器的内部也是可以写异步代码的
那么then中的方法什么时候调用呢？
只有当调用resolve，reject时才会去执行then中的方法
```javascript
let p = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        console.log("setTimeout")
        // resolve("有钱了...")
        reject("没钱了...")
    },1000)
})
p.then((suc)=>{
    console.log(suc)  // 有钱了...
},(err)=>{
    console.log(err)  // 没钱了...
})
```

### 链式调用(重点)
首先我们在目录下面建两个文件，分别是：name.txt 和 age.txt
- name.txt文件里写了一个 age.txt
- age.txt文件里写了一个 666

下面就以读取文件为例，来演示链式调用(需要了解一点node基础)，当你读取文件的时候，如果你用的是 vscode 编辑器，里面会有一个小bug，用相对路径可能会出错，所以最好使用绝对路径
读取文件：
```javascript
let fs = require("fs")
let path = require("path")
let filename = path.join(__dirname,"name.txt")
fs.readFile(filename,"utf8",(err,data)=>{
    if(err){
        console.log(err)
    }
    fs.readFile(path.join(__dirname,data),"utf8",(err,data)=>{
        if(err){
            console.log(err)
        }
        console.log(data)
    })
})
//输出结果 666
```
如果用这种方法，就会出现 回调地狱 ，很难受，所以一般不用
在读取文件时，我们可以专门 封装一个函数 ，功能就是读取一个文件的内容
```javascript
let fs = require("fs")
// 封装一个函数,函数的功能是读取一个文件的内容
// rest参数(下去自己了解一下，就是可以获取到传过来的所有内容)  
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}
//读文件
readFile("./name.txt","utf8").then(data=>{
    console.log(data)   
},err=>{
    console.log(err)  
})
//输出结果  age.txt
```
如果文件不存在，会走第二个函数
```javascript
let fs = require("fs")
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}
// 如果name1不存在，走then的第2个函数
readFile("./name1.txt","utf8").then(data=>{
    console.log(data)  
},err=>{
    console.log(err)   
})
//报错  no such file or directory
```
那么如果我们想要读取age.txt里面的内容呢？
我们可以这么写：
```javascript
let fs = require("fs")
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}
 
readFile("./name.txt","utf8").then(data=>{
    // console.log(data)  // age.txt
    readFile(data,"utf8").then(data=>{
        console.log(data)  // 666
    },err=>{
        console.log(err)  
    })
},err=>{
    console.log(err)  
})
//输出结果  666
```
这样写就可以获取到age.txt文件里面的内容，但是呢，这样写又回到了 回调地狱 ，不是说这种方法不行，而是不够优雅

**使用链式调用**
在promise中可以链式调用 就是  .then  之后，还可以  .then  ,你可以无数次的 .then
.then 之后又返回了一个新的 promise，就是.then的函数参数中会默认返回promise对象，所以当你碰到.then连续调用的时候，你就可以把前面的所有代码当成一个promise
```javascript
let fs = require("fs")
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}

readFile("./name.txt","utf8").then(data=>{
    // console.log(data)  // age.txt
    return false;
},err=>{
    console.log(err)  
}).then(data=>{  // 这里面的data是上一个then中的第一个函数的返回值，这个.then前面的一坨代码就可以当成一个promise
    console.log(data)  // false
},err=>{

})
```
如果没有这个文件，则返回错误信息
```javascript
let fs = require("fs")
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}
readFile("./name1.txt","utf8").then(data=>{
    return data;
},err=>{
    return err;
    // console.log(err)  
}).then(data=>{  
    console.log(data)
},err=>{
    console.log(err)
})
//输出结果   no such file or directory
```
但是如果我们返回一个promise呢？
那么这个promise会执行，并且会采用他的状态
```javascript
let fs = require("fs")
function readFile(...args){
    return new Promise((resolve,reject)=>{
        fs.readFile(...args,function(err,data){
            if(err) reject(err)
            resolve(data)
        })
    });
}

readFile("./name.txt","utf8").then(data=>{
    return data;
},err=>{
    return err;
    // console.log(err)  
}).then(data=>{  
    // console.log(data)
    return new Promise((resolve,reject)=>{	//返回一个promise
        reject("不OK")	//下面的.then采用这个状态(失败态)
    })
},err=>{}).then(data=>{
    console.log(data)
},err=>{
    console.log(err)  // 不OK
})
//输出结果  不OK
```
所以如果返回的是一个promise，那么这个promise会执行，并且会采用它的状态

**小总结：**
如果在上一个then的第一个函数中，返回一个普通值，那么无论你是在第1个函数中返回，还是在第2个函数中返回，都会作为下一个then的成功的结果，如果不返回，undefined就作为下一个then的成功的结果
如果返回的是一个promise，会作为下一个then的promise对象，data err去promise对象中去取，也就是说，前一个then的返回值，会作为后一个then的参数
再给两个小例子，自己看一下：
```javascript
let p = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        resolve("hello")
    },1000)
})
p.then(data=>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve("world")
        },1000)
    })
}).then(data=>{
    console.log(data)  
},err=>{

})
//输出结果 world

```

```javascript
let p = new Promise((resolve,reject)=>{
    resolve("hello")
})
let p1 = p.then(data=>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            reject("不OK")
        },1000)
    })
})
p1.then(data=>{
    console.log(data)  
},err=>{
    console.log(err) 
})
//输出结果   不OK

```

#### 一个坑
接下来说一个小问题，链式调用中的循环引用
有的人不喜欢把前面的一大堆代码后面加.then，所以就用了下面的一种写法(有可能会出现 循环引用)：
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("hello")
})

let p1 = p.then(data=>{	
    return p1	//循环引用 报错 p1在等p1的状态改变，也就是我在等我吃饭，显然是不行的
})

p1.then(data=>{
    console.log(data)
},err=>{
    console.log("-----",err)	//可执行，然后报错
})

```
如果我们把状态确定住，那就可以了
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("hello")
})
let p1 = p.then(data=>{
    // return 123  相当于把等待态改变成成功态
    return 123
})
p1.then(data=>{
    console.log(data)  // 123
},err=>{
    console.log("-----",err)
})
//输出 123 

```
当然改变成失败态也可以
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("hello")
})
let p1 = p.then(data=>{
    //  return new Error("不OK") 把等待态变成失败态
    return new Error("不OK")
})
p1.then(data=>{
    console.log(data) 
},err=>{
    console.log(err)  // Error: 不OK
})

```

### 递归解析
看一个问题
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("hello")
})
let p1 = p.then(data=>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve(new Promise((resolve,reject)=>{
                setTimeout(()=>{
                    resolve("666")
                },1000)
            }))
        },1000)
    })
})
// data是promise 还是666
p1.then(data=>{
    console.log(data)  
},err=>{
    console.log(err) 
})

```
按理说，data打印出来的是一个promise，就是上面resolve里面的一堆代码
然而并不是，promise会 进行递归解析.到最后上面代码会打印出来 666
如果在resolve或reject中又是一个promise，那么就会递归解析(无论有多少个promise)
```javascript
let p = new Promise((resolve, reject) => {
    resolve("hello")
})
let p1 = p.then(data => {
    return new Promise((resolve, reject) => {
        resolve(new Promise((resolve, reject) => {
            resolve(new Promise((resolve, reject) => {
                resolve(new Promise((resolve, reject) => {
                    resolve(new Promise((resolve, reject) => {
                        resolve(new Promise((resolve, reject) => {
                            resolve("666")
                        }))
                    }))
                }))
            }))
        }))
    })
})
p1.then(data => {
    console.log(data)
}, err => {
    console.log(err)
})
//打印结果  666

```

### catch方法
- catch方法，用于注册 onRejected 回调
- catch其实是then的简写，then(null，callback)

catch就是.then的语法糖
如果.then中有第2个函数，在这个.then后面又有catch，如果到失败态，那么会走.then的第2个函数
```javascript
let p = new Promise((resolve,reject)=>{
    reject("不OK")
})

p.then(data=>{

},err=>{
    console.log("1",err)	
}).catch(err=>{
    console.log("2",err)
})
//输出结果 1 不OK

```
如果.then中没有第2个函数，在这个.then后面又有catch，如果到失败态，那么会走catch
```javascript
let p = new Promise((resolve,reject)=>{
    reject("不OK")
})
p.then(data=>{

}).catch(err=>{
    console.log("2",err)
})
//输出结果  2 不OK

```

#### 一个坑
在.then第二个函数中，return err 这个 err 它是 return 到了下一个.then的第一个函数中
```javascript
let p = new Promise((resolve,reject)=>{
    reject("不OK")
})
p.then(data=>{

},err=>{
    // 在这里它并没有reutrn到err中，它reutrn到第一个参数中
    return err
}).then(data=>{
    console.log("data----",data)
},err=>{
    console.log("err----",err)
})
//输出结果 data---- 不OK

```
所以最终：
一个promise中，一般在then中只有一个函数，在then后面有一个catch，一般使用then来获取data，在catch中获取err
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("OK")
})

p.then(data=>{
    console.log(data)
}).catch(err=>{
    console.log(err)
})

```

### 静态方法
在Pomise类上面，提供了几个静态方法:
- resolve
- reject
- finally
- all
- race

#### resolve
```javascript
Promise.resolve("有钱了...").then(data=>{
     console.log(data)  // 有钱了...
})

```
等价于下面这种写法：
```javascript
let p = new Promise((resolve,reject)=>{
    resolve("有钱了...")
})
p.then(data=>{
    console.log(data)
})

```

#### reject 
```javascript
Promise.reject("没钱了...").catch(data=>{
    console.log(data)  // 没钱了...
})
```

#### finally
不管转成成功态还是失败态，都会调用finally这个方法
```javascript
Promise.resolve("有钱").then(data=>{
    console.log(data)
}).finally(()=>{
    console.log("开心...")
})
//打印结果  有钱   开心...
```
```javascript
Promise.reject("没钱").catch(data=>{
    console.log(data)
}).finally(()=>{
    console.log("不开心...")
})
//打印结果  没钱   不开心...
```

#### all
all表示[ ]中的promise都成功了，才能得到最终的值
注意里面是一个数组 读取name.txt和age.txt中的内容
```javascript
let fs = require("fs").promises;
// all表示[]中的promise都成功了，才能得到最终的值
Promise.all([fs.readFile("./name.txt","utf-8"),fs.readFile("./age.txt","utf-8")]).then(data=>{
    console.log(data) // [ 'age.txt', '666' ]
})
//打印结果  [ 'age.txt', '666' ]

```
如果有一个不成功，那么就不行
```javascript
let fs = require("fs").promises;
Promise.all([fs.readFile("./name1.txt","utf-8"),fs.readFile("./age.txt","utf-8")]).then(data=>{
    console.log(data)
})
//这个是不行的

```

#### race
顾名思义,race就是赛跑的意思，意思就是说，Promise.race([p1, p2, p3])里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。
```javascript
let fs = require("fs").promises;
Promise.race([fs.readFile("./name.txt","utf-8"),fs.readFile("./age.txt","utf-8")]).then(data=>{
    console.log(data) // age.txt
})

```
改变文件里面的内容，多尝试几次

## generator + co
先说 生成器 和 迭代器
生成器可以生成迭代器，可以让程序中断，不会把 { } 中的代码全部执行
用法：在function和自己声名的名称之间加一个 * 号，里面用yield
产出数据，然后调用生成器生成迭代器
```javascript
function * read(){
    yield 1;  // 只有产出，并不执行
}
//调用生成器 生成 迭代器   it就是迭代器  
let it = read()

```
生成器可以产出很多值，迭代器只能next一下，拿一个值，next一下，拿一个值
```javascript
function * read(){
    yield 1;  
}
let it = read()
console.log(it.next())  // { value: 1, done: false }
console.log(it.next())  // { value: undefined, done: true }

```
```javascript
function * read(){
    yield 1;  
    yield 2;
    yield 3;
}
// 调用read()  返回值是迭代器
let it = read()
console.log(it.next())  // { value: 1, done: false }
console.log(it.next())  // { value: 2, done: false }
console.log(it.next())  // { value: 3, done: false }
console.log(it.next())  // { value: undefined, done: true }

```
如果 next 中有参数的话，那么他会把这个参数传给上一个生成器声明的变量里
所以第一个 next 中的参数没有任何意义，我们一般不写
```javascript
function * read(){
    let a = yield 1; 
    console.log(a)    // 9
    let b = yield 2;
    console.log(b)    // 10
    let c = yield 3;
    console.log(c)   // 11
}
let it = read()
console.log(it.next())   // { value: 1, done: false }
console.log(it.next(9))    // { value: 2, done: false }
console.log(it.next(10))   // { value: 3, done: false }
console.log(it.next(11))   // { value: undefined, done: true }

```
接下来用这个实现我们的读文件操作
读取name.txt文件
```javascript
const fs = require("fs").promises;
// 生成器
function * read(){
    yield fs.readFile("./name.txt","utf-8")
}
// 迭代器
let it = read()
// console.log(it.next())  // { value: Promise { <pending> }, done: false }
it.next().value.then(data=>{	//因为是一个对象，所以直接.value
    console.log(data)  
})
//输出结果 age.txt

```
然后读取age.txt文件
```javascript
const fs = require("fs").promises;
function * read(){
    let concent = yield fs.readFile("./name.txt","utf-8")
    yield fs.readFile(concent,"utf-8")

}
let it = read()
it.next().value.then(data=>{
    // console.log(data)  
    // console.log(it.next(data)) // { value: Promise { <pending> }, done: false }
    it.next(data).value.then(data=>{
        console.log(data)  
    })
})
//输出结果 666

```
也可以这样
```javascript
const fs = require("fs").promises;
function * read(){
    let concent = yield fs.readFile("./name.txt","utf-8")
    let age = yield fs.readFile(concent,"utf-8")
    return age
}
let it = read()
it.next().value.then(data=>{
    it.next(data).value.then(data=>{
        let r = it.next(data)
        console.log(r)  // { value: '666', done: true }
    })
})

```
是不是感觉又陷入了 回调地狱

**那么就用 co 吧**
安装co npm i co ，用上来：
```javascript
const fs = require("fs").promises;
function * read(){
    let concent = yield fs.readFile("./name.txt","utf-8")
    let age = yield fs.readFile(concent,"utf-8")
    return age
}
let co = require("co")
co(read()).then(data=>{
    console.log(data)  // 
})
//输出结果  666

```
是不是简单多了，爽不爽
co库可以实现自动迭代
既然是自动执行，那么promise的executor中先执行一次it.next()方法,返回value和done；value是一个pending的Promise;如果done=false，说明还没有走完，继续在value.then的成功回调中执行下一次next，即调用read方法；直到done为true，走完所有代码，调用resolve;中间有任何一次next异常，直接调用reject，停止迭代

**总结：**
- function关键字与函数名之间有一个星号
- 函数体内部使用yield语句，定义不同的内部状态
- yield会将函数分割成好多个部分，每产出一次，就暂停一次
- Genenrator是一个生成器，调用Genenrator函数，不会立即执行函数体，只是创建了一个迭代器对象,如上例中的it就是调用read这个Generator函数得到的一个迭代器
- 迭代器有一个next方法，调用一次就会继续向下执行，直到遇到下一个yield或return
- next()方法可以带一个参数，该参数会被当做上一条yield语句的返回值,并赋值给yield前面等号前的变量
- 每遇到一个yield,就会返回一个{value:xxx,done:bool}的对象，然后暂停，返回的value就是跟在yield后面的返回值，done表示这个generator是否已经执行结束了
- 当遇到return时，return后的值就是value值，done此时就是true
- 函数末尾如果没有return，就是隐含的return undefined
- 使用co库，可以自动的将generator迭代
- co执行会返回一个promise，用then注册成功/失败回调
- co将迭代器it作为参数，这里每调用一次read，就执行一次next

## async/await
被称为 异步解决 的终极方案
async、await是什么？
async顾名思义是“异步”的意思，async用于声明一个函数是异步的。
而await从字面意思上是“等待”的意思，就是用于等待异步完成。通俗来说，就是await在这里等待promise返回结果了，再继续执行。并且await只能在async函数中使用。
通常async、await都是跟随Promise一起使用的。
为什么这么说呢？因为 async返回的都是一个Promise对象,同时async适用于任何类型的函数上。这样await得到的就是一个Promise对象(如果不是Promise对象的话那async返回的是什么 就是什么);
**注: **await 不仅仅用于等 Promise 对象，它可以等任意表达式的结果，所以，await 后面实际是可以接普通函数调用或者直接量的(不演示了)
紧跟着上面的代码，再写一段
```javascript
const fs = require("fs").promises;
async function read(){
    let concent = await fs.readFile("./name.txt","utf-8")
    let age = await fs.readFile(concent,"utf-8")
    return age
}
read().then(data=>{
    console.log(data)   // 666
})

```
是不是比上面的写法还爽呢？

await命令后面的 Promise 对象，运行结果可能是 rejected，所以最好把 await 命令放在 try...catch 代码块中
如下：
```javascript
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise().catch(function (err){
    console.log(err);
  });
}

```

**总结：**
async：
- async函数会返回一个Promise对象
- 如果async函数中是return一个值，这个值就是Promise对象中resolve的值
- 如果async函数中是throw一个值，这个值就是Promise对象中reject的值

await：
- await只能在async函数中使用
- await后面要跟一个promise对象
- await等promise返回结果后，在继续执行

## 参考链接
[深入理解 promise、generator+co、async/await 用法](https://juejin.im/post/6844903919554920461)