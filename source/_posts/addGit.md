---
title: git初使用
excerpt: 一篇git入门的文章，内容不多好理解，但不能作为精通git的文章
categories:
- 技术文章
tags:
- git
---

## 前言
这篇只作为入门给大家看，并不是很详细，仅供正常合作没什么问题，如果想深研究请自行搜索学习

## 安装Git
最早Git是在Linux上开发的，很长一段时间内，Git也只能在Linux和Unix系统上跑。不过，慢慢地有人把它移植到了Windows上。现在，Git可以在Linux、Unix、Mac和Windows这几大平台上正常运行了。
要使用Git，第一步当然是安装Git了。根据你当前使用的平台来阅读下面的文字：
 
### 在Windows上安装Git
在Windows上使用Git，可以从Git官网直接下载安装程序，然后按默认选项安装即可。
安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！
安装完成后，还需要最后一步设置，在命令行输入：
![](https://api2.mubu.com/v3/document_image/245f6e7e-7259-4041-b9b9-2839c533c032-3807603.jpg)
因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。你也许会担心，如果有人故意冒充别人怎么办？这个不必担心，首先我们相信大家都是善良无知的群众，其次，真的有冒充的也是有办法可查的。
注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。
 
### 在Mac OS X上安装Git
如果你正在使用Mac做开发，有两种安装Git的方法。
一是安装homebrew，然后通过homebrew安装Git，具体方法请参考homebrew的文档：http://brew.sh/。
第二种方法更简单，也是推荐的方法，就是直接从AppStore安装Xcode，Xcode集成了Git，不过默认没有安装，你需要运行Xcode，选择菜单“Xcode”->“Preferences”，在弹出窗口中找到“Downloads”，选择“Command Line Tools”，点“Install”就可以完成安装了。
![](https://api2.mubu.com/v3/document_image/bfbd2b82-fa4b-4bc5-93cb-99152b373189-3807603.jpg)
Xcode是Apple官方IDE，功能非常强大，是开发Mac和iOS App的必选装备，而且是免费的！
 
### 在Linux上安装Git
首先，你可以试着输入git，看看系统有没有安装Git：
![](https://api2.mubu.com/v3/document_image/bfbdc18a-8e98-462c-80a1-de9ce54c36ce-3807603.jpg)
像上面的命令，有很多Linux会友好地告诉你Git没有安装，还会告诉你如何安装Git。
如果你碰巧用Debian或Ubuntu Linux，通过一条sudo apt-get install git就可以直接完成Git的安装，非常简单。
老一点的Debian或Ubuntu Linux，要把命令改为sudo apt-get install git-core，因为以前有个软件也叫GIT（GNU Interactive Tools），结果Git就只能叫git-core了。由于Git名气实在太大，后来就把GNU Interactive Tools改成gnuit，git-core正式改为git。
如果是其他Linux版本，可以直接通过源码安装。先从Git官网下载源码，然后解压，依次输入：./config，make，sudo make install这几个命令安装就好了。
 
## 创建版本库
什么是版本库呢？版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。
所以，创建一个版本库非常简单，首先，选择一个合适的地方，创建一个空目录：
![](https://api2.mubu.com/v3/document_image/d10bdcee-88f9-4344-af3f-63d8c7b07314-3807603.jpg)
pwd命令用于显示当前目录。在我的Mac上，这个仓库位于/Users/michael/learngit
如果你使用Windows系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文。
第二步，通过git init命令把这个目录变成Git可以管理的仓库
![](https://api2.mubu.com/v3/document_image/467d150a-6c8a-4f17-8e04-7ae7b46ca31e-3807603.jpg)
瞬间Git就把仓库建好了，而且告诉你是一个空的仓库（empty Git repository），细心的读者可以发现当前目录下多了一个.git的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。
如果你没有看到.git目录，那是因为这个目录默认是隐藏的，用ls -ah命令就可以看见。
也不一定必须在空目录下创建Git仓库，选择一个已经有东西的目录也是可以的。不过，不建议你使用自己正在开发的公司项目来学习Git，否则造成的一切后果概不负责。

## 把文件添加到版本库
首先这里再明确一下，所有的版本控制系统，其实只能跟踪文本文件的改动，比如TXT文件，网页，所有的程序代码等等，Git也不例外。版本控制系统可以告诉你每次的改动，比如在第5行加了一个单词“Linux”，在第8行删了一个单词“Windows”。而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只知道图片从100KB改成了120KB，但到底改了啥，版本控制系统不知道，也没法知道。
不幸的是，Microsoft的Word格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的，前面我们举的例子只是为了演示，如果要真正使用版本控制系统，就要以纯文本方式编写文件。
因为文本是有编码的，比如中文有常用的GBK编码，日文有Shift_JIS编码，如果没有历史遗留问题，强烈建议使用标准的UTF-8编码，所有语言使用同一种编码，既没有冲突，又被所有平台所支持。
**使用Windows特别注意：**千万不要使用Windows自带的记事本编辑任何文本文件。原因是Microsoft开发记事本的团队使用了一个非常弱智的行为来保存UTF-8编码的文件，他们自作聪明地在每个文件开头添加了0xefbbbf（十六进制）的字符，你会遇到很多不可思议的问题，比如，网页第一行可能会显示一个“?”，明明正确的程序一编译就报语法错误，等等，都是由记事本的弱智行为带来的。
现在我们编写一个readme.txt文件，内容如下：
![](https://api2.mubu.com/v3/document_image/23f6d9e7-9269-412a-914e-43008357728a-3807603.jpg)
一定要放到learngit目录下（子目录也行），因为这是一个Git仓库，放到其他地方Git再厉害也找不到这个文件。和把大象放到冰箱需要3步相比，把一个文件放到Git仓库只需要两步。
第一步，用命令git add告诉Git，把文件添加到仓库：
![](https://api2.mubu.com/v3/document_image/31d39cf6-664e-4ab1-a925-3cf4142cdf8f-3807603.jpg)
执行上面的命令，没有任何显示，这就对了，Unix的哲学是“没有消息就是好消息”，说明添加成功。
第二步，用命令git commit告诉Git，把文件提交到仓库：
![](https://api2.mubu.com/v3/document_image/fb60f7af-90d8-427a-94eb-b717be59b7d1-3807603.jpg)
简单解释一下git commit命令，-m后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。
git commit命令执行成功后会告诉你，1 file changed：1个文件被改动（我们新添加的readme.txt文件）；2 insertions：插入了两行内容（readme.txt有两行内容）。
为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件，比如：
![](https://api2.mubu.com/v3/document_image/e03d0616-8b99-4ffb-9e1e-8c1e169f1bbd-3807603.jpg)

## 远程仓库
到目前为止，我们已经掌握了如何在Git仓库里对一个文件进行添加。
可是有用过集中式版本控制系统SVN的童鞋会站出来说，这些功能在SVN里早就有了，没看出Git有什么特别的地方。
没错，如果只是在一个仓库里管理文件历史，Git和SVN真没啥区别。为了保证你现在所学的Git物超所值，将来绝对不会后悔，同时为了打击已经不幸学了SVN的童鞋，本章开始介绍Git的杀手级功能之一（注意是之一，也就是后面还有之二，之三……）：远程仓库。
Git是分布式版本控制系统，同一个Git仓库，可以分布到不同的机器上。怎么分布呢？最早，肯定只有一台机器有一个原始版本库，此后，别的机器可以“克隆”这个原始版本库，而且每台机器的版本库其实都是一样的，并没有主次之分。
你肯定会想，至少需要两台机器才能玩远程库不是？但是我只有一台电脑，怎么玩？
其实一台电脑上也是可以克隆多个版本库的，只要不在同一个目录下。不过，现实生活中是不会有人这么傻的在一台电脑上搞几个远程库玩，因为一台电脑上搞几个远程库完全没有意义，而且硬盘挂了会导致所有库都挂掉，所以我也不告诉你在一台电脑上怎么克隆多个仓库。
实际情况往往是这样，找一台电脑充当服务器的角色，每天24小时开机，其他每个人都从这个“服务器”仓库克隆一份到自己的电脑上，并且各自把各自的提交推送到服务器仓库里，也从服务器仓库中拉取别人的提交。
完全可以自己搭建一台运行Git的服务器，不过现阶段，为了学Git先搭个服务器绝对是小题大作。好在这个世界上有个叫GitHub的神奇的网站，从名字就可以看出，这个网站就是提供Git仓库托管服务的，所以，只要注册一个GitHub账号，就可以免费获得Git远程仓库。
在继续阅读后续内容前，请自行注册GitHub账号。由于你的本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以，需要一点设置：
1. 创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
![](https://api2.mubu.com/v3/document_image/b5f4936f-958e-4737-afd1-2cd5a335a03f-3807603.jpg)
你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。
如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。
2. 登陆GitHub，打开“settings”：
![](https://api2.mubu.com/v3/document_image/b2cdb805-43bd-42a1-b545-d2aba99a1950-3807603.jpg)
找到左边的SSH and GPG keys
![](https://api2.mubu.com/v3/document_image/4ae162a5-aa39-4447-86db-b31eb3794627-3807603.jpg)
点击右上角的New SSH key
![](https://api2.mubu.com/v3/document_image/d1139079-6540-4bb2-9329-10691c50d97f-3807603.jpg)
然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容：
点“Add Key”，你就应该看到已经添加的Key

为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。
当然，GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。
最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但只有你自己才能改）。所以，不要把敏感信息放进去。
如果你不想让别人看到Git库，有两个办法，一个是交点保护费，让GitHub把公开的仓库变成私有的，这样别人就看不见了（不可读更不可写）。另一个办法是自己动手，搭一个Git服务器，因为是你自己的Git服务器，所以别人也是看不见的。这个方法我们后面会讲到的，相当简单，公司内部开发必备。
确保你拥有一个GitHub账号后，我们就即将开始远程仓库的学习。

## 添加远程库
现在的情景是，你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作，真是一举多得。
首先，登陆GitHub，然后，在右上角找到加号按钮，点击里面的“New repository”按钮，创建一个新的仓库：
在Repository name填入learngit，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库：
目前，在GitHub上的这个learngit仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。
现在，我们根据GitHub的提示，在本地的learngit仓库下运行命令：
![](https://api2.mubu.com/v3/document_image/8032fb8d-4e51-4b09-8eaf-3baa4ee273f8-3807603.jpg)
请千万注意，把上面的YuelinWang替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
下一步，就可以把本地库的所有内容推送到远程库上：
![](https://api2.mubu.com/v3/document_image/f3aa095c-98b0-4ba4-861e-ada0c2613c41-3807603.jpg)
把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。
由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样
从现在起，只要本地作了提交，就可以通过命令
![](https://api2.mubu.com/v3/document_image/be2a6acb-48d7-40e2-aa19-59dec1c38dfe-3807603.jpg)
把本地master分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！

## 从远程库克隆
上次我们讲了先有本地库，后有远程库的时候，如何关联远程库。
现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。
首先，登陆GitHub，创建一个新的仓库，名字叫second：
我们勾选Initialize this repository with a README，这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件：
现在，远程库已经准备好了，下一步是用命令git clone克隆一个本地库：
![](https://api2.mubu.com/v3/document_image/8c0354cc-6345-4e8f-9e91-1345a8afcccf-3807603.jpg)
![](https://api2.mubu.com/v3/document_image/934886d9-f4d6-4358-8045-de8a04bca154-3807603.jpg)
注意把Git库的地址换成你自己的，然后进入second目录看看，已经有README.md文件了：
![](https://api2.mubu.com/v3/document_image/0aa02e0f-71c5-45bf-9541-91d201a4b4e6-3807603.jpg)
如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。
GitHub给出的地址不止一个，还可以用 *https://github.com/YuelinWang/wyl_wheel.git* 这样的地址。实际上，Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议。
使用https除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https。

## 可能会遇到的问题
### Git之右键没有Git Bash Here的解决办法
1. Win+R 打开运行输入regedit 回车打开注册表
2. 找到[HKEY_CLASSES_ROOT\Directory\Background]。
3. 在[Background]下如果没有[shell],则右键-新建项[shell]。
4. 在[shell]下右键-新建项[Git Bash Here],其值为“Git Bash Here",此为右键菜单显示名称。此时在任意位置鼠标右击就能看到Git Bash Here但是没有关联程序，现在还没有实际作用
5. 在[Git Bash Here]下右键-新建-项[command],其默认值为 "git安装根目录\bin\bash.exe"
6. 再完善一下添加一个Git的小图标：选中Git Bash Here右击-新建-字符串值，名称为Icon,双击编辑，其值为“git安装根目录\\mingw64\share\git\git-for-windows.ico”。

### SSH警告
当你第一次使用Git的clone或者push命令连接GitHub时，会得到一个警告：
![](https://api2.mubu.com/v3/document_image/b5d1b48c-8707-44e3-bc0d-cd33e6266335-3807603.jpg)
这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。
Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：
![](https://api2.mubu.com/v3/document_image/7c266ca0-44fd-4457-8835-4ae200b4380f-3807603.jpg)
这个警告只会出现一次，后面的操作就不会有任何警告了。
如果你实在担心有人冒充GitHub服务器，输入yes前可以对照GitHub的RSA Key的指纹信息是否与SSH连接给出的一致。

## 后记
再强调下，这里写的这点知识只能作为自己简单使用，万万不可认为自己已熟悉git，此文整理自自己原来整理的[Git](https://mubu.com/docrDFpTiS_c0)，同时有和此文相关的[视频](https://www.bilibili.com/video/BV1rp4y1C7n1)，童鞋们可以移步看一下，最后推荐一下我的[个人github](https://github.com/YuelinWang)，欢迎follow~