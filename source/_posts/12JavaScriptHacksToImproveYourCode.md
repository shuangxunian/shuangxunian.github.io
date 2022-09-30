---
title: 12个 Javascript 小技巧帮你提升代码质量
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 提炼函数
好处：
- 避免出现超大函数。
- 独立出来的函数有助于代码复用。
- 独立出来的函数更容易被覆写。
- 独立出来的函数如果拥有一个良好的命名，它本身就起到了注释的作用。
- 语义化将多段分离的逻辑放在不同的函数中实现，可以使代码逻辑清晰，清楚的看到每一步在做什么。

代码举例：
- 要求：实现获取数据，然后操作dom显示数据，最后添加事件
- 函数提炼前
    ```javascript
    // 逻辑都写在一起，需要将所有逻辑看完才知道这段代码是干嘛的，局部逻辑无法复用
    function main() {
        $.ajax.get('/getData').then((res) => {
            const ul = document.getElementById('ul');
            ul.innerHTML = res.list.map(text => `<li class="li">${text}</li>`).join('\n');
            const list = document.getElementsByClassName('li');
            for (let i = 0; i < list.length; i ++) {
                list[i].addEventListener('focus', () => {
                    // do something
                });
            }
        });
    }
    ```
- 函数提炼后
    ```javascript
    function getData() {
        return $.ajax.get('/getData').then((res) => res.data.list);
    }
    function showList(list) {
        const ul = document.getElementById('ul');
        ul.innerHTML = list.map(text => `<li class="li">${text}</li>`).join('\n');
    }
    function addEvent() {
        const list = document.getElementsByClassName('li');
        for (let i = 0; i < list.length; i ++) {
            list[i].addEventListener('focus', () => {
                // do something
            });
        }
    }
    // 逻辑清晰，一眼读懂每一步在做什么，某些提炼出来的函数还可以被复用
    async function main() {
        const list = await getData(); // 获取数据
        showList(list); // 显示页面
        addEvent(); // 添加事件
    }
    ```

## 合并重复的条件片段
如果一个函数体内有一些条件分支语句，而这些条件分支语句内部散布了一些重复的代码，那么就有必要进行合并去重工作。
```javascript
// 合并前
function main( currPage ){
    if ( currPage <= 0 ){
        currPage = 0;
        jump( currPage ); // 跳转
    }else if ( currPage >= totalPage ){
        currPage = totalPage;
        jump( currPage ); // 跳转
    }else{
        jump( currPage ); // 跳转
    }
};

// 合并后
function main( currPage ){
    if ( currPage <= 0 ){
        currPage = 0;
    }else if ( currPage >= totalPage ){
        currPage = totalPage;
    }
    jump( currPage ); // 把jump 函数独立出来
};

```

## 把条件分支语句提炼成函数
复杂的条件分支语句是导致程序难以阅读和理解的重要原因，而且容易导致一个庞大的函数。有时可以将条件分支语句提炼成语义化的函数，使代码更加直观，逻辑清晰。
```javascript
// 根据不同季节决定打折力度
function getPrice( price ){
    var date = new Date();
    if ( date.getMonth() >= 6 && date.getMonth() <= 9 ){ // 夏天
        return price * 0.8;
    }
    return price;
};


// 是否是夏天
function isSummer(){
    var date = new Date();
    return date.getMonth() >= 6 && date.getMonth() <= 9;
};
// 提炼条件后
function getPrice( price ){
    if ( isSummer() ){
        return price * 0.8;
    }
    return price;
};
```

## 合理使用循环
如果多段代码实际上负责的是一些重复性的工作，那么可以用循环代替，使代码量更少。
```javascript
// 判断是什么浏览器
function getBrowser(){
    const str = navigator.userAgent;
    if (str.includes('QQBrowser')) {
	return 'qq';
    } else if (str.includes('Chrome')) {
	return 'chrome';
    } else if (str.includes('Safari')) {
        return 'safri';
    } else if (str.includes('Firefox')) {
        return 'firefox';
    } else if(explorer.indexOf('Opera') >= 0){
        return 'opera';
    } else if (str.includes('msie')) {
        return 'ie';
    } else {
        return 'other';
    }
};
// 循环判断，将对应关系抽象为配置，更加清晰明确
function getBrowser(){
    const str = navigator.userAgent;
    const list = [
        {key: 'QQBrowser', browser: 'qq'},
        {key: 'Chrome', browser: 'chrome'},
        {key: 'Safari', browser: 'safari'},
        {key: 'Firefox', browser: 'firefox'},
        {key: 'Opera', browser: 'opera'},
        {key: 'msie', browser: 'ie'},
    ];
    for (let i = 0; i < list.length; i++) {
        const item = list[i];
        if (str.includes(item.key)) {return item.browser};
    }
    return 'other';
}

```

## 提前让函数退出代替嵌套条件分支
让函数变成多出口提前返回，替换嵌套条件分支。
```javascript
function del( obj ){
    var ret;
    if ( !obj.isReadOnly ){ // 不为只读的才能被删除
        if ( obj.isFolder ){ // 如果是文件夹
            ret = deleteFolder( obj );
        }else if ( obj.isFile ){ // 如果是文件
            ret = deleteFile( obj );
        }
    }
    return ret;
};

function del( obj ){
    if ( obj.isReadOnly ){ // 反转if 表达式
        return;
    }
    if ( obj.isFolder ){
        return deleteFolder( obj );
    }
    if ( obj.isFile ){
        return deleteFile( obj );
    }
};
```

## 传递对象参数代替过长的参数列表
函数参数过长那么就增加出错的风险，想保证传递的顺序正确就是一件麻烦的事，代码可读性也会变差，尽量保证函数的参数不会太长。如果必须传递多个参数的话，建议使用对象代替。

一般来说，函数参数最好不要超过3个
```javascript
function setUserInfo( id, name, address, sex, mobile, qq ){
    console.log( 'id= ' + id );
    console.log( 'name= ' +name );
    console.log( 'address= ' + address );
    console.log( 'sex= ' + sex );
    console.log( 'mobile= ' + mobile );
    console.log( 'qq= ' + qq );
};
setUserInfo( 1314, 'sven', 'shenzhen', 'male', '137********', 123456 );

function setUserInfo( obj ){
    console.log( 'id= ' + obj.id );
    console.log( 'name= ' + obj.name );
    console.log( 'address= ' + obj.address );
    console.log( 'sex= ' + obj.sex );
    console.log( 'mobile= ' + obj.mobile );
    console.log( 'qq= ' + obj.qq );
};
setUserInfo({
    id: 1314,
    name: 'sven',
    address: 'shenzhen',
    sex: 'male',
    mobile: '137********',
    qq: 123456
});
```

## 少用三目运算符
三目运算符性能高，代码量少。
但不应该滥用三目运算符，我们应该在简单逻辑分支使用，在复杂逻辑分支避免使用。
```javascript
// 简单逻辑可以使用三目运算符
var global = typeof window !== "undefined" ? window : this;

// 复杂逻辑不适合使用
var ok = isString ? (isTooLang ? 2 : (isTooShort ? 1 : 0)) : -1;
```

## 合理使用链式调用
优点：
- 链式调用使用简单，代码量少。

缺点：
- 链式调用带来的坏处就是在调试不方便，如果我们知道一条链中有错误出现，必须得先把这条链拆开才能加上一些调试 log 或者增加断点，这样才能定位错误出现的地方。

如果该链条的结构相对稳定，后期不易发生修改，可以使用链式。
```javascript
var User = {
    id: null,
    name: null,
    setId: function( id ){
        this.id = id;
        return this;
    },
    setName: function( name ){
        this.name = name;
        return this;
    }
};
User
  .setId( 1314 )
  .setName( 'sven' );

var user = new User();
user.setId( 1314 );
user.setName( 'sven' );
```

## 分解大型类
大型类的分解和函数的提炼很像，类太大会出现逻辑不清晰，难以理解和维护的问题。

合理的大类分解可以使类的逻辑清晰，且子模块可以方便复用。

## 活用位操作符
编程语言计算乘除的性能都不高，但是某些情况使用位操作符可以提升乘除等运算的性能。

## 纯函数
纯函数是指不依赖于且不改变它作用域之外的变量状态的函数。

纯函数的返回值只由它调用时的参数决定，它的执行不依赖于系统的状态（执行上下文）。

相同的输入参数，一定会得到相同的输出，也就是内部不含有会影响输出的随机变量。

不属于纯函数的特点：
- 更改文件系统
- 往数据库插入记录
- 发送一个 http 请求
- 可变数据
- 打印/log
- 获取用户输入
- DOM 查询
- 访问系统状态

纯函数的作用：
- 可靠性：函数返回永远和预期一致
- 可缓存性：因为只要输入一样输出一定一样，因此可将输入作为key，输出作为值，使用对象缓存已经计算的结果
- 可移植性：因为没有外部依赖，所以移植到任何环境都可正确运行
- 可测试性：方便针对函数做单元测试
- 可并行性：对一些复杂计算，可以并行计算（例如使用nodejs多个子进程同时并行计算多个任务，提高计算速度）

应用场景：
- 工具函数最好使用纯函数
- 多平台使用的代码（nodejs、浏览器、微信小程序、native客户端等）
- 相对独立的功能

```javascript
var a = 1;
// 非纯函数
function sum(b) {
    return a + b;
}
// 非纯函数
function sum(b) {
    a = 2;
    return b;
}
// 非纯函数
function sum(b) {
    return b + Math.random();
}


// 纯函数
function sum (b, c) {
    return b + c;
}
```

## 用映射代替重复逻辑分支
```javascript
// 重复的逻辑分支
function example(type) {
    if (type === 'a') {
        return '类型1';
    } else if (type === 'b')  {
        return 'type 2';
    } else if (type === 'c')  {
        return '3';
    } else if (type === 'd')  {
        return '4';
    }
}

// 使用映射代替重复的逻辑分支
function example(type) {
    const map = {
        a: '类型1',
        b: 'type 2',
        c: '3',
        d: '4'
    };
    return map[type];
}
```

## 参考链接
[12个 Javascript 小技巧帮你提升代码质量](https://juejin.cn/post/6909638377247604750)







