---
title: 面试官昨天问我对base64的理解，着实被问懵了
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---

## 一、为什么要使用 base64
我们知道一个字节可表示的范围是 0 ～ 255（十六进制：0x00 ～ 0xFF）， 其中 ASCII 值的范围为 0 ～ 127（十六进制：0x00 ～ 0x7F）；而超过 ASCII 范围的 128～255（十六进制：0x80 ～ 0xFF）之间的值是不可见字符。

> ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统。它主要用于显示现代英语，而其扩展版本延伸美国标准信息交换码则可以部分支持其他西欧语言，并等同于国际标准 ISO/IEC 646。

在 ASCII 码中 0 - 31和 127 是控制字符，共 33 个。以下是其中一部分控制字符：

![](https://api2.mubu.com/v3/document_image/2aa93139-84da-489a-93fc-15618db98f6d-3807603.jpg)

其余 95 个，即 32 - 126 是可打印字符，包括数字、大小写字母、常用符号等。

![](https://api2.mubu.com/v3/document_image/9c26b822-be89-4344-9ded-7055c0b2ef96-3807603.jpg)

当不可见字符在网络上传输时，比如说从 A 计算机传到 B 计算机，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，这是不利于传输的。

为了解决这个问题，我们可以先对数据进行编码，比如 base64 编码，变成可见字符，也就是 ASCII 码可表示的可见字符，从而确保数据可靠传输。Base64 的内容是有 0 ～ 9，a ～ z，A ～ Z，+，/ 组成，正好 64 个字符，这些字符是在 ASCII 可表示的范围内，属于 95 个可见字符的一部分。

## 二、什么是 base64
Base64 是一种基于 64 个可打印字符来表示二进制数据的表示方法。由于 2⁶ = 64 ，所以每 6 个比特为一个单元，对应某个可打印字符。3 个字节有 24 个比特，对应于 4 个 base64 单元，即 3 个字节可由 4 个可打印字符来表示。相应的转换过程如下图所示：

![](https://api2.mubu.com/v3/document_image/579ed710-c5f6-4157-9285-0383a4b14f50-3807603.jpg)

Base64 常用于在处理文本数据的场合，表示、传输、存储一些二进制数据，包括 MIME 的电子邮件及 XML 的一些复杂数据。在 MIME 格式的电子邮件中，base64 可以用来将二进制的字节序列数据编码成 ASCII 字符序列构成的文本。使用时，在传输编码方式中指定 base64。使用的字符包括大小写拉丁字母各 26 个、数字 10 个、加号 + 和斜杠 /，共 64 个字符，等号  = 用来作为后缀用途。Base64 相应的索引表如下：

![](https://api2.mubu.com/v3/document_image/3f7b4c21-52de-436e-aaab-a17a2d3f8c0a-3807603.jpg)

了解完上述的知识，我们以编码 Man 字符串为例，来直观的感受一下编码过程。Man 由 M、a 和 n 3 个字符组成，它们对应的 ASCII 码为 77、97 和 110。

![](https://api2.mubu.com/v3/document_image/bd3f3af6-86cb-42bb-9953-b28dc964caee-3807603.jpg)

接着我们以每 6 个比特为一个单元，进行 base64 编码操作，具体如下图所示：

![](https://api2.mubu.com/v3/document_image/f705c809-32b9-455b-815f-cc87ca46d949-3807603.jpg)

由图可知，Man （3字节）编码的结果为 TWFu（4字节），很明显经过 base64 编码后体积会增加 1/3。Man 这个字符串的长度刚好是 3，我们可以用 4 个 base64 单元来表示。但如果待编码的字符串长度不是 3 的整数倍时，应该如何处理呢？

如果要编码的字节数不能被 3 整除，最后会多出 1 个或 2 个字节，那么可以使用下面的方法进行处理：先使用 0 字节值在末尾补足，使其能够被 3 整除，然后再进行 base64 的编码。

以编码字符 A 为例，其所占的字节数为 1，不能被 3 整除，需要补 2 个字节，具体如下图所示：

![](https://api2.mubu.com/v3/document_image/c6978d27-a4ef-4ae8-8491-546056abe4d1-3807603.jpg)

由上图可知，字符 A 经过 base64 编码后的结果是 QQ==，该结果后面的两个 = 代表补足的字节数。而最后个 1 个 base64 字节块有 4 位是 0 值。

接着我们来看另一个示例，假设需编码的字符串为 BC，其所占字节数为 2，不能被 3 整除，需要补 1 个字节，具体如下图所示：

![](https://api2.mubu.com/v3/document_image/4a7c635e-768e-4501-a8df-673332dfe75a-3807603.jpg)

由上图可知，字符串 BC 经过 base64 编码后的结果是 QkM=，该结果后面的 1 个 = 代表补足的字节数。而最后个 1 个 base64 字节块有 2 位是 0 值。

## 三、base64 编码的应用
### 3.1 显示 base64 编码的图片
在编写 HTML 网页时，对于一些简单图片，通常会选择将图片内容直接内嵌在网页中，从而减少不必要的网络请求，但是图片数据是二进制数据，该怎么嵌入呢？绝大多数现代浏览器都支持一种名为 Data URLs 的特性，允许使用 base64 对图片或其他文件的二进制数据进行编码，将其作为文本字符串嵌入网页中。

Data URLs 由四个部分组成：前缀（data:）、指示数据类型的 MIME 类型、如果非文本则为可选的 base64 标记、数据本身：
```
data:[<mediatype>][;base64],<data>
```

mediatype 是个 MIME 类型的字符串，例如 "image/jpeg" 表示 JPEG 图像文件。如果被省略，则默认值为 text/plain;charset=US-ASCII。如果数据是文本类型，你可以直接将文本嵌入（根据文档类型，使用合适的实体字符或转义字符）。如果是二进制数据，你可以将数据进行 base64 编码之后再进行嵌入。比如嵌入一张图片：
```html
<img alt="logo" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...">
```

> MIME（Multipurpose Internet Mail Extensions）多用途互联网邮件扩展类型，是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。
> 常见的 MIME 类型有：超文本标记语言文本 .html text/html、PNG图像 .png image/png、普通文本 .txt text/plain 等。

但需要注意的是：如果图片较大，图片的色彩层次比较丰富，则不适合使用这种方式，因为该图片经过 base64 编码后的字符串非常大，会明显增大 HTML 页面的大小，从而影响加载速度。 除此之外，利用 HTML FileReader API，我们也可以方便的实现图片本地预览功能，具体代码如下：

```html
<input type="file" accept="image/*" onchange="loadFile(event)">
<img id="output"/>
<script>
  const loadFile = function(event) {
    const reader = new FileReader();
    reader.onload = function(){
      const output = document.querySelector('#output');
      output.src = reader.result;
    };
    reader.readAsDataURL(event.target.files[0]);
  };
</script>
```

在完成本地图片预览之后，可以直接把图片对应的 Data URLs 数据提交到服务器。针对这种情形，服务端需要做一些相关处理，才能正常保存上传的图片，这里以 Express 为例，具体处理代码如下：

```javascript
const app = require('express')();
app.post('/upload', function(req, res){
  let imgData = req.body.imgData; // 获取POST请求中的base64图片数据
  let base64Data = imgData.replace(/^data:image\/\w+;base64,/, "");
  let dataBuffer = Buffer.from(base64Data, 'base64');
  fs.writeFile("image.png", dataBuffer, function(err) {
    if(err){
       res.send(err);
    }else{
       res.send("图片上传成功！");
    }
    });
});
```

### 3.2 浏览器端图片压缩
在一些场合中，我们希望在上传本地图片时，先对图片进行一定的压缩，然后再提交到服务器，从而减少传输的数据量。在前端要实现图片压缩，我们可以利用 Canvas 对象提供的 toDataURL() 方法，该方法接收 type 和 encoderOptions 两个可选参数。

其中 type 表示图片格式，默认为 image/png。而 encoderOptions 用于表示图片的质量，在指定图片格式为 image/jpeg 或 image/webp 的情况下，可以从 0 到 1 的区间内选择图片的质量。如果超出取值范围，将会使用默认值 0.92，其他参数会被忽略。

下面我们来看一下具体如何实现图片压缩：
```javascript
// compress.js
const MAX_WIDTH = 800; // 图片最大宽度

function compress(base64, quality, mimeType) {
  let canvas = document.createElement("canvas");
  let img = document.createElement("img");
  img.crossOrigin = "anonymous";
  return new Promise((resolve, reject) => {
    img.src = base64;
    img.onload = () => {
      let targetWidth, targetHeight;
      if (img.width > MAX_WIDTH) {
        targetWidth = MAX_WIDTH;
        targetHeight = (img.height * MAX_WIDTH) / img.width;
      } else {
        targetWidth = img.width;
        targetHeight = img.height;
      }
      canvas.width = targetWidth;
      canvas.height = targetHeight;
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, targetWidth, targetHeight);
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      let imageData = canvas.toDataURL(mimeType, quality / 100);
      resolve(imageData);
    };
  });
}
```

对于返回的 Data URL 格式的图片数据，为了进一步减少传输的数据量，我们可以把它转换为 Blob 对象：
```javascript
function dataUrlToBlob(base64, mimeType) {
  let bytes = window.atob(base64.split(",")[1]);
  let ab = new ArrayBuffer(bytes.length);
  let ia = new Uint8Array(ab);
  for (let i = 0; i < bytes.length; i++) {
    ia[i] = bytes.charCodeAt(i);
  }
  return new Blob([ab], { type: mimeType });
}
```

在转换完成后，我们就可以压缩后的图片对应的 Blob 对象封装在 FormData 对象中，然后再通过 AJAX 提交到服务器上：
```javascript
function uploadFile(url, blob) {
  let formData = new FormData();
  let request = new XMLHttpRequest();
  formData.append("image", blob);
  request.open("POST", url, true);
  request.send(formData);
}
```

其实 Canvas 对象除了提供 toDataURL() 方法之外，它还提供了一个 toBlob() 方法，该方法的语法如下：
```javascript
canvas.toBlob(callback, mimeType, qualityArgument)
```

和 toDataURL() 方法相比，toBlob() 方法是异步的，因此多了个 callback 参数，这个 callback 回调方法默认的第一个参数就是转换好的 blob 文件信息。

介绍完上述的内容，我们来看一下本地图片压缩完整的示例：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>本地图片压缩</title>
  </head>
  <body>
    <input type="file" accept="image/*" onchange="loadFile(event)" />
    <script src="./compress.js"></script>
    <script>
      const loadFile = function (event) {
        const reader = new FileReader();
        reader.onload = async function () {
          let compressedDataURL = await compress(
            reader.result,
            90,
            "image/jpeg"
          );
          let compressedImageBlob = dataUrlToBlob(compressedDataURL);
          uploadFile("https://httpbin.org/post", compressedImageBlob);
        };
        reader.readAsDataURL(event.target.files[0]);
      };
    </script>
  </body>
</html>
```

## 四、如何进行 base64 编码和解码
### 4.1 使用 btoa 与 atob 函数
在 JavaScript 中，有两个函数被分别用来处理解码和编码 base64 字符串：
- btoa()：从字符串创建一个 base64 编码的 ASCII 字符串，其中字符串中的每个字符都被视为一个二进制数据字节。
- atob()：该函数能够解码通过 base64 编码的字符串数据。

btoa 使用示例：
```javascript
const name = 'Semlinker';
const encodedName = btoa(name);
console.log(encodedName); // U2VtbGlua2Vy
```

atob 使用示例：
```javascript
const encodedName = 'U2VtbGlua2Vy';
const name = atob(encodedName);
console.log(name); // Semlinker
```

介绍完 btoa 和 atob 这两个函数，我们再来看一下它们的兼容性：

![](https://api2.mubu.com/v3/document_image/87fcc98e-7981-4b32-8263-5bf6c1e0a45e-3807603.jpg)

由上图可知，除了 IE6-9 和 Opera 10.1 这些版本的浏览器之外，主流的浏览器都支持 btoa 和 atob 这两个函数。

### 4.2 使用第三方库
对于不支持 btoa 和 atob 这两个函数的浏览器来说，我们可以使用第三方库，比如 js-base64 这个库，来实现 base64 的编码和解码。

具体的使用示例如下：
```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Base64 编码与解码示例</title>
    <script src="https://cdn.jsdelivr.net/npm/js-base64@3.6.0/base64.min.js"></script>
  </head>
  <body>
    <h3>Base64 编码与解码示例</h3>
    <script>
      let name = Base64.encode("阿宝哥");
      console.log(name);
      console.log(Base64.decode(name));
    </script>
  </body>
</html>
```

在前端进行二进制处理的场景中，你可能会遇到 Data URL 转换成 Blob/File 对象的情形，接下来阿宝哥将汇总一下常用的转换函数。

## 五、常用转换函数
### 5.1 Data URL 转 Blob 对象
```javascript
function dataUrlToBlob(dataurl, mimeType) {
  let bytes = window.atob(dataurl.split(",")[1]);
  let ab = new ArrayBuffer(bytes.length);
  let ia = new Uint8Array(ab);
  for (let i = 0; i < bytes.length; i++) {
    ia[i] = bytes.charCodeAt(i);
  }
  return new Blob([ab], { type: mimeType });
}

// 使用示例
let blob = dataUrlToBlob('data:text/plain;base64,aGVsbG8gd29ybGQ=','hello.txt');
console.log(blob);
```

### 5.2 Data URL 转 File 对象
```javascript
function dataUrlToFile(dataurl, filename) {
  let arr = dataurl.split(","),
  mime = arr[0].match(/:(.*?);/)[1],
  bstr = atob(arr[1]),
  n = bstr.length,
  u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([u8arr], filename, { type: mime });
}

// 使用示例
let file = dataUrlToFile('data:text/plain;base64,aGVsbG8gd29ybGQ=','hello.txt');
console.log(file);
```

### 5.3 URL 转 File 对象
```javascript
function urlToFile(url, filename, mimeType) {
  return fetch(url).then((res) => {
    return res.arrayBuffer();
  }).then((buffer) =>{
    return new File([buffer], filename, { type: mimeType });
  });
}

// 使用示例
urlToFile('data:text/plain;base64,aGVsbG8gd29ybGQ=', 'hello.txt','text/plain')
  .then(function(file){ console.log(file);});
```

## 六、总结
Base64 是一种数据编码方式，目的是为了保障数据的安全传输。但标准的 base64 编码无需额外的信息，即可以进行解码，是完全可逆的。因此在涉及传输私密数据时，并不能直接使用 base64 编码，而是要使用专门的对称或非对称加密算法。

## 七、参考链接
[面试官昨天问我对base64的理解，着实被问懵了](https://mp.weixin.qq.com/s/biiijdxv9M1Sc31k_HLYdw)
