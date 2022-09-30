---
title: js迭代器实现斐波那契
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 介绍
斐波那契数列（Fibonacci sequence），又称黄金分割数列、因数学家莱昂纳多·斐波那契（Leonardoda Fibonacci）以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：0、1、1、2、3、5、8、13、21、34、……在数学上，斐波那契数列以如下被以递推的方法定义：F(0)=0，F(1)=1, F(n)=F(n - 1)+F(n - 2)（n ≥ 2，n ∈ N*）在现代物理、准晶体结构、化学等领域，斐波纳契数列都有直接的应用，为此，美国数学会从 1963 年起出版了以《斐波纳契数列季刊》为名的一份数学杂志，用于专门刊载这方面的研究成果。

## c++
斐波那契在一些算法比赛中比较喜欢作为签到题出现，直接给出代码：
```c++
int n;
while(cin>>n){
    int a=1,b=1,c;
    for(int i=3;i<=n;i++){
        c=a+b;
        a=b;
        b=c;
    }
    cout<<(n>2?c:1)<<endl;
}
```

## js(迭代器)
正常写和c++写没啥区别，这里就不重复了，这里用一下迭代器给大家实现
```javascript
let fb = () => {
    let arr = [0, 1];
    let index = 0;
    return {
        next() {
            let value = arr[index];
            if (value === undefined) {
                value = arr[index - 1] + arr[index - 2];
                arr[index] = value;
            }
            index += 1;
            console.log(value);
            return {
                value: value,
                done: false
            }
        }
    }
}

let fn = fb();
fn.next();
fn.next();


//yeild

let fb = function*() {
    let arr = [0, 1];
    let index = 0;
    while (true) {
        let value = arr[index];
        if (value === undefined) {
            value = arr[index - 1] + arr[index - 2];
            arr[index] = value;
        }
        yield value;
        index += 1;
    }
}

let fn = fb();
fn.next();

```

