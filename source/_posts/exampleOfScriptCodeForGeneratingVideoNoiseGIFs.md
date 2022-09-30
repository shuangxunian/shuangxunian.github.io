---
title: 客户端网页/屏幕生成视频噪音动图的脚本代码例子
excerpt: 一个小demo
date: 2022-09-30
categories:
- 技术文章
tags:
- html
- js
---

## 客户端网页/屏幕生成视频噪音动图的脚本代码例子
看起来比较好的：
```html
<html>

<head>
</head>

<body>
    <canvas id=myCanvas0 width=1600 height=320></canvas>
</body>

<script>
    var li_w = myCanvas0.width;
    var li_h = myCanvas0.height;
    var lg_ocanvas = document.createElement("canvas");
    lg_ocanvas.width = li_w << 1;
    lg_ocanvas.height = li_h << 1;

    var lc_Ocontext = lg_ocanvas.getContext("2d", {
        alpha: false
    });
    var canvas_data = lc_Ocontext.createImageData(lg_ocanvas.width, lg_ocanvas.height);
    var vbuff = new Uint8Array(canvas_data.data.buffer);

    // render noise once, to the offscreen-canvas
    whitenoise(lc_Ocontext);

    // main loop draw the offscreen canvas to random offsets
    var lc_context = myCanvas0.getContext("2d", {
        alpha: false
    });
    (function loop() {
        var li_x = (li_w * Math.random()) | 0;
        var li_y = (li_h * Math.random()) | 0;

        lc_context.drawImage(lg_ocanvas, -li_x, -li_y);
        requestAnimationFrame(loop)
    })()

    function whitenoise(lc_context) {
        var li_len = vbuff.length - 1;
        while (li_len--) vbuff[li_len] = Math.random() < 0.5 ? 0 : -1 >> 0;
        lc_context.putImageData(canvas_data, 0, 0);
    }
</script>

</html>
```

Uint8Array 如果改成 Uint32Array 会更酷， 但是在 (linux) GOOGLE CHROME BROWSER 上会失效或者很难看出效果，虽然在 FIREFOX 上很正常。 因此，俺用 Uint8Array 而不是 Uint32Array。

比较简短的：
```html
<html>

<head>
</head>

<body>
    <canvas id=myCanvas0 width=1200 height=600></canvas>
</body>

<script>
    var lc_context0 = myCanvas0.getContext("2d", {
        alpha: false
    });
    var canvas_data = lc_context0.createImageData(myCanvas0.width, myCanvas0.height);
    var vbuffer = new Uint8Array(canvas_data.data.buffer);

    (function loop() {
        noise(lc_context0);
        requestAnimationFrame(loop)
    })()

    function noise(lc_context0) {
        var li_len = vbuffer.length - 1;
        while (li_len--) vbuffer[li_len] = Math.random() < 0.5 ? 0 : -1 >> 0;
        lc_context0.putImageData(canvas_data, 0, 0);
    }
</script>

</html>
```

Uint8Array 如果改成 Uint32Array 会更酷， 但是在 (linux) GOOGLE CHROME BROWSER 上会失效或者很难看出效果，虽然在 FIREFOX 上很正常。 因此，俺用 Uint8Array 而不是 Uint32Array。

## 其他
这个有滚屏效果， 动感十足。
```html
<html>

<head>
</head>

<body>
    <canvas id="myCanvas0" width="800" height="600"></canvas>

</body>

<script>
    var canvas = null;
    var context = null;
    var time = 0;
    var intervalId = 0;

    var makeNoise = function() {
        var imgd = context.createImageData(canvas.width, canvas.height);
        var pix = imgd.data;

        for (var i = 0, n = pix.length; i < n; i += 4) {
            var c = 7 + Math.sin(i / 50000 + time / 7); // A sine wave of the form sin(ax + bt)
            pix[i] = pix[i + 1] = pix[i + 2] = 40 * Math.random() * c; // Set a random gray
            pix[i + 3] = 255; // 100% opaque
        }

        context.putImageData(imgd, 0, 0);
        time = (time + 1) % canvas.height;
    }

    var setup = function() {
        canvas = document.getElementById("myCanvas0");
        context = canvas.getContext("2d");
    }

    setup();
    intervalId = setInterval(makeNoise, 50);
</script>

</html>
```

## 参考链接
[计算机随机生成一个数是不是真的是随机的？](https://www.zhihu.com/question/285909806/answer/1578054401)