---
title: 如何维护更新日志
excerpt: 更新日志绝对不应该是git日志的堆砌物
date: 2022-09-30
categories:
- 技术文章
tags:
- 软实力
---

## 更新日志是什么？
更新日志（Change Log）是一个由人工编辑，以时间为倒序的列表， 以记录一个项目中所有版本的显著变动。

## 为何要提供更新日志？
为了让用户和开发人员更简单明确的知晓项目在不同版本之间有哪些显著变动。

## 哪些人需要更新日志？
人人需要更新日志。无论是消费者还是开发者，软件的最终用户都关心软件所包含什么。 当软件有所变动时，大家希望知道改动是为何、以及如何进行的。

## 怎样制作高质量的更新日志？
### 指导原则
- 记住日志是写给人的，而非机器。
- 每个版本都应该有独立的入口。
- 同类改动应该分组放置。
- 版本与章节应该相互对应。
- 新版本在前，旧版本在后。
- 应包括每个版本的发布日期。
- 注明是否遵守语义化版本格式.

### 变动类型
- Added 新添加的功能。
- Changed 对现有功能的变更。
- Deprecated 已经不建议使用，准备很快移除的功能。
- Removed 已经移除的功能。
- Fixed 对bug的修复
- Security 对安全的改进

## 如何减少维护更新日志的精力？
在文档最上方提供 Unreleased 区块以记录即将发布的更新内容。

这样有两大意义：
- 大家可以知道在未来版本中可能会有哪些变更
- 在发布新版本时，可以直接将Unreleased区块中的内容移动至新发 布版本的描述区块就可以了

## 有很糟糕的更新日志吗？
当然有，下面就是一些糟糕的方式。

### 使用git日志
使用git日志作为更新日志是个非常糟糕的方式：git日志充满各种无意义的信息， 如合并提交、语焉不详的提交标题、文档更新等。

提交的目的是记录源码的演化。 一些项目会清理提交记录，一些则不会。

更新日志的目的则是记录重要的变更以供最终受众阅读，而记录范围通常涵盖多次提交。

### 无视即将弃用功能
当从一个版本升级至另一个时，人们应清楚（尽管痛苦）的知道哪些部分将出现问题。 应该允许先升级至一个列出哪些功能将会被弃用的版本，待去掉那些不再支持的部分后， 再升级至把那些弃用功能真正移除的版本。

即使其他什么都不做，也要在更新日志中列出derecations，removals以及其他重大变动。

### 易混淆的日期格式
在美国，人们将月份写在日期的开头(06-02-2012对应2012年6月2日)， 与此同时世界上其他地方的很多人将至写作2 June 2012，并拥有不同发音。 2012-06-02从大到小的排列符合逻辑，并不与其他日期格式相混淆，而且还 符合ISO标准。因此，推荐在更新日志中采用使用此种日期格式。

## 常见问题
### 是否有一个标准化的更新日志格式？
并没有。虽然GNU提供了更新日志样式指引，以及那个仅有两段长的GNU NEWS文件“指南”， 但两者均远远不够。

此项目意在提供一个 更好的更新日志惯例 所有点子都来自于在开源社区中对优秀实例的观察与记录。

对于所有建设性批评、讨论及建议，我们都非常 欢迎。

### 更新日志文件应被如何命名？
可以叫做CHANGELOG.md。 一些项目也使用 HISTORY、NEWS或RELEASES。

当然，你可以认为更新日志的名字并不是什么要紧事，但是为什么要为难那些仅仅是 想看到都有哪些重大变更的最终用户呢？

### 对于GitHub发布呢？
这是个非常好的倡议。Releases可通过手动添加发布日志或将带 有注释的git标签信息抓取后转换的方式，将简单的git标签（如一个叫v1.0.0的标签） 转换为信息丰富的发布日志。

GitHub发布会创建一个非便携、仅可在GitHub环境下显示的更新日志。尽管会花费更 多时间，但将之处理成更新日志格式是完全可能的。

现行版本的GitHub发布不像哪些典型的大写文件(README, CONTRIBUTING, etc.)，仍可以认为是不利于最终用户探索的。 另一个小问题则是界面并不提供不同版本间commit日志的链接。

### 更新日志可以被自动识别吗？
非常困难，因为有各种不同的文件格式和命名。

Vandamme 是一个Ruby程序，由 Gemnasium 团队制作，可以解析多种 （但绝对不是全部）开源库的更新日志。

### 那些后来撤下的版本怎么办？
因为各种安全/重大bug原因被撤下的版本被标记'YANKED'。 这些版本一般不出现在更新日志里，但建议他们出现。 显示方式应该是：

> ## 0.0.5 - 2014-12-13 [YANKED]

[YANKED] 的标签应该非常醒目。 人们应该非常容易就可以注意到他。 并且被方括号所包围也使其更易被程序识别。

### 是否可以重写更新日志？
当然可以。总会有多种多样的原因需要我们去改进更新日志。 对于那些有着未维护更新日志的开源项目，我会定期打开pull请求以加入缺失的发布信息。

另外，很有可能你发现自己忘记记录一个重大功能更新。这种情况下显然你应该去重写更新日志。

## 参考链接
[如何维护更新日志](https://keepachangelog.com/zh-CN/1.0.0/)