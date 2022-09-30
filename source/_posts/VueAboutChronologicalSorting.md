---
title: vue关于时间顺序排序
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- vue
---

## 前言
项目有一个需求：在使用后台传来的数据需要以对象时间进行升序或者降序进行排列

## 解决方案
我所传入的时间类似于：2018-07-14 17:31:21，通过replace()函数，将其转换为：2018/07/14 17:31:21 ，

再通过getTime()方法返回距1970年1月1日之间的毫秒数

通过sort()函数进行比较替换，具体代码如下：
```javascript
changeDate() {
    this.giftList.sort((a, b) => {
        let aTimeString = a.createTime
        let bTimeString = b.createTime
        aTimeString = aTimeString.replace(/-/g,'/')     // 将其中的'-'替换为'/'
        bTimeString = bTimeString.replace(/-/g,'/')
        let aTime = new Date(aTimeString).getTime()     // 转为1970年的毫秒
        let bTime = new Date(bTimeString).getTime()
        return bTime - aTime                            // 降序排列
    })
}

```

其中的giftList为res.data，可以再template中直接v-for循环

## 参考链接
[vue关于时间顺序排序](https://blog.csdn.net/weixin_35252758/article/details/81045787)