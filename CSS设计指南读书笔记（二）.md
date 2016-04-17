---
title: CSS设计指南读书笔记（二）
date: 2016-04-16 23:20:14
tags: CSS
---

## 盒子模型

一个元素或者也可以叫一个盒子，是由四部分组成

- content
- padding
- border
- margin

它们由内而外分别是 **content->panding->border->margin** 

### 盒子的边框

盒子的边框位于 padding 和 margin 之间的分界线叫做 border。

border 有三个相关的属性

- border-width 用于设置宽度。有 thin、medium、thick等文本值，也可以使用除了百分比和负值之外的数值，
-  border-style 用于设置样式。有 none、dotted、dashed、solid、double、groove、ridge、inset和outset等文本值。
- border-color 用于设置颜色。可以使用 RGB、HSL、十六进制、关键字。
- border-radius 

border-width 中的的3个文本值会因为浏览器的不同而不同，border-style 中只有 soild 是 CSS 明确规定的。

### 盒子的内边距

盒子内边距位于 content 和 border 之前的区域。

相关属性有

- padding
- padding-top
- padding-right
- padding-bottom
- padding-left

四个属性都是设置内边距的宽度，可以指定 px、em等。

其中 padding 为可以简写模式，也可以非简写模式

```css
//简写模式，设置上下为10px，左右为5px
p {padding: 10px 5px;}
```

```css
//非简写模式，分别设置上右下左内边距（即顺时针）为 1px、2px、3px、4px
p {padding: 1px 2px 3px 4px;}
```

### 盒子的外边距

盒子的外边距位于 border 之外。

属性和使用方法和 padding 一模一样 使用 margin 替代 padding 即可。

盒子的外边距在 block 元素中会有叠加发生，inline 元素则不会有。

例如有两个 p 标签

```html
<p id="p1">i am p1</p>
<p id="p2">i am p2</p>
```

使用 CSS 设置他们的 margin

```css
p#p1 {margin: 10px 0px;}
p#p2 {margin: 5px 0px;}
```

这时 p1 和 p2 之间的距离是较大的那个外边距就是 10px，如果相同，那么就是那个值。

对于 inline 元素则完全不同，就是两个左或者右边距相加。

## 盒子有多大

世界上有两种盒子，一种是设置了 width 和 没有设置 width 的。

对于没有设置 width 的盒子 宽度默认是父元素的宽度。
有设置 width 的盒子，这里设置 width 只是设置了内容的宽度。

对没有 width 的盒子，给他添加左右 padding 和 margin 会使得内容区域变窄，盒子大小不变。
对于有 width 的盒子，添加左右 padding 和 margin 内容区域不变，盒子变大。

## 浮动与删除

让元素层叠的关键就是 float 属性，可以让元素浮动在其他元素之上，使用 clear 可以设置不允许有浮动元素层叠。

### 浮动

float 属性目的是实现文本围绕图片，同时也能做分栏。

#### 文本围绕图片

```html
    <img src="st3.png" height="50" width="130">
    <p>
        CSS3 Multi-column Layout Module 规定了如何用CSS 定义多栏布局。但在本书写作
    时，只有Opera 和IE10 支持相应属性。因此在可以预见的未来，float 属性仍然是
    创建多栏布局的最佳途径。
    </p>
```

这时候布局是上面是 img，在它下面是 p

想让文本包围 img

```css
img {float:left;}
```

#### 分栏

再次设置一次 float 并设置一下 p 的 width 属性。同时你的一行足够容纳下这些元素。

使用 float 时候必须设置 width ，图片有自己本身的 width 所有不用设置也可以。

### 围住浮动元素的三种方法

浮动了元素之后， 该元素脱离文本流，它的父元素找不到他了，自然也无法包围他。

1. 设置父元素属性 ```overflow:hidden;```
2. 设置父元素浮动 ```float:left; width:100%;```
3. 添加非浮动的清除元素

对于第三种方法是最好的方法，实现方式有两种

```html
<section>
    <img src="images/rubber_duck.jpg">
    <p>It's fun to float.</p>
    <div class="clear_me"></div>
</section>
```

添加一个空的 div 并设置 clear 
```css
.clear_me {clear:left;}
```

还有一种方式是不添加这个没有显示功能的 div，可以给父元素添加一个类

```html
<section class="clearfix">
    <img src="images/rubber_duck.jpg">
    <p>It's fun to float.</p>
</section>
```
然后设置 **.clearfix**

```css
.clearfix:after {
    content:".";
    display:block;
    height:0;
    visibility:hidden;
    clear:both;
}
```

这是 css 设置了最小内容 **.** 并设置高度为 0 让其不可见， 再使用 clear 清楚两边的浮动元素，这样所有的浮动元素有会到这个 div 的下方去。

三种方法不能应付的场景：

不能再下来拉菜单的顶级元素使用 **overflow:hidden** ，因为下拉框的下拉内容是显示在父元素的区域外，而这个属性设置的是不显示超出父元素的内容，这就导致下拉框不显示。

不能对已经靠自动外边距居中的元素使用“浮动父元素”技术，否则它就不会再居中，而是根据浮动值浮动到左边或右边了。

有时候清除浮动元素时候并没有父元素来强制包围。可以把 **.clearfix** 类添加到某个具体的元素上，强制这个元素清楚周围的浮动。

## 定位

css 的定位使用的 position 这个属性，这个属性有四个值 static、relative、absolute、fixed，默认值为static。通过定位可以对元素重新定位。

### 静态定位

静态定位是默认的方式，就是按照文本流 block 元素换行， inline 元素不换行，从头到尾排列。

### 相对定位

相对定位的相对，是相对于原来的位置。之后可以使用 top 、left 设置新的位置。

```css
p#specialpara {position:relative; top:25px; left:30px;}
```

原则上使用 top 和 left 就可以了，因为他们支持负数。元素移动出来的同时，空出来的位置会继续保留。

### 绝对定位

绝对定位和相对定位不同的是，空出来的区域被回收了，也就是说移动出来的元素，脱离了文本流。

定位的位置和**定位上下文**有关，默认的定位上下文的 body 元素。只要把父元素的 position 属性设置为  relative 那么个这个父元素就成了新的上下文。

### 固定定位

元素脱离文本流，固定在相对于屏幕的某个位置，随着浏览器滚动，这是元素始终在那个位置。这是功能可以用来做固定的导航。

## 显示属性

所有的元素都有 display 属性，大多数的默认值不是 block 就是 inline，还有其他很多属性。

让两个 block 元素并排显示可以设置

```css
p {
    display: inline;
}
```

让一个元素隐藏可以使用

```css
p {
    display: none;
}
```
被隐藏的元素包括他所在位置都会消失

```css
p {
    visibility: hidden;
}
```
但是这种方式，元素被隐藏但是空出来的位置还在。

## 背景

一个元素的展现可以分成3成，由里到外是

1. 文本或者图片
2. 背景图片
3. 背景颜色

### CSS 背景属性

- background-color
- background-image
- background-repeat
- background-position
- background-size
- background-attachment
- background（简写属性）

### 背景颜色

```css
body {background-color: #caebff;}
```

### 背景图片

设置方式需要这样： **background-image:url(图片路径/图片文件名)**

默认图片是重复的位置也是默认在左上角，图片会重左上角重复铺开，直到填满整个元素。

### 背景重复

控制背景重复方式的background-repeat 属性有4 个值。

- repeat （默认）
- repeat-x （只在水平方向重复）
- repeat-y （只在垂直方向上重复）
- no-repeat （背景图片显示一次）

### 背景位置
用于控制背景位置的是 background-position 属性

有五个关键字

- top
- left
- bottom
- right 
- center

background-position 属性同时设定元素和图片的原点

让一张图片居中不重复的方法

```css
div {
    height:150px;
    width:250px;
    border:2px solid #aaa;
    margin:20px auto;
    background-image:url(images/turq_spiral_150.png);
    background-repeat:no-repeat;
    background-position:50% 50%;
}
```

通过把 background-position 设定为50% 50%，把background-repeat 设定为no-repeat，实现了图片在背景区域内居中的效果。

### 背景尺寸

background-size 是CSS3 规定的属性，但却得到了浏览器很好的支持。

- 50% 缩放图片，使其填充背景区的一半。
- 100px 50px：把图片调整到100 像素宽，50 像素高。
- cover：拉大图片，使其完全填满背景区；保持宽高比。
- contain：缩放图片，使其恰好适合背景区；保持宽高比。

### 背景粘附

background-attachment 属性控制滚动元素内的背景图片是否随元素滚动而移动。

- scroll （默认） 滚动
- fixed 不滚动

###  简写背景属性

```css
body {background:url(images/watermark.png) center #fff no-repeat contain fixed;}
```

第一个路径，第二个是位置，第三个是背景色，第四个是重复方式，第五和是背景尺寸，都五个是是否滚动。

### 多背景图片

```css
p {
    height:150px;
    width:348px;
    border:2px solid #aaa;
    margin:20px auto;
    font:24px/150px helvetica, arial, sansserif;
    text-align:center;
    background:
    url(images/turq_spiral.png) 30px -10px no-repeat,
    url(images/pink_spiral.png) 145px 0px no-repeat,
    url(images/gray_spiral.png) 140px -30px no-repeat, #ffbd75;
}
```

CSS 规则中先列出的图片在上层。
