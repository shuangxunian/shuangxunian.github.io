---
title: CSS让文字变细
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- css
---

## 内容
影响文字的粗细的其实有三个因素:
- font-weight
- font-family
- -webkit-font-smoothing

要理解CSS能通过font-weight设置字体的粗细, 首先要明白CSS字体的基本设定. 以字体Times字体为例, Times实际是一个字体系列, 它由多种字体变形组合而成, 包括 TimesRegular, TimesBold, TimesItalic等. 每种变形都是一种具体的字体风格(font face).

当font-weight变小时, 浏览器会在指定字体系列中, 找到更细的字体变种. 因而如果使用的字体系列中, 只有一种变种, 则无论font-weight怎么变化, 字体的粗细都不会有变化的

因此文字的粗细还跟font-family有关

但就算设置了以上两项, 我在Mac的Chrome浏览网页时, 还是觉得文字有点”粗”. 这种”粗”用专业的话来说就是, “字体存在半个像素的锯齿，浏览器渲染的时却直接显示一个像素”, 则此时需要对文字进行”抗锯齿处理”. 也即设置为	：
- -webkit-font-smoothing: antialiased;

> 注意到这是个带-webkit前缀的属性, 因此Firefox/IE以及基于IE改造的国产浏览器是没有效果的

## 参考链接
[CSS让文字变细](http://levy.work/2016-09-30-css-make-font-weight-lighter/)
