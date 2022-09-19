---
title: 一个项目的设计方案
excerpt: 某项目的设计方案
categories:
- 其他
tags:
- 其他
---

## 版心
1400/700，左右留白

## 组件
尺寸：默认顺序宽*高

颜色：在这里所有颜色都用白色，颜色需要根据公司的风格来确定一个底色+内容的颜色

组件：所有组件的div均是直角，实现的过程中需要根据具体位置添加padding/box-shadow/border-radius

1. 页面头部：左右结构，左边公司logo(270px * 100px)，右边一个登录注册的a标签(200px * 100)

    size: 100% * 100px
2. 导航：n个面包屑，n取决于有多少超链接

    size: auto(max:900) * 100px
3. 公司说明：三个标签，img(700px * 500px),p(700px * 500px),img(700px * 500px)，从父组件中拿参数(0/1/2;ps:0
img左p右，1img右p左，2img上p下)用来判断图片在左在右，图片文字不能占100%，应该水平垂直居中(相对于img标签)。[vue项目如何监听窗口变化，达到页面自适应？](https://segmentfault.com/a/1190000016512967)

    size: 1400px * 500px
4. 版权声明：带备案版权等全部信息，这个组件不能固定宽度且没有最大限制

    size: auto * 130px
5. 个人信息：头像+欢迎你+name+天气预报[api](https://www.free-api.com/doc/69)+每日一句[API](http://api.guaqb.cn/v1/onesaid/)，每日一句最好可以走自己的API，这种先暂时用着，各模块位置移步demo。
    说明：天气预报API需要地理位置，现在提供两种解决方案：

    1. 先用`navigator.geolocation.getCurrentPosition`拿到经纬度，再通过高德API将经纬度转为地理位置，再将地理位置发给天气API获取天气，再绘制到页面上，这种方法消耗的是客户端。
    2. 后端获取客户端的IP，再通过IP的[API](https://www.free-api.com/doc/236)获取地理位置，再通过天气API获取天气，再将商议好的json发给前端，前端根据json去渲染页面，这种方法消耗的是服务器，但可以在服务端将IP进行缓存，一个学校的IP可以不再重复请求，会节省消耗。

    size: 700px * 500px

    ![](https://api2.mubu.com/v3/document_image/6aa13990-cc8c-42ef-87a1-b5510e787061-3807603.jpg)

6. 日历：一个日历，右上角定位一个签到按钮，每日签到对应的日期会打对号，签到完文字变成已签到并且不可以再次点击

    size: 700px * 500px

    ![](https://api2.mubu.com/v3/document_image/8222ab21-f60a-492b-bf84-5a9bfc02ee34-3807603.jpg)

7. 当前课程：下方有一个按钮，导向课程页。

    size: 700px * 500px

    ![](https://api2.mubu.com/v3/document_image/859e693f-f32c-4e76-b0bd-886532ca7c0e-3807603.jpg)
    
8. todo：从本地缓存拿数据，要写日报等等，可以自己添加

    size: 700px * 500px

    ![](https://api2.mubu.com/v3/document_image/8b13440e-7e03-4ec1-9d54-05660ef3b26a-3807603.jpg)

9. 登录：默认false(隐藏)，对组件1的a标签绑定一个click事件，回调将值改为true，显示出来，一个大的div，背景为#999999，垂直居中一个div，size:500px * 800px


## 页面
页面均是按从上到下的顺序去实现

### 页面一  公司首页
组件1   页面头部

组件3   公司说明(0)

组件3   公司说明(1)

组件3   公司说明(0)

组件4   版权声明

组件9   登录


![](https://api2.mubu.com/v3/document_image/c07fa101-1be3-4122-938c-c1449b4b7f57-3807603.jpg)

![](https://api2.mubu.com/v3/document_image/f37007e8-3c23-43bb-be96-0c3c798d76d1-3807603.jpg)

### 页面二  个人控制台
组件1   页面头部

组件5   个人信息

组件6   日历

组件7   当前课程

组件8   todo

![](https://api2.mubu.com/v3/document_image/01924291-a32c-41f4-8e66-74555e8622cd-3807603.jpg)


