---
title: ES6笔记
excerpt: 关于ES6的一点自己的理解
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- 面试
---


## 前言
记的比较随便，格式可能不太好看，有需要补充的地方请私聊我~

## var
1. 重复声明
```javascript
var a=12;
var a=5;
```
上边不会报错，最后a=5，在大型项目中如果重复声明一个变量直接崩盘。

2. 控制修改
```javascript
var GIT_HOST='https://github.com/YuelinWang';
if(GIT_HOST='error'){

}
```
上边会直接把内容修改掉并进入到if中，敲的时候就要特别小心

3. 块级作用域
let     防止重复声明，变量
const   防止重复声明，常量

for循环中如果用var，想要console每一个i只简单的写是实现不了的- -，所以用闭包来实现。
```javascript
let bt=document.gerElementsByTagName('input');
for(var i=0;i<bt.length;i++){
    (function(i){
        bt[i].onclick=function(){
            alert(i);
        };
    })(i);
}
```

我是真的烦闭包，一直弄不太清楚，但是如果用let，可以直接写，因为块级作用域
```javascript
let bt=document.gerElementsByTagName('input');
for(let i=0;i<bt.length;i++){
    bt[i].onclick=function(){
        alert(i);
    };
}
```

ES5的var的作用域——函数级
ES6的let的作用域——块级

块级即一个代码块——{}

```javascript
eg:
for(){

}
if(){

}
{

}
```
以上三种均是代码块

**小结：**
var     重复声明，不能限制修改，函数级
let     不能，变量，块级
const   不能，常量，块级

## 解构赋值
正常：
```javascript
let a=1;
let b=2;
```

解构赋值：
```javascript
let json={a:12,b:5};
let {a,b}=json;

let arr=[12,5,8];
let [a,b,c]=arr;
```

注意：
1. 两边结构一样
2. 右边必须是个东西
3. 赋值和解构必须同时完成

## 箭头函数
```javascript
function(){

}
()=>{

}
```

举个栗子
```javascript
//排序
//原数组
let arr=[12,0,5,9,22,4];
//原来
arr.sort(function(n1,n2){
    return n1-n2;
});
//now
arr.sort((n1,n2)=>{return n1-n2;});
```

**简写** 
注意事项：
1. 如果有且仅有一个参数，()也可以不写。
2. 如果有且仅有一个语句，花括号也可以不写，注意必须是return，还是上边的例子。
举个栗子
```javascript
//排序
//原数组
let arr=[12,0,5,9,22,4];
//原来
arr.sort(function(n1,n2){
    return n1-n2;
});
//now
arr.sort((n1,n2)=>{return n1-n2;});
//简写
arr.sort((n1,n2)=>n1-n2;);
```

**修正this**


## ...
参数/数组/
1. 收集
2. 展开
举个栗子：
```javascript
//收集
function show(a,b,...c){
    console.log(a,b,c);
}
show(1,2,3,4,5,6,10,7);
//1进a，2进b，剩下全进c，成一个数组

//展开
let arr = [12,5,8];
function show(a,b,c){
    alert(a+b+c);
}
show(...arr);//25

//合并
let arr1 = [3,4,6];
let arr2 = [...arr,...arr1];
alert(arr2);
```
```javascript
//json
let json1={a:12,b:5,c:9};
let json2={
    ...json1,
    d:12
}
console.log(json2);
```

## map
映射，一一对应
```javascript
let arr=[68,53,12,98,64];

//第一种
let arr2=arr.map(function(item){
    if(item>=60) return '及格';
    else return '不及格';
    return item>=60?'及格'?'不及格';
});

//第二种
let arr2=arr.map(function(item){
    return item>=60?'及格'?'不及格';
});

//第三种
let arr2=arr.map(item=>item>=60?'及格'?'不及格');


console.log(arr);
console.log(arr2);
```

## reduce
n个进入，出来1个
```javascript
//求平均数
let arr=[68,53,12,98,64];
//tmp临时
//当前数
//当前位置
//第一种
let re=arr.reduce(function(tmp,item,index){
    return tmp+item;
});
alert(re/arr.length);

//第二种
let re=arr.reduce(function(tmp,item,index){
    if(index===arr.length-1) return (tmp+item)/arr.length;
    else return tmp+item;
});
alert(re);
```

## filter
过滤
```javascript
//求平均数
let arr=[68,53,12,98,64];

//1
let arr2=arr.filter(item=>{
    if(item%2) return false;
    else return true;
});

//2
let arr2=arr.filter(item=>{
    return item%2===0;
});

//3
let arr2=arr.filter(item=>item%2===0);

console.log(arr2);
```

## forEach
没有返回值，遍历
```javascript
let arr=[68,53,12,98,64];

arr.forEach((item,index)=>{
    //1
    alert('第'+index+'个：'+item);
    //2     反单引号    左上角1左边的
    alert(`第${index}个：${item}`);

});
```

## json
```javascript
//注意都要有双引号
JSON.stringify({a:12,b:5})      =>  '{"a":12,"b":5}'
JSON.parse('{"a":12,"b":5}')    =>  {a:12,b:5}
```

## 参考链接
[【智能社】ES6精讲—主讲老师：石川（Blue）—高清版本](https://www.bilibili.com/video/BV1wt411t7hg)