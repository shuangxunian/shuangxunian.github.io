---
title: 为什么计算机时间要从1970年1月1日开始算起
excerpt: 奇怪的冷知识又增加了
categories:
- 其他
tags:
- 其他
---

## 最懒的解释
很多编程语言起源于UNIX系统，而UNIX系统认为1970年1月1日0点是时间纪元，所以为偶们常说的UNIX时间戳是以1970年1月1日0点为计时起点时间的。

## 深入的了解
最初计算机操作系统是32位，而时间也是32为表示。
```java
System.out.println(Integer.MAX_VALUE);     //2147483647
```
Integer在java内用32位表示，因此32为能表示的最大值就是2147483647。另外一年365天的总秒数是31536000,2147483647/31536000=68.1，也就是说32为能表示的最长时间是68.1，也就是说32为能表示的最长时间就是68年，从1970年开始的话，加上68.1年，实际最终到2038年01月19日03时14分07秒，便会达到最大时间，过了这个时间点，所有32为操作系统时间便会变为10000000 00000000 00000000 00000000,算下来也就是1901年12月13日20时45分52秒，这样便会出现时间回归的现象，很多软件便会运行异常。

到这里我想问题的答案已经显现出来了，那就是因为用32为来表示时间的最大间隔是68年，而最早出现的UNIX系统考虑到计算机产生的年代个应用的时限，综合取了1970年1月1日作为UNIX TIME的纪元时间（开始时间），至于时间回归现象相信随着64位操作系统可以表示到292,277,026,596年的12月4日14时30分08秒，这是时间已经是千亿年以后了，所以也不用担心了。

## 参考链接
[为什么计算机时间要从1970年1月1日开始算起](https://blog.csdn.net/kang19940713/article/details/60466393/)