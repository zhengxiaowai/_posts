---
title: CSS设计指南读书笔记（四）
date: 2016-04-16 23:20:35
tags: CSS
---

# CSS 设计指南读书笔记（四）

多栏布局的三种基本实现方案：固定宽度、流动、弹性。

-   固定宽度不会随着页面的缩放而变化，一般选择固定宽度为 960 px，能被多种整数整除，实现多栏布局。
-   流动布局会随着浏览器的窗口大小变化而变化，这种布局能更好的适应大屏幕，但是文明本行的长度和页面的元素之间的位置关系可能会发生变化。
-   响应式设计利用媒体查询，为提供不同的 CSS 成为可能，使得不同的屏幕可以使用固定布局，正在替代流动布局。
-   弹性布局在浏览器大小发生变化时候，所有的元素和布局都会缩放，这种技术实现难度大。

**布局高度**一般要保持 auto ，这样在垂直方向上添加元素时候会自动向下拓展，如果设置的高度，那么元素可能会被剪掉，或者跑出元素外面去。

**布局宽度**需要精确的控制，在浏览器宽度合理变化时候提供合理的调整。必须要给定栏宽，其中的元素不需要给定宽度，使用默认行为填充满整个父元素的宽度。

## 三栏-固定布局

三栏布局中，需要计算出三栏的宽度等于父元素的宽度。

``` html
<div id="wrapper">
    <nav>
        <!-- 无序列表 -->
    </nav>
    <article>
        <!-- 这里是一些文本元素 -->
    </article>
    <aside>
        <!-- 文本 -->
    </aside>
</div>
```

三栏的元素分别是 nav、article、aside，假设 #wrapper 的宽度是 960px，那么这三个元素的宽度也要是 960px。

``` css
#wrapper {width:960px; margin:0 auto; border:1px solid;}
nav {
    width:150px;
    float:left;
    background:#dcd9c0;
}
nav li {
    list-style-type:none;
}
article {
    width:600px;
    float:left;
    background:#ffed53;
}
aside {
    width:210px;
    float:left;
    background:#3f7ccf;
    width:210px;
    float:left;
    background:#3f7ccf;
}
```

给这三个固定宽度元素添加内外边距时，由于被指定宽度的元素再添加内外边距时候元素会被扩大，会出现元素错位的情况，有三种方式可以解决。

-   计算元素宽度时候就把内外边距也考虑上，这样太麻烦，不推荐。
-   把这三个元素用 div 包起来，由于这个 div 是没有被设置宽度的，所以添加内外边距时大小不变
-   给三个元素使用 box-sizing:border-box 的属性，就不会导致给设定宽度的元素添加内外边距时，导致元素扩大，这个方法好。

## 三栏-中栏流动布局

目前而言 CSS 中的 table 属性是最简单、最容易实现的，但是在低于 IE7 的浏览器不被支持，也没有任何的替代方法。

CSS可以把一个 HTML元素的 display 属性设定为 table 、 table-row 和 table-cell 。

而通过 CSS把布局中的栏设定为 table-cell 有三个好处。

-   单元格（table-cell）不需要浮动就可以并排显示，而且直接为它们应用内边距也不会破坏布局。


-   默认情况下，一行中的所有单元格高度相同，因而也不需要人造的等高栏效果了。


-   任何没有明确设定宽度的栏都是流动的。

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style type="text/css">
        #main{
            width: 960px;
            margin: auto auto;
        }
      nav {
        display: table-cell;
        width: 100px;
        background-color: red;
        padding: 20px 20px;
      } 
      article {
        display: table-cell;
        width: 600px;
        background-color: blue;
        padding: 20px 20px;
      } 
      aside {
        display: table-cell;
        width: 260px;
        background-color: yellow;
        padding: 20px 20px;
      }
    </style>
</head>
    <body>
        <div id='main'>
            <nav>
                <!-- 一些东西 -->
            </nav>
            <article>
                <!-- 一些东西 -->
            </article>
            <aside>
                <!-- 一些东西 -->
            </aside>
        <div>
    </body>
</html>
```
