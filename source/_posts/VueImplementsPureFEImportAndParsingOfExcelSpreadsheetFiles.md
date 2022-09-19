---
title: vue实现纯前端导入与解析excel表格文件
excerpt: 如题目
categories:
- 技术文章
tags:
- vue
---

## 场景
前端导入excel表格，直接前端解析文件，将数据传给后端。

## 需要的库
1. 安装
    `npm install xlsx`
2. 使用
    `import XLXS from "xlsx";`

## 代码实现
1. html部分
    ```html
    <div class="container">
      {{ upload_file || "导入" }}
      <input
        type="file"
        accept=".xls,.xlsx"
        class="upload_file"
        @change="readExcel($event)"
      />
    </div>
    ```
2. JS部分
    ```javascript
    readExcel(e) {
      // 读取表格文件
      let that = this;
      const files = e.target.files;
      if (files.length <= 0) {
        return false;
      } else if (!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())) {
        this.$message({
          message: "上传格式不正确，请上传xls或者xlsx格式",
          type: "warning"
        });
        return false;
      } else {
        // 更新获取文件名
        that.upload_file = files[0].name;
      }

      const fileReader = new FileReader();
      fileReader.onload = ev => {
        try {
          const data = ev.target.result;
          const workbook = XLSX.read(data, {
            type: "binary"
          });
          const wsname = workbook.SheetNames[0]; //取第一张表
          const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); //生成json表格内容
					console.log(ws);
        } catch (e) {
          return false;
        }
      };
      fileReader.readAsBinaryString(files[0]);
    }
    ```
3. 整体代码
```vue
<template>
<div>
    <div class="container">
    {{ upload_file || "导入" }}
    <input
        type="file"
        accept=".xls,.xlsx"
        class="upload_file"
        @change="readExcel($event)"
    />
    </div>
</div>
</template>

<script>
import XLSX from "xlsx";

export default {
data() {
    return {
    upload_file: "",
    lists: []
    };
},
methods: {
    submit_form() {
    // 给后端发送请求，更新数据
    console.log("假装给后端发了个请求...");
    },
    readExcel(e) {
    // 读取表格文件
    let that = this;
    const files = e.target.files;
    if (files.length <= 0) {
        return false;
    } else if (!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())) {
        this.$message({
        message: "上传格式不正确，请上传xls或者xlsx格式",
        type: "warning"
        });
        return false;
    } else {
        // 更新获取文件名
        that.upload_file = files[0].name;
    }

    const fileReader = new FileReader();
    fileReader.onload = ev => {
        try {
        const data = ev.target.result;
        const workbook = XLSX.read(data, {
            type: "binary"
        });
        const wsname = workbook.SheetNames[0]; //取第一张表
        const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); //生成json表格内容
        that.lists = [];
        // 从解析出来的数据中提取相应的数据
        ws.forEach(item => {
            that.lists.push({
            username: item["用户名"],
            phone_number: item["手机号"]
            });
        });
        // 给后端发请求
        this.submit_form();
        } catch (e) {
        return false;
        }
    };
    fileReader.readAsBinaryString(files[0]);
    }
}
};
</script>
```

## 样式
原本的文件上传样式可能会跟页面整体风格不搭，所以需要修改其样式。不过此处并不是直接修改其样式而是通过写一个div来覆盖原有的上传按钮。此处样式与element UI中的primary按钮样式相同。

实现该样式的关键在于.upload_file的opacity和position。

## 最后
前端的日益强大导致很多功能都可以在前端去直接实现，并且可以减少服务器压力。

当然单纯地去实现这样的数据传输，尤其对于重要数据，是很不安全的，因此在前后端数据传输的时候，可以加上加密校验，这个后期会来写的。

## 参考链接
[vue实现纯前端导入与解析excel表格文件](https://zhuanlan.zhihu.com/p/114607174)