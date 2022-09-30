---
title: 开发文档
excerpt: 如题目
date: 2022-09-30
categories:
- 其他
tags:
- 其他
---

# baseUrl

/lying-web

## /paper/userLegal

- 登录
- 方法：post
- 传入：
  - 姓名：stuName
  - 学号：stuNo
  - 密码：passWord
- 接收：
  - stuId
  - studentName
  - studentNo
  - ruleId
  - checkCount
  - checkYear
  - token

## /paper/resetpassword

- 修改密码
- 方法：post
- 传入：
  - oldPassWord
  - newPassWord
  - stuId
  - stuNo
  - token
- 接收
  - 状态码

## /paper/userLogout

- 登出
- 方法：post
- 传入：
  - 无
- 接收：
  - 无

## /paper/downloadResult

- 下载文档
- 传入
  - filename
  - checkYear
- 接收
  - response

## /paper/queryResult

- 请求列表
- 传入
  - stuName
  - stuNo
  - checkCount
  - token
- 接收
  - 一个列表：resQ.data.list，内含processState
- eg
  - ![](https://api2.mubu.com/v3/document_image/09358fe5-e59a-481d-bd27-ed8b48a4fb1b-3807603.jpg)

## /paper/uploadPaper

- 上传文件
- 传入
  - formData
- 接收
  - success：true/false