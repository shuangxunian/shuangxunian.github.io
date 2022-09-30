---
title: js之Promise封装nodejs的request实现
excerpt: 如题目~
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

```javascript
var request = require('request');   // 引入nodejs的request模块
// 请求参数
var requestData = {
    city_name:"北京市",
    language:"Zh_CN"
}
// 定义Http 请求方法的实现
function requestGet(requestData){
    return new Promise((resolve, reject)=>{
        var url = "http://localhost:8888/";
        var option ={
            url: url,
            method: "GET",   //指定请求方法类型：GET, POST
            json: true,
            timeout: 30000,  // 设置请求超时，单位是毫秒  
            headers: {
                "content-type": "application/json",
            },
            qs: requestData    // 进行GET请求时，此处的参数一定是qs,请注意，如果是POST请求，参数是form
        }
        request(option, function(error, response, body) {
            if (!error && response.statusCode == 200) {
                resolve(body)   // 返回response的内容
            }else{
                reject(error);   // 返回错误信息
            }
        });
    });
 
};
// 调用request的Get请求，Promise对请求进行封装，返回Promise
 
requestGet(requestData).then( data=>{
    console.log(data);   // 对请求结果进行业务处理
}).catch(ex=>{
    console.log('catch');        // 此处可处理错误信息
}).finally(()=>{
    console.log('finally');      // 进行最后的处理
});
```