---
title: 操作系统期末考试——5
excerpt: 本文涉及到文件管理
categories:
- 考试
tags:
- 操作系统
- 考试
---

## 文件定义
**定义1**
文件是指由创建者所定义的、具有文件名的一组相关元素的集合，可分为有结构文件和无结构文件；
- 有结构文件
若干个相关记录组成的文件
- 无结构文件
字符流文件

**定义2**
文件是为了某种目的而组织起来的信息集合。

**理解**
文件是计算机组织信息的单位。
文件是有一定格式的，由若干个信息的逻辑单元组成。
这些逻辑单元的结构与意义由不同的软件、不同的用户进行具体解释。
数据库文件：每个逻辑单元（一个记录）表示一组信息；
数据文件：每个逻辑单元表示一个数据，如实数、整数等；
程序文件：每个逻辑单元表示一条指令；
文本文件：每个逻辑单元是一行字符串

## 文件逻辑结构定义及分类

从用户（应用软件程序员）角度看到的文件组织形式；
文件的逻辑结构与存储设备特性无关；

**分类：**
- 无结构文件
长度以字节为单位。把流式文件看作是记录式文件的一个特例。
例如：可执行文件、 库函数等
对流式文件的访问，采用读写指针来指出下一个要访问的字符。
- 有结构文件
若干记录组成
记录是信息逻辑单元
核心问题：如何根据关键值快速定位文件中的记录

![](https://api2.mubu.com/v3/document_image/8551f572-88aa-4cea-812c-66ea14015d08-3807603.jpg)

## 文件的物理结构及分类
又称为文件的存储结构，是指文件在外存的存储组织形式。
文件的物理结构与存储设备的特性关系密切。

## 文件控制块
文件控制块（FCB）是操作系统为管理文件而设置的数据结构；
存放管理文件所需有关信息（文件属性）；
文件管理和访问必须从FCB中得到必要的文件属性信息，至少是文件名和起始存储地址。

## 文件目录的定义（文件控制块的有序集合）
**文件目录**：把所有的FCB组织在一起，就构成了文件目录，即文件控制块的有序集合；
**目录项**：一个目录项就是一个文件的FCB；
**目录文件**：为了实现对文件目录的管理，通常将文件目录以文件的形式保存在外存，这个文件就叫目录文件；

## 索引节点概念、引入索引节点的原因
在UNIX系统中，采用了把文件名与文件描述信息分开的办法，使文件描述信息单独形成一个称为索引结点的数据结构，简称为i结点。
在文件目录中的每个目录项，仅由文件名和指向该文件所对应的i结点的指针所构成。
文件目录通常存放在磁盘上，当文件很多时，文件目录可能要占用大量的盘块，查找一个文件需启动磁盘多次；
解决方案：精简目录，去除文件的描述信息。