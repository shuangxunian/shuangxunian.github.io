---
title: js实现基于Base64的编码及解码
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---

## 需求
支付时为优化后端传的二维码图片为Base64，前端进行解码展示。

## 什么是Base64
所谓Base64，就是选出64个字符作为一个基本字符集（A-Z，a-z，0-9，+，/，再加上作为垫字的"="，实际是65个字符），其它所有符号都转换成这个字符集中的字符。

## 编码规则
- 第一步，将每三个字节作为一组，一共是24个二进制位。
- 第二步，将这24个二进制位分为四组，每个组有6个二进制位。
- 第三步，在每组前面加两个00，扩展成32个二进制位，即四个字节。
- 第四步，根据下表，得到扩展后的每个字节的对应符号，这就是Base64的编码值。

eg：
比如`dog`这个单词：

- d: ASCII值 100 ，二进制为 0110 0100
- o: ASCII值 111 ，二进制为 0110 1111
- g: ASCII值 103，二进制为 0110 0111

这24个二进制位 0110 0100 0110 1111 0110 0111 分为四组：
011001 000110 111101 100111

每组前面加两个00，即：
00011001 00000110 00111101 00100111

对应码值分别为：25、6、61、39
查找下表，可以知道得到编码后的字符串为：ZG9n

|   ASCII值  |  符号  |  ASCII值  |  符号  |  ASCII值  |  符号  |  ASCII值  |  符号  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|   0   |   A   |   16   |   Q   |   32   |   g  |   48    |   w   |
|   1   |   B   |   17   |   R   |   33   |   h  |   49    |   x   |
|   2   |   C   |   18   |   S   |   34   |   i  |   50    |   y   |
|   3   |   D   |   19   |   T   |   35   |   j  |   51    |   z   |
|   4   |   E   |   20   |   U   |   36   |   k  |   52    |   0   |
|   5   |   F   |   21   |   V   |   37   |   l  |   53    |   1   |
|   6   |   G   |   22   |   W   |   38   |   m  |   54    |   2   |
|   7   |   H   |   23   |   X   |   39   |   n  |   55    |   3   |
|   8   |   I   |   24   |   Y   |   40   |   o  |   56    |   4   |
|   9   |   J   |   25   |   Z   |   41   |   p  |   57    |   5   |
|   10   |   K   |   26   |   a   |   42   |   q  |   58    |  6    |
|   11   |   L   |   27   |   b   |   43   |   r  |   59    |   7   |
|   12   |   M   |   28   |   c   |   44   |   s  |   60    |   8   |
|   13   |   N   |   29   |   d   |   45   |   t  |   61    |   9   |
|   14   |   O   |   30   |   e   |   46   |   u  |   62    |   +   |
|   15   |   P   |   31   |   f   |   47   |   v  |   63    |   /   |

那如果字节数不足3呢？

2个字节的情况：将这2个字节的一共16个二进制位，按照上面的规则，转成三组(6,6,4)，最后一组除了前面加两个0以外，后面也要加两个0。这样得到一个三位的Base64编码，再在末尾补上一个"="号。

比如，“Ma"这个字符串是两个字节，可以转化成三组00010011、00010110、00000100以后，对应Base64值分别为T、W、E，再补上一个”="号，因此"Ma"的Base64编码就是TWE=。

1个字节的情况：将这1个字节的8个二进制位，按照上面的规则转成2组(6,2)，最后一组除了前面加二个0以外，后面再加4个0。这样得到一个二位的Base64编码，再在末尾补上两个"="号。

比如，“M"这个字母是一个字节，可以转化为二组00010011、00010000，对应的Base64值分别为T、Q，再补上二个”="号，因此"M"的Base64编码就是TQ==。

## 如何解码
了解了编码规则之后，解码就很容易理解了，就是编码过程的逆过程
- 第一步，将每4个字符为一组，查找上表，找到每个字符对应的ASCII值
- 第二步，将4个ASCII值写成二进制形式，并将每个二进制的前2个00去掉
- 第三步，将剩下的24位二进制位分成3份，即3个字节
- 第四步，查找ASCII值表（下表），找到每个字节对应的字符。

## window自带函数进行Base64编码解码
window有两个函数atob、btoa，但是这两个函数并不支持Unicode字符的编码，需结合encodeURIComponent、decodeURIComponent函数来使用。

### btoa
binary to ascii，用于将binary数据用ascii码表示。常用于编码字符串。
```javascript
var str = "This is a string";
var encoded_str = btoa(str);
console.log(encoded_str); // Outputs: "VGhpcyBpcyBhIHN0cmluZw=="
```

但是不能编码Unicode字符

### atob
ascii to binary，用于将ascii码解析成binary数据。用于解码Base64编码的字符串。
```javascript
var encoded_str = "VGhpcyBpcyBhIHN0cmluZw==";
var str= atob(encoded_str);
console.log(str); // Outputs: "This is a string"
```

### 如何让btoa支持Unicode字符编码
编码时，先用encodeURIComponent对字符串进行编码，再进行btoa进行Base64编码
解码时，先用atob对Base64编码的串进行解码，再用decodeURIComponent对字符串进行解码
```javascript
var str = "hello,中国";
var encoded_str = btoa(encodeURIComponent(str));
var decoded_str = decodeURIComponent(atob(encoded_str));
console.log(encoded_str); // Outputs: "aGVsbG8lMkMlRTQlQjglQUQlRTUlOUIlQkQ="
console.log(decoded_str); // Outputs: "hello,中国"
```

### IE9不支持atob、btoa
对于IE9不支持这两种编码解码方式，可以使用公共类库来兼容IE9

## 图片base64编码
前端在实现页面时，对于一些简单图片，通常会选择将图片内容直接内嵌在页面中，避免不必要的外部资源加载，增大页面加载时间，但是图片数据是二进制数据，该怎么嵌入呢？绝大多数现代浏览器都支持一种名为 Data URL 的特性，允许使用Base64对图片或其他文件的二进制数据进行编码，将其作为文本字符串嵌入网页中。
例如，我将本地一张图片进行Base64编码后（推荐一个在线图片转换工具：http://imgbase64.duoshitong.com/ ），其编码结果如下：
```javascript
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAJQAAACRCAYAAAAl8ZFnAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAJ0SURBVHhe7dYxalRRFIBhl+AKXEzqNNZuwzKNnVZTBRtJkyUELCysBRtxHdM9El5k4MkLpMhwYQb8rd5XfN053c+599U0TQtUBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERWrzQd3//rXMtzdP7n/+GM5wvk0HNV/vluXN6xf+XL0fznKezQa1XqbjmJ49fL0b7nDaZoMaXadnjx8/DHc4bbNBrdGMYlqt/6nRDqcJ6sjh8mKZ9vvhDqcJ6ojn7t8ISlApQQkqJShBpQQlqJSgBJUSlKBSghJUSlCCSglKUClBCSolKEGlBCWolKAElRKUoFKCElRKUIJKCUpQKUEJKiUoQaUEJaiUoASVEpSgUoISVEpQgkoJSlApQQkqJShBpQQlqJSgBJUSlKBSghJUSlCCSglKUClBCSolKEGlBCWolKAElRKUoFKCElRKUIJKCUpQKUEJKiUoQaUEJaiUoASVEpSgUoISVEpQgkoJSlApQQkqJShBpQQlqJSgBJUSlKBSghJUSlCCSglKUClBCSolKEGlBCWolKAElRKUoFKCElRqu0HtPgnqP9hsUA/fvw2Dmm9vhvOcZ7NBrQ7v3r6I6XB5sUz7/XCW82w6qNX85fPTMzdf78QU2HxQtARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQpARFSlCkBEVKUKQERUpQhKblL8jJczAt5hHJAAAAAElFTkSuQmCC
```

读者可将此Data URL 复制到浏览器地址栏试一下，发现可以解析出此图片。
用此方式可以将图片直接内嵌到网页中：
```javascript
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAJQAAACRCA..."/>
```

## 参考链接
[js实现基于Base64的编码及解码](https://blog.csdn.net/weixin_42420703/article/details/81384901)
[JavaScript中window对象的函数btoa和atob](https://blog.csdn.net/weixin_42420703/article/details/88422441)
