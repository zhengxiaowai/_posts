---
title: CSS设计指南读书笔记（三）
date: 2016-04-16 23:20:21
tags: CSS
---

# CSS设计指南读书笔记（三）

## 字体

网页中字体的三个来源

- 用户机器安装的字体
- 第三方网站上的字体
- 储存在 web 服务器上字体

CSS 中有6个与字体有关的属性：font-family、font-size、font-style、font-weight、font-variant、font 

### 字体族

font-family 可以设置 字体族，也就是指定文本用什么字体，可以设置多个，排列在前面的优先级高

```css
body {font-family:"trebuchet ms", tahoma, sans-serif;}
```

带有空格的字体需要加上引号，前面的字体优先选择，如果找不到，选择下一个，一般最后一个是要设置一个通用字体。

-  serif ，也就是衬线字体，在每个字符笔画的末端会有一些装饰线；
-  sans-serif ，也就是无衬线字体，字符笔画的末端没有装饰线；
-  monospace ，也就是等宽字体，顾名思义，就是每个字符的宽度相等（也称代码体）；
-  cursive ，也就是草书体或手写体
-  fantasy ，不能归入其他类别的字体（一般都是奇形怪状的字体）

### 字体大小

每个 HTML 元素都设置默认字体大小，当在修改字体大小时候都是修改了默认值，同时也是可以继承的

```css
 h2 {font-size:18px;}
```
设置字体大小可以有绝对字体大小和相对字体大小

绝对字体大小不会随页面缩放不会继承父元素属性，一般使用 px 单位，也可以使用关键字 x-small 、medium 、x-large

相对字体大小会随着页面缩放会继续父元素的字体大小，再其基础上缩放。使用百分比、em、rem 作为单位。

rem 是相对根元素的的大小

### 字体样式

```css
h2 {font-style:italic;}
```

- normal 正常
- italic 斜体

### 字体粗细

font-weight 属性的两个值： bold 和 normal

### 字体变化

font-variant 属性除了 normal ，就只有一个值，即 small-caps 。这个值会导致所有
小写英文字母变成小型大写字母：

```css
h3 {font-variant:small-caps;}
```


## 文本属性

以下是几个最有用的 CSS文本属性：

-  text-indent
-  letter-spacing
-  word-spacing
-  text-decoration
-  text-align
-  line-height
-  text-transform
-  vertical-align

### 文本缩进

```css
 p {text-indent:3em;}
```

text-indent 属性设定行内盒子相对于包含元素的起点。默认情况下，这个起点就是包含元素的左上角。

正值向右，负值向左

text-indent 是可以被子元素继承的。但是继承的是最终的值。

>假设有一个 400 像素宽的 div，包含的文本缩进 5%，则缩进的距离是 20 像素（400 的 5%）。
在这个 div 中有一个 200 像素宽的段落。作为子元素，它继承父元素的 text-indent 值，所以
它包含的文本也缩进。但继承的缩进值是多少呢？不是 5%，而是 20 像素

### 字符间距

```css
p {letter-spacing:.2em;}
```
>letter-spacing 为正值时增大字符间距，为负值时缩小间距。无论设定字体大小时使用的是什么单位，设定字符间距一定要用相对单位，以便字间距能随字体大小同比例变化。

### 单词间距

```css
 p {word-spacing:.2em;}
```

### 文本装饰

```css
.retailprice {text-decoration:line-through;}
```

 值有：underline 、 overline 、 line-through 、 blink 、 none，其中 blink 不使用

### 文本对齐

```css
p {text-align:right;}
```

>text-align 属性只有 4 个值， left 、 right 、 center 和 justify ，控制着文本在水平方向对齐的方式。其中， center 值也可以用来在较大的元素中居中较小的固定宽度的元素或图片。

### 行高

```css
 p {line-height:1.5;}
```

值：任何数字值（不用指定单位）


### 文本转换

```css
 p {text-transform:capitalize;}
```

值： none 、 uppercase 、 lowercase 、 capitalize 。

### 垂直对齐

值：任意长度值以及 sub 、 super 、 top 、 middle 、 bottom 等。
```css
span {vertical-align:60%;} 。
```

>vertical-align 以基线为参照上下移动文本，但这个属性只影响行内元素。如果你想在垂直方向上对齐块级元素，必须把其 display 属性设定为 inline 。

 HTML 标签  sup 和 sub 有默认的上标和下标样式，但重新设定一下vertical-align 和 font-size 属性能得到更美观的效果。

### @font-face

```css
@font-face {
/*这就是将来在字体栈中引用的字体族的名字*/
font-family:'UbuntuTitlingBold';
src: url('UbuntuTitling-Bold-webfont.eot');
    src: url('UbuntuTitling-Bold-webfont.eot?#iefix')
        format('embedded-opentype'),
        url('UbuntuTitling-Bold-webfont.woff')
        format('woff'),
        url('UbuntuTitling-Bold-webfont.ttf')
        format('truetype'),
        url('UbuntuTitling-Bold-webfont.
        svg#UbuntuTitlingBold') format('svg');
font-weight: normal;
font-style: normal;
}
```

把以上代码添加到网页中之后，就可以使用 font-family 以常规方式引用该字体了。引用字体时要使用 @font-face 规则中 font-family 属性的值作为字体族的名字。