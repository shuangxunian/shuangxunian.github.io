---
title: Element el-upload上传组件详解
excerpt: 如题目
categories:
- 技术文章
tags:
- element
- vue
---

## 前言
upload上传是前端开发很常用的一个功能，在Vue开发中常用的Element组件库也提供了非常好用的upload组件。

## 基本用法
**代码：**
```html
<el-upload :action="uploadActionUrl">
    <el-button size="small" type="primary">点击上传</el-button>
</el-upload>
```
这个基本不用说，:action是执行上传动作的后台接口，el-button是触发上传的按钮。

## 上传文件数量
首先是设置是否可以同时选中多个文件上传，这个也是&lt;input type='file'&gt;的属性，添加multiple即可。另外el-upload组件提供了:limit属性来设置最多可以上传的文件数量，超出此数量后选择的文件是不会被上传的。:on-exceed绑定的方法则是处理超出数量后的动作。代码如下：
```html
<el-upload 
    :action="uploadActionUrl"
    multiple
    :limit="3"
    :on-exceed="handleExceed">
    <el-button size="small" type="primary">点击上传</el-button>
</el-upload>
```

## 上传格式及大小限制
如果需要限制上传文件的格式，需要添加accept属性，这个是直接使用&lt;input type='file'&gt;一样的属性了，accept属性的值可以是accept="image/gif, image/jpeg, text/plain, application/pdf"等等。文件格式的提示，则可以使用slot。代码如下：
```html
<el-upload 
    :action="uploadActionUrl"
    accept="image/jpeg,image/gif,image/png"
    multiple
    :limit="3"
    :on-exceed="handleExceed">
    <el-button size="small" type="primary">点击上传</el-button>
    <div slot="tip" class="el-upload__tip">请上传图片格式文件</div>
</el-upload>
```
注意这里只是选择文件时限制格式，其实用户还是可以点选“所有文件”选项，上传其他格式。如果需要在在上传时再次校验，择需要在:before-upload这个钩子绑定相应的方法来校验，代码如下：
```html
<el-upload 
    :action="uploadActionUrl"
    accept="image/jpeg,image/gif,image/png"
    :before-upload="onBeforeUpload"
    multiple
    :limit="3"
    :on-exceed="handleExceed">
    <el-button size="small" type="primary">点击上传</el-button>
    <div slot="tip" class="el-upload__tip">请上传图片格式文件</div>
</el-upload>
```
```javascript
onBeforeUpload(file)
{
    const isIMAGE = file.type === 'image/jpeg'||'image/gif'||'image/png';
    const isLt1M = file.size / 1024 / 1024 < 1;

    if (!isIMAGE) {
    this.$message.error('上传文件只能是图片格式!');
    }
    if (!isLt1M) {
    this.$message.error('上传文件大小不能超过 1MB!');
    }
    return isIMAGE && isLt1M;
}
```
这里在限制文件格式的同时，也做了文件大小的校验。

## 上传过程中的各种钩子
这里直接搬运官网的说明了，如图：

![](https://api2.mubu.com/v3/document_image/8d3bf50a-0348-40d6-abce-df7606f40d9b-3807603.jpg)

## 显示已上传文件列表
这个功能可以说很强大了，可以很清晰的显示已上传的文件列表，并且可以方便的删除，以便上传新的文件。
代码：
```html
<el-upload 
    :action="uploadActionUrl"
    accept="image/jpeg,image/gif,image/png"
    multiple
    :limit="3"
    :on-exceed="handleExceed"    
    :on-error="uploadError"
    :on-success="uploadSuccess"
    :on-remove="onRemoveTxt"
    :before-upload="onBeforeUpload"
    :file-list="files">
    <el-button size="small" type="primary">点击上传</el-button>
    <div slot="tip" class="el-upload__tip">请上传图片格式文件</div>
</el-upload>
```
实现方法就是:file-list="files"这个属性的添加，其中files是绑定的数组对象，初始为空。

## 上传时提交数据
上传文件同时需要提交数据给后台接口，这时就要用到:data属性。
```html
<el-upload 
    :action="uploadActionUrl"
    accept="image/jpeg,image/gif,image/png"
    :data="uploadData"
    multiple
    :limit="3"
    :on-exceed="handleExceed"    
    :on-error="uploadError"
    :on-success="uploadSuccess"
    :on-remove="onRemoveTxt"
    :before-upload="onBeforeUpload"
    :file-list="files">
    <el-button size="small" type="primary">点击上传</el-button>
    <div slot="tip" class="el-upload__tip">请上传图片格式文件</div>
</el-upload>
<!--
uploadData: {
    dataType: "0",
    oldFilePath:""
}
-->
```

## 选取和上传分开处理
有时我们需要把选取和上传分开处理，比如上传图片，先选取文件提交到前端，图片处理后再把base64内容提交到后台。
代码如下：
```html
<el-upload
  action=""
  accept="image/jpeg,image/png"
  :on-change="onUploadChange"
  :auto-upload="false"
  :show-file-list="false">
    <el-button slot="trigger" size="small" type="primary">选取</el-button>
    <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传</el-button>
    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不能超过1m</div>
</el-upload>
```
```javascript
  submitUpload()
  {
    console.log("submit")
  },
  onUploadChange(file)
  {
    const isIMAGE = (file.raw.type === 'image/jpeg' || file.raw.type === 'image/png');
    const isLt1M = file.size / 1024 / 1024 < 1;

    if (!isIMAGE) {
      this.$message.error('只能上传jpg/png图片!');
      return false;
    }
    if (!isLt1M) {
      this.$message.error('上传文件大小不能超过 1MB!');
      return false;
    }
    var reader = new FileReader();
    reader.readAsDataURL(file.raw);
    reader.onload = function(e){
        console.log(this.result)//图片的base64数据
    }
  }
```

## 参考链接
[Element el-upload上传组件详解](https://segmentfault.com/a/1190000013796215)