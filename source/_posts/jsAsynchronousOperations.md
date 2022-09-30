---
title: js之异步操作入门
excerpt: 关于js异步的简单理解
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## ajax
```javascript
ajax('http://aaa.com/a/1.txt',function(){});
```
异步操作：同时进行多个操作，用户体验好，代码混乱
同步操作：一次只能执行一个操作，用户体验不好，但清晰

```javascript
//异步——麻烦
ajax('http://aaa.com/api/uesr',function(data1){
    ajax('http://aaa.com/api/item',function(data2){
        ajax('http://aaa.com/api/test',function(data3){


        },function(){
            alert('error');
        });
    },function(){
        alert('error');
    });
},function(){
    alert('error');
});
```
```javascript
//同步——容易
let data1=ajax('http://aaa.com/api/uesr');
let data2=ajax('http://aaa.com/api/item');
let data3=ajax('http://aaa.com/api/test');
```

## Promise
融合异步同步
封装的例子：
```javascript
//需要jQuery
let p = new Promise(function(resolve, reject) {
    //异步
    //resolve   解决
    //reject    拒绝
    $.ajax({
        url: 'data/1.txt',
        dataType: 'json',
        success(arr) {
            resolve(arr);
        },
        error(res) {
            reject(res);
        }
    })
})
p.then(function(arr) {
    alert('成功');
    console.log(arr);
}, function(res) {
    alert('失败');
    console.log(res);
});
```
jQuery的ajax本身就是promise，即：
```javascript
//需要jQuery
$.ajax({
    url: 'data/1.txt',
    dataType: 'json'
}).then(arr => {
    alert(arr);
}, res => {
    alert('失败');
})
```

如果不止一个数据，即：
```javascript
//需要jQuery
//只要有一个失败，就全算失败
//只有都成功，才算成功
Promise.all([
    $.ajax({ url: 'data/1.txt', dataType: 'json' }),
    $.ajax({ url: 'data/2.txt', dataType: 'json' }),
    $.ajax({ url: 'data/3.txt', dataType: 'json' })
]).then(arr => {

    alert(arr);

    //解构
    let [data1, data2, data3] = arr;
}, res => {
    alert('失败');
});
//或者
Promise.all([
    $.ajax({ url: 'data/1.txt', dataType: 'json' }),
    $.ajax({ url: 'data/2.txt', dataType: 'json' }),
    $.ajax({ url: 'data/3.txt', dataType: 'json' })
]).then(([data1, data2, data3]) => {
    console.log(data1, data2, data3);
}, res => {
    alert('失败');
});
```

Promise.race 或的意思，只有都失败才会走res
不是所有情况都可以用promise，举个栗子，普通用户普通广告，特殊用户特殊广告：
```javascript
let userData = ajax('http://xxx.com/api/userdata');
if (userData.vip) {
    let data2 = ajax('http://xxx.com/api/vipItem');
    let data3 = ajax('http://xxx.com/api/vipAd');
} else {
    let data2 = ajax('http://xxx.com/api/Item');
    let data3 = ajax('http://xxx.com/api/Ad');
}
```
```javascript
ajax('http://xxx.com/api/userdata').then(userData => {
    if (userData.vip) {
        let data2 = ajax('http://xxx.com/api/vipItem');
        let data3 = ajax('http://xxx.com/api/vipAd');
    } else {
        let data2 = ajax('http://xxx.com/api/Item');
        let data3 = ajax('http://xxx.com/api/Ad');
    }
}, err => {
    alert('error');
});

```
可以看到又回去了= =
这里引出async/await概念

## async/await
简单来说，他是函数的一种特殊形式

```javascript
async function show(){
    xxx;
    xxx;
    let data=await $.ajax();
}
show();
```

普通函数：一直执行，直到结束
async函数：能“暂停”

举个栗子：
```javascript
async function show(){
    let a=10,b=5;
    let data=await $.ajax({url:'data/1.txt',dataType:'json'});
    alert(a+b+data[0]);
}
show();
```

## 参考链接
[【智能社】ES6精讲—主讲老师：石川（Blue）—高清版本](https://www.bilibili.com/video/BV1wt411t7hg)
