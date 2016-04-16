---
title: CSS设计指南读书笔记（一）
date: 2016-04-16 23:17:23
tags: CSS
---

# CSS设计指南读书笔记（一）

---

## 剖析CSS规则

给文档添加样式的方法有三种：**行内样式**、**嵌入样式**、**链接样式**。

1. **行内样式**
    
    直接写在属性里：
    ```html
    <p style="font-size: 14px; color: red;">我是一个内容</p>    
    ```
    行内样式只能影响所在的标签，而且还会覆盖嵌入样式和链接样式。

2. **嵌入样式**
    
    写在 `style` 标签中：
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title></title>
        <style type="text/css">
            h1 {font-size: 15px;}
            p {color: red;}
        </style>
    </head>
    <body>
        <h1>我是h1标签</h1>
        <p>我是p标签</p>
    </body>
    </html>
    ```
    嵌入样式只适用于当前页面，会覆盖链接样式，同时会被行内样式覆盖。

3. **链接样式**

    写在 `link` 标签中：
    ```html
    <link rel="stylesheet" type="text/css" href="css/style.css">
    ```
    链接样式适用于全部网站，前提是要把他引入。会被行内样式和嵌入样式覆盖，同时会被下一个链接样式覆盖。

### CSS命名规则

CSS的规则由**选择符**和**声明符**两部分组成。
```css
p {color: red;}
```

- `p` 是选择符，选择了`p`标签
- `{color: red;}` 是声明符

选择符可以有：
- 上下文选择符
- ID和类选择符
- 属性选择符

声明符由三部分组成：

1. `{ }` 花括号包围
2. `color` 标签的一个属性
3. `red` 属性的一个值

有三种使用方法：

- 一条规则多个声明
```css
p {
    color: red;
    font-size: 14px;
    font-weight: bold;
}
```

- 多个属性选择符共用某些属性
```css
h1 h2 h3 {
    color: red;
    font-size: 14px;
    font-weight: bold;
}
```

- 对一个属性设置多次
```css
h1 h2 h3 {
    color: red;
    font-size: 14px;
    font-weight: bold;
}
h3 {font-style: italic;}
```

### 上下文选择符

格式：标签1 标签2 {声明}

多个标签用空格分隔，而不是逗号分隔。

假如有下面设个代码片段：

```css
<body>
    <article>
        <h1>Contextual selectors are <em>very</em> selective</h1>
        <p>This example shows how to target a <em>specific</em> tag.</p>
    </article>
    <aside>
        <p>Contextual selectors are <em>very</em> useful!</p>
    </aside>
    <footer>
    &copy;2015
    </footer>
</body>
```

要想把 `article` 下所有的 `em` 变成红色:

```css
article em {
    color: red;
}
```
只要是在 `article` 下的所有 `em` 标签都会有影响，无论在什么地方。但是 `aside` 下的 `em` 标签没有影响。


#### 子选择器

格式： 标签1 > 标签2 > ... {声明}

```css
article > h1 > em {
    color: blue;
}
```
每级的标签必须有父子关系，就像 `article` 是 `h1` 的父元素，`h1是` 的 `em` 父元素。

#### 紧邻同胞选择器

格式：标签1 + 标签2 + ... {声明}

```css
h1 + p {
    color: blue;
}
```
只会影响 `article` 中的 `h1` 和 `p`, 因为他们是同胞元素而且 `p` 紧跟着 `h1`。

#### 一般同胞选择器

格式： 标签1 ~ 标签2 ~ ... {声明}

```css
article ~ footer{
    text-align: center;
}
```
`article` 和 `footer` 同胞元素就可以。

#### 通用选择符

`*` 代表所有元素

```css
* {
    padding: 0px;
    margin: 0px;
}
```
取消所有元素的内外边距。

```css
article * em {
    color:red;
}
```
只要 `em` 是祖父元素是 `article` 就会被设置成红色，无论父元素是什么。

### ID和类选择器

对于每个标签都可为它设置一个ID或者多个类。

#### 类属性

**格式： .类名**

```html
<h1 class="specialtext">
    This is a heading with the <span>same class</span> as the second paragraph.
</h1>

<p>
    This tag has no class.
</p>

<p class="specialtext featured"> 
    When a tag has a class attribute, you can target it <span>regardless</span> of its position in the hierarchy.
</p>
```

- 类选择符

```css
.specialtext {
    color: red;
}
```

选择 `class="specialtext"` 的标签设置前景色为红色。

- 标签带类选择符

```css
p.specialtext {
    color: blue;
}
```

选择所有类名为 `specialtext` 的p标签。

- 多类选择符

```css
.specialtext.featured {
    color: balck;
}
```
选择 类名同时为 `specialtext` 和 `featured` 的标签。

#### ID选择器

ID作为一个独一无二的存在，不能重复，使用方法了类选择器一样。例外，ID选择器可以作为页面内的跳转标记。

```html
<ul>
    <li><a href="#paragraph1">第一段</a></li>
    <li><a href="#paragraph2>第二段</a></li>
    <li><a href="#paragraph3>第三段</a></li>
</ul>
```
点击可以跳转到id为 `paragraph1` 的段落，`#`默认回到顶部。

#### 何时使用类选择器和ID选择器

- 给页面独一无二的元素使用ID选择器，比如一个主布局 `<div id="main"></div>`
- 拥有相同类型的元素使用类选择器，比如操作一个显示热门商品的列表。

### 属性选择器

**格式： 标签名[属性名]**

```html
<img src="images/yellow_flower.jpg" title="yellow flower" alt="yellow flower" />
```

可以选择拥有 `title` 属性的元素

```css
img[title] {border:4px solid green;}
```

也可以选择 `title` 的值为 `yellow flower` 的元素

```css
img[title="yellow flower"] {border:4px solid green;}
```

### 伪类

以下所有伪类只是常用的，更多伪类信息参考[点击这里] (http://www.stylinwithcss.com)

伪类分成两种

1. UI 伪类，在 HTML 元素处于某个状态下应用样式
2. 结构化伪类，在某种关系结构上应用样式

#### UI伪类

- 链接伪类

    1. :link：等待点击时候
    2. :visited：点击过之后
    3. :hover：鼠标悬停时候
    4. :active：正在被点击时候
    
    使用时候顺序以上顺序，要不然可能会因为特指度问题导致失败。

    可以应用到任何元素，不仅仅是 a 标签

- :focus 伪类

    当元素获取焦点时候，也就是鼠标点击的时候

    ```css
    input:focus{border: 1px solid blue;}
    ```

- :target 伪类

    当点击一个跳转到其他页面的链接，那个带有链接的元素就是 target,可以用伪伪类选中

    点击以下链接
    ```html
    <a href="#more_info">More Information</a>
    ```
    跳转到
    ```html
    <h2 id="more_info">This is the information you are looking for.</h2>
    ```
    使用伪类设置 target 的样式
    ```css
    #more_info:target {background:#eee;}
    ```

#### 结构化伪类

结构化伪类可以根据标签的结构来指定样式

- :first-child 和 last-chile

    把一组同胞元素的第一个和最后一个设置样式

    ```html
    <ol class="results">
        <li>My Fast Pony</li>
        <li>Steady Trotter</li>
        <li>Slow Ol' Nag</li>
    </ol>
    ```

    ```css
    ol.results li:last-child {color:red;}
    ```

    最后一个 li 被设置成红色

- :ntc-child(n)
    
    选择同胞元素中的第 n 个设置样式


#### 伪元素

- ::first-letter
    
    选择内容首字母，放大 3 倍
    
    ```css
    p::first-letter {font-size:300%;}
    ```

- :: first-line

    选择首段落与浏览器窗口大小有关，设置字体

    ```css
    p::first-line {font-variant:small-caps;}
    ```

- ::before 和 ::after

    在某个元素前后添加内容

    ```css
    p.age::before {content:"Age: ";}
    p.age::after {content:" years.";}
    ```