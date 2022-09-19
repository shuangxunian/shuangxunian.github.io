---
title: css怎么清除ul,li默认样式？
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

## 前言
在CSS中，可以使用list-style-type属性，设置值为none来清除ul li的默认样式。下面本篇文章就来给大家介绍一下CSS list-style-type属性，希望对大家有所帮助。

`list-style-type`属性用于设置列表项标记的类型。设置`list-style-type：none`就可将列表项 li 默认样式移除。

## 可能的值（CSS2）：

- none 无标记。
- disc 默认。标记是实心圆。
- circle 标记是空心圆。
- square 标记是实心方块。
- decimal 标记是数字。
- decimal-leading-zero 0开头的数字标记。(01, 02, 03, 等。)
- lower-roman 小写罗马数字(i, ii, iii, iv, v, 等。)
- upper-roman 大写罗马数字(I, II, III, IV, V, 等。)
- lower-alpha 小写英文字母The marker is lower-alpha (a, b, c, d, e, 等。)
- upper-alpha 大写英文字母The marker is upper-alpha (A, B, C, D, E, 等。)
- lower-greek 小写希腊字母(alpha, beta, gamma, 等。)
- lower-latin 小写拉丁字母(a, b, c, d, e, 等。)
- upper-latin 大写拉丁字母(A, B, C, D, E, 等。)
- hebrew 传统的希伯来编号方式
- armenian 传统的亚美尼亚编号方式
- georgian 传统的乔治亚编号方式(an, ban, gan, 等。)
- cjk-ideographic 简单的表意数字
- hiragana 标记是：a, i, u, e, o, ka, ki, 等。（日文片假名）
- katakana 标记是：A, I, U, E, O, KA, KI, 等。（日文片假名）
- hiragana-iroha 标记是：i, ro, ha, ni, ho, he, to, 等。（日文片假名）
- katakana-iroha 标记是：I, RO, HA, NI, HO, HE, TO, 等。（日文片假名）

## 浏览器支持

所有浏览器都支持 list-style-type 属性。

注释：任何的版本的 Internet Explorer （包括 IE8）都不支持属性值 "decimal-leading-zero"、"lower-greek"、"lower-latin"、"upper-latin"、"armenian"、"georgian" 或 "inherit"。

## 示例1：清除ul li默认样式
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<style type="text/css">
			ul.circle {
				list-style-type: circle
			}
			
			ul.square {
				list-style-type: square
			}
			
			ul.none {
				list-style-type: none
			}
		</style>
	</head>

	<body>
		<ul>
			<li>默认样式</li>
			<li>默认样式</li>
			<li>默认样式</li>
		</ul>
		<ul class="none">
			<li>去除默认样式</li>
			<li>去除默认样式</li>
			<li>去除默认样式</li>
		</ul>
	</body>
</html>
```

## 示例2：设置ul li样式
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<style type="text/css">
			ul.circle {
				list-style-type: circle
			}
			
			ul.square {
				list-style-type: square
			}
			
		</style>
	</head>

	<body>
		<ul>
			<li>默认样式</li>
			<li>默认样式</li>
			<li>默认样式</li>
		</ul>

		<ul class="circle">
			<li>空心圆样式</li>
			<li>空心圆样式</li>
			<li>空心圆样式</li>
		</ul>

		<ul class="square">
			<li>实心方块样式</li>
			<li>实心方块样式</li>
			<li>实心方块样式</li>
		</ul>

		
	</body>

</html>
```

## 参考链接
[css怎么清除ul,li默认样式？](https://www.html.cn/qa/css3/13926.html)