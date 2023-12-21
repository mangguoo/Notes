# CSS

## CSS选择器

### 选择器类型

- 基础选择器

	- 元素选择器 `a{}`

	- ID选择器 `#id{}`

	- class选择器 `.link{}`

	- 通用选择器 `*{}`

- 复合选择器

	- 后代选择器 `div h3{}`

	- 子元素选择器 `div > h3{}`

	- 兄弟选择器 `div + p{}`(选择紧跟在 <div> 元素之后的第一个 <p> 元素)

	- 相邻选择器 `p ~ ul{}`(选择p元素之后的每一个ul元素)

	- 并集选择器 `div,span{}`

- 属性选择器

	- `[target]`：选择所有带有target属性元素
	- `[target=_blank]`：选择所有使用target="_blank"的元素
	- `[title~=flower]`：选择标题属性包含单词"flower"的所有元素
	- `[lang|=en]`：选择 lang 属性等于 en，或者以 en 开头的所有元素
	- `a[src^="https"]`：选择每一个src属性的值以"https"开头的元素
	- `a[src$=".pdf"]`：选择每一个src属性的值以".pdf"结尾的元素
	- `a[src*="runoob"]`：选择每一个src属性的值包含子字符串"runoob"的元素

- 伪元素选择器

	- `::before`：元素的开始
	- `::after`：元素的结束
	- `::first-letter`：第一个字母
	- `::first-line`：第一行
	- `::section`：光标选中的部分

- 伪类选择器

	- 锚点伪类选择器 `:target`(选择器可用于当前活动的target元素的样式)

	  ```css
	  :target{
	      border: 2px solid #D4D4D4;
	      background-color: #e5eecc;
	  }
	  ```
	
	  ```html
	  <a href="#aa">baibai</a>
	  <div id="aa"></div>
	  ```

- 链接伪类选择器

	- `a:link`：未被访问的链接
	- `a:visited`：已被访问的链接
	- `a:hover`：鼠标经过的链接
	- `a:active`：鼠标选中的链接

- 表单伪类选择器

	- `input:focus`：选择具有焦点的输入元素
	- `input:checked`：选择每个选中的输入元素
	- `input:optional`：用于匹配可选的输入元素(匹配没有设置 "required" 属性的 input 元素)
	- `input:disabled`：选择每一个禁用的输入元素
	- `input:enabled`：选择每一个已启用的输入元素(即没有disabled属性的input)
	- `input:out-of-range`：匹配值在指定区间之外的input元素
	- `input:in-range`：匹配值在指定区间之内的input元素(只作用于能指定区间值的元素，例如 input 元素中的 min 和 max 属性)
	- `input:read-write`：用于匹配可读及可写的元素
	- `input:read-only`：用于匹配设置 "readonly"属性的元素
	- `input:invalid`：用于匹配输入值为非法的元素
	- `input:valid`：用于匹配输入值为合法的元素(只作用于能指定区间值的元素，例如 input 元素中的 min 和 max 属性，及正确的 email 字段, 合法的数字字段等)

- 结构伪类选择器

	- `p:first-of-type`：选择p元素，并且该p元素是其父级中的第一个p元素
	- `p:last-of-type`：选择p元素，并且该p元素是其父级中的最后一个p元素
	- `p:only-of-type`：选择p元素，并且该p元素是其父级中的唯一一个p元素
	- `p:nth-of-type(2)`：选择p元素，并且该p元素是其父级中的第二个p元素
	- `p:first-child`：选择p元素，且该p元素必须是其父级的第一个子元素
	- `p:last-child`：选择p元素，且该p元素必须是其父级的最后一个子元素
	- `p:only-child`：选择p元素，且该p元素必须是其父级的唯一一个子元素
	- `p:nth-child(2)`：选择p元素，且该p元素必须是其父级的第二个子元素

- 其它伪类选择器

  - `:not(p)`：选择每个并非p元素的元素

  - `:root`：选择文档的根元素

  - `p:empty`：选择每个没有任何子级的p元素（包括文本节点）

  - `:is :where`：CSS伪类函数以选择器列表作为参数，并选择该列表中任意一个选择器可以选择的元素

    ```css
    /* 选择标题、主体和页脚部分中的所有段落 */
    :where(header, main, footer) p:hover {
      color: red;
      cursor: pointer;
    }
    
    /* 以上内容等效于以下内容 */
    header p:hover,
    main p:hover,
    footer p:hover {
      color: red;
      cursor: pointer;
    }
    ```

### :is和:where的区别：

> 两者之间的区别在于，`:is()`会计入整个选择器的优先级（它采用其最具体参数的优先级），而`:where()`的优先级为0

```html
<article>
  <h2>:where()-styled links</h2>
  <section class="styling">
    <p>
      Here is my main content. This
    </p>
  </section>

  <aside class="styling">
    <p>
      Here is my aside content. This
    </p>
  </aside>

  <footer class="styling">
    <p>
      This is my footer, also containing
    </p>
  </footer>
</article>
```

```css
:is(section.styling, aside.styling, footer.styling) a {
  color: red;
}
```

这里，如果想要使用一个简单的选择器来覆盖article中所有p标签中文字的颜色：

```css
article a {
  color: blue;
}
```

这个红色的样式不起作用，因为`:is()`中的选择器会计入整体选择器的优先级，并且类选择器的优先级高于元素选择器，而如果使用:where()，由于他列表中的选择器优先级都为0，因此可以通过上述方法进行简单覆盖

```css
:where(section.styling, aside.styling, footer.styling) a {
  color: orange;
}
```

### 选择器权重

**规则一：**

- ID选择器       +100
- 类 属性 伪类    +10
- 元素 伪元素     +1
- 其它选择器      +0

**规则二：**相同权重，后写的生效

```css
#div1 {  // 不生效
    color: red;
}
#div1 {   // 生效
    color: blue;
}
```

**规则三：**import优先级最高

```css
#div1 {
    color: red!important;
}
```

## CSS非布局样式

### 垂直对齐方式 vertical-align

> 该属性定义行内元素的基线相对于该元素所在行的基线的垂直对齐。在单元格中，这个属性会设置单元格框中的单元格内容的对齐方式
>
> - baseline： 默认属性，元素放在父元素的基线上；
> - sub： 垂直对齐文本的下标；
> - super： 垂直对齐文本的上标；
> - top： 把元素的顶端与行中最高元素的顶端对齐；
> - middle： 把元素放置在父元素的中部；
> - bottom： 把元素的底部与整行的底部对齐；
> - text-top： 把元素的顶端与父元素字体的顶端对齐；
> - text-bottom： 把元素的底端与父元素字体的底端对齐；

```html
<span style="font-size: 30px; vertical-align: bottom;">
    AFTER
</span>

<span style="font-size: 30px; vertical-align: bottom;">
    CENTER
</span>

<span style="font-size: 10px; vertical-align: bottom;">
    AFTER
</span>
```

**图片3px缝隙问题:**

行内元素默认是基线对齐，而图片也是行内元素，因此图片与文字在同一行的时候，由于两者基线对齐，导致图片会与父元素底部有一定的缝隙，这就是图片3px缝隙问题

```html
<div style="border: 1px solid black;">
    <span style="font-size: 30px;">
        AFTER
    </span>
    <img src="./Snipaste_2022-04-29_13-04-54.png">
</div>
```

![image-20221222155651000](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222155651000.png)

```html
<div style="border: 1px solid black;">
    <span style="font-size: 30px;">
        AFTER
    </span>
    <img src="./Snipaste_2022-04-29_13-04-54.png" style="vertical-align: bottom;">
</div>
```

![image-20221222160036854](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222160036854.png)

### 背景 background
**背景颜色:**

```css
<-- 背景颜色 -->
background-color: red;
background-color: rgba(red, green, blue, alpha);
background-color: #FF0000FF;
background-color: hsla(hue, saturation, lightness, alpha);

<-- 渐变背景色 -->
background-image: linear-gradient(to right, red, green);
background-image: linear-gradient(45deg, red, green);
background-image: linear-gradient(135deg, red 0, green 50%, blue 100%);

<-- 多背景叠加 -->
div{
    height: 90px;
    background-image: linear-gradient(135deg, transparent 0, transparent 49.5%, green 49.5%, green 50.5%, transparent 50.5%, transparent 100%), linear-gradient(45deg, transparent 0, transparent 49.5%, red 49.5%, red 50.5%, transparent 50.5%, transparent 100%);
    background-size: 30px 30px;
    background-repeat: repeat;
}
```

background-size不只可以对背景图片生效，对渐变背景颜色同样生效，因此上例中实际上是指定了线性渐变的范围为30px的正方形。并且由于设置了重复背景，所以才有了下方栅栏状的背景

![image-20221222160545539](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222160545539.png)

**背景图片:**

- 背景图片：`background-image: url(./Snipaste_2022-04-29_13-04-54.png);`

- 重复背景图片：`background-repeat: repeat | no-repeat | repeat-x | repeat-y;`

- 背景图片相对位置：`background-position: 20px 20px;`
                            `background-position: center top;`

- 背景图片大小：`background-size: 20px 20px;`（也可以设置百分比）

  - `cover`: 此时会保持图像的纵横比并将图像缩放成将完全覆盖背景定位区域的最小大小

  - `contain`: 此时会保持图像的纵横比并将图像缩放成将适合背景定位区域的最大大小

- 相对位置的目标：`background-origin:padding-box;`
  - `padding-box`: 背景图像填充框的相对位置                     
  - `border-box`: 背景图像边界框的相对位置                 
  - `content-box`: 背景图像的相对位置的内容框
- 背景图像是否随着页面滚动：`background-attachment：scroll;`
  - `scroll`: 背景图片随着页面的滚动而滚动(默认)
  - `fixed`: 背景图片不会随着页面的滚动而滚动
  - `local`: 背景图片会随着元素内容的滚动而滚动

**背景图优化方案：雪碧图**

```css
#div1 {
    border: 1px solid black;
    width: 100px;
    height: 100px;
    background-color: red;
    background-image: url(./Snipaste_2022-04-29_13-04-54.png);
    background-size: 529px 558px;
    background-position: -100px -100px;
}
#div2 {
    border: 1px solid black;
    width: 50px;
    height: 50px;
    background-color: red;
    background-image: url(./Snipaste_2022-04-29_13-04-54.png);
    background-size: 264px 279px;
    background-position: -50px -50px;
}
```

```html
<div id="div1"></div>
<div id="div2"></div>
```

![image-20221222161715797](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222161715797.png)

**base64和性能优化:**

> 图片经过base64编码之后，可以直接放进html文档中，这样用户在请求文档时，就可以少一个http请求，从而节约性能

```css
div{
    width: 100px;
    height: 20px;
    border: 1px solid black;
    background-repeat: no-repeat;
    background-image: url(data:image/png;base64,iVBO......);
}
```

**雪碧图动画:**

```css
.i{
    width: 20px;
    height: 20px;
    background: url(./background.png) no-repeat;
    background-size: 20px 40px;
    transition: background-position .4s;
}
.i:hover{
    background-position: 0 -20px;
}
```

```html
<div class="i"></div>
```

![image-20221222162434819](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222162434819.png)

### 边框 border
**边框的属性:**

```css
<-- 设置边框类型、宽度、颜色 -->
border-style: dotted | dashed | solid | double;
border-width: 1px;
border-color: red;

<-- 分别设置边框类型、宽度、颜色 -->
border-bottom: 1px solid red;
border-top: 1px solid red;
border-left:1px solid red;
border-right: 1px solid red;
```

**边框背景图:**

> border-image的第二个参数是指定边框背景图片以多少长度进行分割，他的单位默认是px，因此不需要再写单位

```css
#div1{
    width: 200px;
    height: 100px;
    border: 30px solid transparent;
    border-image: url(./border.png) 30 repeat;
}
#div2{
    width: 200px;
    height: 100px;
    border: 30px solid transparent;
    border-image: url(./border.png) 30 round;
}
```

![image-20221222162801372](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222162801372.png)

**边框衔接:**

```html
<style>
    div {
        display: inline-block;
    }
    #div1{
        width: 50px;
        border: 30px solid transparent;
        border-bottom-color: red;
    }
    #div2{
        border: 30px solid transparent;
        border-bottom-color: red;
    }
    #div3{
        border: 30px solid transparent;
        border-bottom-color: red;
        border-left-width: 10px;
        border-right-width: 10px;
    }
    #div4{
        border: 30px solid transparent;
        border-bottom-color: red;
        border-radius: 30px;
    }
</style>

<div id="div1"></div>
<div id="div2"></div>
<div id="div3"></div>
<div id="div4"></div>
```

![image-20221222162921821](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222162921821.png)

**outline:**

> outline（轮廓）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用

```css
.container {
    width: 20px;
    height: 20px;
    padding: 20px;
    margin: 20px;
    border: 5px solid red;
    outline: 3px solid black;
}
```

![image-20221222163001626](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222163001626.png)

<div class="container"></div>

### 滚动 overflow
- `hidden`： 隐藏超出的内容；
- `visble`： 显示超出的内容；
- `scroll`： 超出容器时显示滚动条；
- `auto`： 自动选择是否显示滚动条；

![image-20221222163058760](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222163058760.png)

### 文字相关样式

**em与rem:**

em是CSS中的一个相对单位，相对于当前对象内文本的字体尺寸。可以作用在width、height、line-height、margin、padding、border等样式的设置上。

如果元素中文本的大小为16px，那么1em=16px;如果元素中文本大小为20px,那么1em=20px……

rem即表示root em，它是根据html根目录下的字体大小进行计算的。当我们改变根目录下的字体大小的时候，下面字体都改变。rem不仅可以设置字体的大小，也可以设置元素宽、高等属性。

**首字下沉:**

```css
p:first-letter{
    padding: 6px;
    font-size: 32pt;
    float: left;
}
```

```html
<p>埋藏中国城枯叶</p>
```

![image-20221222163249263](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222163249263.png)

**文字颜色 color:**

**字体 font-family:**

- 字体族: 字体族表示一类有相同特征的字体，下图中介绍了一些字体族

![image-20221222163442618](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222163442618.png)

- 指定字体

```css
#div1 {
	font-family: "华文彩云";
}
```

- 多字体fallback

> 当用户电脑中没有Corbel时，就使用黑体，如果黑体也没有，那么就monospace字体族。
> 利用这个fallback特性就可以为中文和英文规定不同的字体。
>
> 注意：使用字体族时不能有引号！！！

```css
#div1 {
	font-family: "Corbel", "黑体", monospace;
}
```

![image-20221222163620919](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222163620919.png)

- 自定义字体

```css
#div1 {
    font-family: "IF";
}
@font-face {
    font-family: IF;
    src: url(.....);
}
```

**行高 line-height:**

> **行高的构成及行高的作用：**
>
> 行高是由line box的高度组成的(line box就是把一行内容看做一个虚拟的盒子)，而line box的高度是由inline box的高度决定的(在本例中就是div的高度是由span元素决定的)。
> line-height不决定span元素的高度，但是它决定span元素所占的空间大小，所以经常会看到设置了line-height的元素有留白的情况

```html
<style>
span {
    background-color: red;
    font-size: 20px;
    line-height: 60px;
}
div {
    border: 1px solid black;
}
</style>

<div>
    <span>
        line-height
    </span>
</div>
```

![image-20221222164102080](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222164102080.png)



**字体大小 font-size:**

- xx-small、x-small、small、medium、large、x-large、xx-large： 把字体 
  的尺寸设置为不同的尺寸，从 xx-small 到 xx-large，默认值为medium；
- smaller：把 font-size 设置为比父元素更小的尺寸；
- larger：把 font-size 设置为比父元素更大的尺寸；
- length：把 font-size 设置为一个固定的值；
- %：把 font-size 设置为基于父元素的一个百分比值；

**字重(粗体) font-weight:**

```css
<p class="lighter">字体粗细： lighter</p>
<p class="normal">字体粗细： normal</p>
<p class="bold">字体粗细： bold</p>
<p class="bolder">字体粗细： bolder</p>
```

数字必须是100的整数倍，取值在100~900 之间，值越大字体越粗。400 等同于 normal，700 等同于 bold

**斜体 font-style: itatic;**

**文字水平排列方式 text-align:**

- left: 把文本排列到左边
- right: 把文本排列到右边
- center: 把文本排列到中间
- justify: 实现两端对齐文本效果

**首行缩进 text-index:**

> text-indent 属性规定文本块中首行文本的缩进。允许使用负值。如果使用负值，那么首行会被缩进到左边

```css
.container p{
      text-indent: 2em;
}
```

**下划线 text-decoration:**

```css
/* 设置线型 */
text-decoration-style: dashed | dotted | wavy | double;
/* 设置线的位置 */
text-decoration-line: underline | overline | line-through;
/* 设置线的颜色 */
text-decoration-color: red;
/* 设置线的粗细 */
text-decoration-thickness: 5px;

/* 整合属性 */
text-decoration: underline overline line-through wavy red 5px;
```

**文字溢出显示 text-overflow:**

> `text-overflow` 属性指定当文本溢出包含它的元素时，应该如何显示，可以设置溢出后，文本被剪切、显示省略号 (...) 或显示自定义字符串。`text-overflow` 需要配合以下两个属性使用：
>
> - `white-space: nowrap；`
> - `overflow: hidden；`

`text-overflow`有如下几个可选值：

- `clip`：剪切文本；
- `ellipsis`：显示省略符号 ... 来代表被修剪的文本；
- `string`：使用给定的字符串来代表被修剪的文本；

```css
li{
    width: 150px;
    overflow:hidden;
    text-overflow:ellipsis;
}

<li>fadsfsdafsdfasfasdfdasfadsfasd</li>
<li>fadsfsdafsdfasfasdfdasfadsfasd</li>
<li>fadsfsdafsdfasfasdfdasfadsfasd</li>
```

![image-20221222165351876](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222165351876.png)

### 文字折行

**通用换行控制 overflow-wrap:**

> - normal：换行将按照原始换行规则进行；
> - break-word： 太长而无法放入包含元素的单词会被分成几部分；

```css
#div1 {
    border: 1px solid black;
    width: 200px;
    height: 50px;
    overflow-wrap: normal;
}
#div2 {
    border: 1px solid black;
    width: 200px;
    height: 50px;
    overflow-wrap: break-word;
}

<div id="div1">fjadklfjlksadjfaskldfjlsadkfjalsdf</div><br>
<div id="div2">fjadklfjlksadjfaskldfjlsadkfjalsdf</div>
```

![image-20221222170634681](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222170634681.png)

**指定非CJK换行规则 word-break:** 

> - normal： 使用浏览器默认的换行规则；
> - break-all： 允许在单词内换行；
> - keep-all： 只能在半角空格或连字符处换行；
>
> CJK表示中国、日本和韩国

```html
<div style="width:210px;height: 200px;background: #ccc;word-break:keep-all">
    北京上海 上海上  北京上海 北京 上海上海上海 北京上海 上海 北京上海 上海 北京 
    上海 北京 上海 北京 上海 北京 上海 北京 上海 北京上海 上海 北京上海 上海
</div>

<div style="width:210px;height: 200px;background: #ccc;word-break:break-all">
    北京上海 上海上  北京上海 北京 上海上海上海 北京上海 上海 北京上海 上海 北京 
    上海 北京 上海 北京 上海 北京 上海 北京 上海 北京上海 上海 北京上海 上海
</div>
```

![image-20221222170452437](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222170452437.png)

**指定元素内空白如何处理 white-space:**

> - normal: 默认。空白会被浏览器忽略
> - pre: 空白会被浏览器保留。其行为方式类似 HTML中的 <pre> 标签
> - nowrap: 文本不会换行，文本会在在同一行上继续，直到遇到 <br>标签为止
> - pre-wrap: 保留空白符序列，但是正常地进行换行
> - pre-line: 合并空白符序列，但是保留换行符

```html
<div id="div1" style="white-space: normal;">
fjlkadsfjlkasdjflsadjfasl;dfjasl;
fjasdkljflkadsjf
fjlkads
fdsf
</div>
<div id="div1" style="white-space: pre; word-wrap: break-word;">
fjlkadsfjlkasdjflsadjfasl;dfjasl;
fjasdkljflkadsjf
fjlkads
fdsf
</div>
<div id="div1" style="white-space: nowrap;">
fjlkadsfjlkasdjflsadjfasl;dfjasl;
fjasdkljflkadsjf
fjlkads
fdsf
</div>
<div id="div1" style="white-space: pre-wrap; word-wrap: break-word;">
fjlkadsfjlkasdjflsadjfasl;dfjasl;
fjasdklj      flkadsjf
fjlkads
fdsf
</div>
<div id="div1" style="white-space: pre-line; word-wrap: break-word;">
fjlkadsfjlkasdjflsadjfasl;dfjasl;
fjasdkl       jflkadsjf
fjlk     ads
fdsf
</div>
```

![image-20221222170611616](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222170611616.png)

### 装饰性样式

**指针样式 cursor:**

- cursor: help;
- cursor: wait;
- cursor: crosshair;
- cursor: not-allowed;
- cursor: zoom-in;
- cursor: grab;

**列表符号样式 list-style:**

> list-style 简写属性在一个声明中设置所有的列表属性，可以设置的属性为： list-style-type, list-style-position, list-style-image。当然list-style也可以拆开开来写：

list-style-type：设置列表项标记的类型

- none：无标记

   + disc：默认，标记是实心圆

   + circle：标记是空心圆

   + square：标记是实心方块

   + decimal：标记是数字

list-style-position：指示如何相对于对象的内容绘制列表项标记：

+ inside：列表项目标记放置在文本以内，且环绕文本根据标记对齐
+ outside：默认值。保持标记位于文本的左侧。列表项目标记放置在文本以外，且环绕文本不根据标记对齐

list-style-image：使用图像来替换列表项的标记：

- list-style-image:url('sqpurple.gif')

```css
li{
    border: 1px solid black;
    width: 200px;
}

<ul style="list-style-position: inside;">
    <li>aaaa</li>
    <li>aaaa</li>
</ul>
<ul style="list-style-position: outside;">
    <li>aaaa</li>
    <li>aaaa</li>
</ul>
```

![image-20221222171255295](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222171255295.png)


### table相关样式

**设置单元格是否合并 border-collapse:**

该属性设置表格的边框是否被合并为一个单一的边框，还是像在标准的 HTML 中那样分开显示：

- collapse: 如果可能，边框会合并为一个单一的边框。会忽略 border-spacing 和 empty-cells 属性
- separate: 默认值。边框会被分开。不会忽略 border-spacing 和 empty-cells 属性

**设置相邻单元格距离 border-spacing:**（仅用于"边框分离"模式）

**border-spacing: length length;**

- 规定相邻单元格的边框之间的距离，使用 px、cm 等单位，不允许使用负值；
- 如果定义一个 length 参数，那么定义的是水平和垂直间距；
- 如果定义两个 length 参数，那么第一个设置水平间距，而第二个设置垂直间距；

**设置表格标题的位置 caption-side:**

设置表格标题的位置：

- top：默认值。把表格标题定位在表格之上；
- bottom：把表格标题定位在表格之下；

**是否显示表格中的空单元格 empty-cells:**（仅用于"边框分离"模式）

设置是否显示表格中的空单元格：

- hide：不在空单元格周围绘制边框；
- show：在空单元格周围绘制边框；

### 案例

#### checkbox美化

```css
input {
    display: none;
}
label {
    line-height: 28px;
    background-image: url(./checkbox1.png);
    background-size: 20px 20px;
    background-repeat: no-repeat;
    padding-left: 24px;
}
input:checked + label {
	background-image: url(./checkbox2.png);
}

<form action="">
    <input type="checkbox" id="ckbox1">
    <label for="ckbox1">checkbox</label>
</form>
```

#### radiobox选项卡

```html
<table>
<thead>
    <tr>
        <td>
            <input type="radio" name="table" id="rb1" checked>
            <label for="rb1">tab1</label>
        </td>
        <td>
            <input type="radio" name="table" id="rb2">
            <label for="rb2">tab2</label>
        </td>
        <td>
            <input type="radio" name="table" id="rb3">
            <label for="rb3">tab3</label>
        </td>
    </tr>
</thead>
<tbody>
    <tr>
        <td colspan="3">
            <span>content1</span>
            <span>content2</span>
            <span>content3</span>
        </td>
    </tr>
</tbody>
</table>

<script>
    var radioArray = document.querySelectorAll("thead input")
    var spanArray = document.querySelectorAll("tbody span")
    var head = document.querySelector("thead")
    head.onclick = function() {
        for (var i = 0; i < radioArray.length; i++) {
            if(radioArray[i].checked) {
                spanArray[i].style.display = "inline"
            } else {
                spanArray[i].style.display = "none"
            }
        }
    }
    head.click()
</script>
```

```css
* {
    margin: 0;
    padding: 0;
}
input {
    display: none;
}
table {
    border-collapse: collapse;
    width: 600px;
    border: 1px solid gray;
}
thead td {
    line-height: 40px;
    text-align: center;
    border: 1px solid gray;
    border-bottom: none;
}
tbody {
    height: 200px;
    padding: 20px;
}
thead label {
    display: block;
    width: 100%;
    height: 100%;
    background-color: azure;
    border-bottom: 1px solid gray;
}
thead input:checked+label {
    border-bottom: none;
    background-color: white;
}
```

## CSS布局

### 表格布局

```css
* {
    margin: 0px;
    padding: 0px;
}
table {
    width: 500px;
    height: 200px;
    border-collapse: collapse;
}
.left {
    background-color: red;
    width: 100px;
}
.right {
    background-color: blue;
}
.table {
    margin-top: 20px;
    display: table;
    width: 500px;
    height: 200px;
}
.table-row {
    display: table-row;
}
.table-cell {
    display: table-cell;
    vertical-align: middle;
}

<table>
    <tr>
        <td class="left">左</td>
        <td class="right">右</td>
    </tr>
</table>
<div class="table">
    <div class="table-row">
        <div class="table-cell left">左</div>
        <div class="table-cell right">右</div>
    </div>
</div>
```

![image-20221222212516290](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222212516290.png)

### 布局属性

**盒模型:**

![image-20221222212553318](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222212553318.png)

**display:**

规定元素的显示类型： block | inline | inline-block

**position:**

- 规定元素的位置：
  - static定位： HTML 元素的默认值，即没有定位，遵循正常的文档流对象，静态定位的元素不会受到 top, bottom, left, right影响；
  - fixed 定位：元素的位置相对于浏览器窗口，即使窗口滚动元素的位置也不会发生改变；
  - relative 定位：元素相对于自己原本文档流的位置进行定位，对元素进行相对定位后，它原本所占的空间不会改变，相对定位元素经常被用来作为绝对定位元素的容器块；
  - absolute 定位： 绝对定位的元素的位置相对于最近的已定位父元素，如果元素没有已定位的父元素，那么它的位置相对于<html>，absolute 定位使元素的位置与文档流无关，因此不占据空间；

- 元素重叠：
  - z-index： 元素的定位与文档流无关，所以它们可以覆盖页面上的其它元素，z-index属性指定了一个元素的堆叠顺序。z-index的值是一个数字，可以是正值也可以是负值，值越大会排在越前面

**box-sizing:**

- border-box： width = borderWidth + paddingWidth + contentWidth
- content-box： width = contentWidth

### 弹性盒子布局

**示例:**

```html
<div style="width: 200px; display: flex; background-color: blue;">
    <div style="flex: 1;">left</div>
    <div style="flex: none; width: 120px; background-color: red;">center</div>
    <div style="flex: 1;">right</div>
</div>
```

**flex属性:**

- 父元素设为flex： display: flex;
- 子元素通过flex属性设置比例：flex:1;
- 子元素也可以固定宽度：flex:none; width: 100px;

### float布局

**BFC:**

> 页面元素中的隐含属性：Block Formatting Context 即块格式化上下文，简称BFC，当开启元素的BFC以后，元素会变成一个独立的布局区域，不会在布局上影响到外面的元素，BFC理解为一个封闭的大箱子，箱子内部的元素不会影响到外部

开启BFC后，元素将会具有如下的特性：

1. 父元素的垂直外边距不会和子元素重叠
2. 开启BFC的元素不会被浮动元素所覆盖
3. 开启BFC的元素可以包含浮动的子元素（可解决高度塌陷）

如何开启元素的BFC：

1. 设置元素浮动，使用这种方式开启，虽然可以撑开父元素，但是会导致父元素的宽度丢失，而且使用这种方式也会导致下边的元素上移，不能解决问题；
2. 设置元素为inline-block，这种方式可以解决问题，但是会导致宽度丢失，不推荐使用这种方式；
3. 将元素的overflow设置为一个非visible的值；
4. 设置元素绝对定位；

**浮动的影响:**

> - 浮动的元素会脱离文档流，但不会脱离文本流，因此可以用来做文字环绕的效果。
> - 被设置浮动后的行内元素会形成“块”(BFC)，因此设置浮动后的行内元素可以设置宽高。

```css
.container{
    background:red;
    width:400px;
    margin:20px;
}
.p1{
    background:green;
    float:left;
    width:200px;
    height:50px;
}
.p2{
    clear: left;
}

<div class="container">
    <span class="p1">
        float
    </span>
    <div>
        ....
    </div>
</div>
<div class="container">
    <span class="p1">
        float
    </span>
    <div class="p2">
        ....
    </div>
</div>
```

![image-20221222213609366](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222213609366.png)

**浮动元素的位置:**

> 浮动元素会尽量靠上，向左浮动的元素会尽量靠左，向右浮动的元素会尽量靠右。

```css
.container{
    background:red;
    width:400px;
    margin:20px;
}
.p1{
    background:green;
    float:left;
    width:200px;
    height:50px;
}

<div class="container container2">
    <span>写几个字</span>
    <span class="p1">
        float
    </span>
    <span class="p1">
        float
    </span>
</div>
<div class="container" style="width: 600px; margin-top: 100px;">
    <span>写几个字</span>
    <span class="p1">
        float
    </span>
    <span class="p1">
        float
    </span>
</div>
<div class="container" style="width: 200px; margin-top: 100px;">
    <span>写几个字</span>
    <span class="p1">
        float
    </span>
    <span class="p1">
        float
    </span>
</div>
```

![image-20221222213742983](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222213742983.png)

**解决高度塌陷:**

```css
.container{
    background:red;
    width:300px;
    margin:20px;
}
.container1::after{
    content: 'aaaa';
    height: 0;
    clear: left;
    display: block;
    visibility: hidden;
}
.p1{
    background:green;
    float:left;
    width:200px;
    height:50px;
}

<-- 方法一： 开启父元素的BFC属性 -->
<div class="container" style="overflow: hidden;">
    <span class="p1">写几个字</span>
</div>

<-- 方法二： 设置一个看不见的after -->
<div class="container container1">
    <span class="p1">写几个字</span>
</div>

<-- 方法三： 直接给父元素设置固定高度 -->
<div class="container" style="height: 50px;">
    <span class="p1">写几个字</span>
</div>
```

**浮动布局:**

```css
.main {
    height: 200px;
    width: 600px;
}
.left {
    height: 100%;
    float: left;
    width: 200px;
    background-color: red;
}
.right {
    height: 100%;
    float: right;
    width: 200px;
    background-color: red;
}
.center {
    width: 200px;
    margin-left: 200px;
    background-color: blue;
}

<div class="main">
    <div class="center">center</div>
    <div class="left">left</div>
    <div class="right">right</div>
</div>
<br><br>
<div class="main">
    <div class="left">left</div>
    <div class="center">center</div>
    <div class="right">right</div>
</div>
<br><br>
<div class="main">
    <div class="left">left</div>
    <div class="right">right</div>
    <div class="center">center</div>
</div>
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222214059617.png" alt="image-20221222214059617" style="zoom:80%;" />

- 第一个三联布局中，center被首先渲染，并且因为center是block元素，所以它会把上面的一整行全部占据，导致left和right浮动时只能在它下面找位置；

- 第二个三联布局中，left向左浮动，首先占据了左上位置，然后center还是占据原来的位置，但是因为center为block元素，因right向右浮动时，只能浮动到center的下方位置；

- 第三个三联布局中，left和right首先进行渲染，因为他们浮动时可以占据左上和右上的位置，然后center再进行渲染，占据他原本的位置；

### inline-block布局

> inline-block布局相较于浮动布局，不需要担心高度塌陷的问题，但是是它也会产生新的问题，比如两个inline-block之间的空隙。
> 这是由于代码之间的空格导致的，可以通过设置font-size来解决这个问题，也可以通过直接把空格删除掉来解决：
>
> ```html
> <div class="left">
>     left
> </div><div class="right">
>     right
> </div>
> ```

```css
.main{
    height: 200px;
    width: 450px;
}
.main1{
    height: 200px;
    width: 400px;
    font-size: 0;
}
.left{
    width: 200px;
    height: 200px;
    background-color: red;
    display: inline-block;
    font-size: 14px;
}
.right{
    width: 200px;
    height: 200px;
    background-color: blue;
    display: inline-block;
    font-size: 14px;
}

<div class="main">
    <div class="left">left</div>
    <div class="right">right</div>
</div><br>
<div class="main1">
    <div class="left">left</div>
    <div class="right">right</div>
</div>
```

![image-20221222214311132](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222214311132.png)

### 响应式布局

**meta:**

> 这行代码用于让页面适应不同宽度的设备。

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**rem:**

> rem是相对长度单位。相对于根元素(即html元素)font-size计算值的倍数。比如html设置的字体为20px，那么页面中的1rem就相当于20px。因此rem可以用来做响应式布局：

```css
@media (max-width: 375px){
	html{
		font-size:24px;
	}
}
@media (max-width: 320px){
	html{
		font-size:20px;
	}
}
```

## CSS效果属性

### box-shadow

- h-shadow：必需的。水平阴影的位置。允许负值
- v-shadow：必需的。垂直阴影的位置。允许负值
- blur：可选。模糊距离
- spread：可选。阴影的大小
- color：可选。阴影的颜色
- inset：可选。从外层的阴影（开始时）改变阴影内侧阴影

![image-20221222214643593](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222214643593.png)

### text-shadow

- h-shadow：必需。水平阴影的位置。允许负值；
- v-shadow：必需。垂直阴影的位置。允许负值；
- blur：可选。模糊的距离；
- color：可选。阴影的颜色；

```css
.container{
    font-size: 18px;
    line-height: 2em;
    font-family: STKaiti;
}
.shadow1 {
    text-shadow: 1px 1px 0 #aaa;
}
.shadow2 {
    text-shadow: 0 0 1px rgba(128,128,128,.2);
}
.shadow3 {
    background: black;
    text-shadow: -1px -1px 0 white,
        -1px 1px 0 white,
        1px -1px 0 white,
        1px 1px 0 white;
}

<div class="container shadow1">
    我与父亲不相见已二年余了，我最不能忘记的是他的背影。
</div>
<div class="container shadow2">
    我与父亲不相见已二年余了，我最不能忘记的是他的背影。
</div>
<div class="container shadow3">
    我与父亲不相见已二年余了，我最不能忘记的是他的背影。
</div>
```

![image-20221222214808255](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222214808255.png)


### border-radius

- border-radius可以同时设置1到4个值：如果设置1个值，表示4个圆角都使用这个值。
  `border-radius:40px /20px;`

- 如果设置两个值，表示左上角和右下角使用第一个值，右上角和左下角使用第二个值。
  `border-radius: 10px 50px / 5px 25px;`

- 如果设置三个值，表示左上角使用第一个值，右上角和左下角使用第二个值，右下角使用第三个值。
  `border-radius: 10px 50px 40px / 5px 25px 20px;`

- 如果设置四个值，则依次对应左上角、右上角、右下角、左下角（顺时针顺序）。
  `border-radius: 10px 20px 30px 40px / 5px 10px 15px 20px;`

- 单独设置一个角的写法：
  `border-top-left-radius: 20px 10px; //设置左上角`
  `border-top-right-radius: 20px 10px; //设置右上角`
  `border-bottom-left-radius: 20px 10px; //设置左下角`
  `border-bottom-right-radius: 20px 10px; //设置左下角`

border-radius的值可以设置百分比，50%的时候为圆形。也可以设置为准备数值，但是设为数值时，要把border的宽度也考虑进去

```css
.container{
    width: 100px;
    height: 100px;
    background: red;
    border: 50px solid green;
    display: inline-block;
}
.rd1{
    border-radius: 50%;
}
.rd2{
    border-radius: 50px;
}
.rd3{
    border-radius: 100px;
}
<div class="container rd1"></div>
<div class="container rd2"></div>
<div class="container rd3"></div>
```

![image-20221222215147362](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222215147362.png)

### clip-path

> clip-path 属性可以创建一个只有元素的部分区域可以显示的剪切区域。区域内的部分显示，区域外的隐藏。剪切区域是被引用内嵌的URL定义的路径或者外部svg的路径，或者作为一个形状

- 基本图形inset：定义一个矩形，可以传入5个参数，分别对应top,right,bottom,left的裁剪位置,round radius；
  `clip-path: inset(100px 50px 100px 50px round 20px)`

- 基本图形circle：定义一个圆，可以传人2个可选参数，一个是圆的半径，另一个是圆心位置；
  `circle(30% at 150px 120px)`

- 基本图形ellipse： 定义一个椭圆，可以传人3个可选参数，首先是椭圆的X轴半径和椭圆的Y轴半径，还有就是椭圆中心位置；
  `clip-path: ellipse(45% 30% at 50% 50%)`

- 基本图形polygon：定义一个多边形，可以传入任意多个参数，表示多边形的连接点；
  `clip-path: polygon(50% 0,100% 50%,0 100%)`

- 还可以传一个svg图形的引用;

  `clip-path: url(#clipPath)`

**变形动画:**

```css
.outer{
    width:100px;
    height: 100px;
    background:orange;
    clip-path: polygon(20% 0%, 80% 0%, 100% 20%, 100% 80%, 80% 100%, 20% 100%, 0% 80%, 0% 20%);
    transition:.5s clip-path;
}  
.outer:hover{
    clip-path:polygon(0 0,0 0,100% 0,100% 0,100% 100%,100% 100%,0 100%,0 100%);
}

<div class="outer"></div>
```

**svg path:**

```css
#clipped {
    margin-bottom: 20px;
    clip-path: url(#cross);
}

<img id="clipped" src="https://mdn.mozillademos.org/files/12668/MDN.svg" alt="MDN logo">

<svg height="0" width="0">
    <defs>
        <clipPath id="cross">
        <rect y="110" x="137" width="90" height="90"/>
        <rect x="0" y="110" width="90" height="90"/>
        <rect x="137" y="0" width="90" height="90"/>
        <rect x="0" y="0" width="90" height="90"/>
        </clipPath>
    </defs>
</svg>
```

![image-20221222215854484](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222215854484.png)


### transform变形

**transform:**

1. rotate(xx deg)(2D), rotateX()(3D), rotateY()(3D)：
   以中心为基点，deg表示旋转的角度，为负数时表示逆时针旋转。

2. translate(x,y) ，translateX(x) ，translateY(y)：
   以中心为基点按照设定的x,y参数值,对元素进行进行平移。

3. scale(x,y)，scaleX(x)， scaleY(Y)：
   以中心为基点进行绽放，缩放基数为1，如果其值大于1元素就放大，反之其值小于1为缩小。

4. skew(x,y)， skewX(x)， skewY(y)：
   以中心为基点，第一个参数是水平方向扭曲角度，第二个参数是垂直方向扭曲角度。

**transform-origin:**

所有操作的默认的基点都在中心位置，我们可以使用transform-origin：(x,y,z)来改变元素基点

- x是水平方向取值，可以取left，center ，right，固定数值和百分比；
- y是垂直方向的取值，可以取top ，center， bottom，固定数值和百分比；
- z只可以取固定数值；

**transform-style:**

transform-style 属性用于设置元素的子元素是位于 3D 空间中还是平面中，如果选择平面，元素的子元素将不会有 3D 的遮挡关系。由于这个属性不会被继承，因此必须为元素的所有非叶子子元素设置它

- flat：设置元素的子元素位于该元素的平面中；
- preserve-3d：指示元素的子元素应位于 3D 空间中；

**prespective:**

perspective元素指定了观察者与 z=0 平面的距离，使具有三维位置变换的元素产生透视效果。 z>0 的三维元素比正常大，而 z<0 时则比正常小，大小程度由该属性的值决定。三维元素在观察者后面的部分不会绘制出来，即 z 轴坐标值大于 perspective 属性值的部分

当为元素定义 perspective 属性时，其子元素会获得透视效果，而不是元素本身。perspective 属性只影响 3D transform元素

**prespective-origin:**

perspective-origin 指定了观察者的位置，与 perspective 属性一起使用：

`perspective-origin: x-position y-position;`

- <length-percentage> 长度值或相对于元素宽度的百分比值，可为负值；
- left, 关键字，0值的简记；
- center, 关键字，50%的简记；
- right, 关键字，100%的简记；

**backface-visibility:**

- backface-visibility 属性定义当元素不面向屏幕时是否可见：
=>  visible：背面是可见的；
=>  hidden：背面是不可见的；

**图解透视:**

> - 透视距离perspective是指观察者沿着平行于z轴的视线与屏幕之间的距离，简称视距；
> - 透视原点perspective-origin是指观察者的位置，一般地，观察者位于与屏幕平行的另一个平面上，观察者始终是与屏幕垂直的。观察者的活动区域是被观察元素的盒模型区域；

![image-20221222220550139](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222220550139.png)

**立方体示例:**


``` css
.container{
    margin:50px;
    padding: 10px;
    border: 1px solid red;
    width: 200px;
    height: 200px;
    position: relative;
    perspective: 500px;
    perspective-origin: 50px 50px;
}
#cube{
    width:200px;
    height:200px;
    transform-style: preserve-3d;
    transform: translateZ(-100px);
    transition: transform .4s;
}
#cube div{
    width: 200px;
    height:200px;
    position: absolute;
    line-height: 200px;
    font-size:50px;
    text-align: center;
    /* backface-visibility: hidden; */
}
#cube:hover{
    transform: translateZ(-100px) rotateX(90deg) rotateY(90deg);
}
.front{
    transform: translateZ(100px);
    background:rgba(255,0,0,.3);
}
.back{
    transform-origin: 50% 50% -50px;
    transform: rotateY(180deg);
    background:rgba(0,255,0,.3);
}
.left{
    transform: translateX(-100px) rotateY(-90deg);
    background:rgba(0,0,255,.3);
}
.right{
    transform: translateX(100px) rotateY(90deg);
    background:rgba(255,255,0,.3);
}
.top{
    transform: translateY(-100px) rotateX(90deg);
    background:rgba(255,0,255,.3);
}
.bottom{
    transform: translateY(100px) rotateX(-90deg);
    background:rgba(0,255,255,.3);
}

<div class="container">
    <div id="cube">
        <div class="front">1</div>
        <div class="back">2</div>
        <div class="right">3</div>
        <div class="left">4</div>
        <div class="top">5</div>
        <div class="bottom">6</div>
    </div>
</div>
```

![image-20221222220700120](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222220700120.png)

### 动画

#### timing-function

**cubic-bezier:**

> timing-function的工作图如下，总共有4个点来描述整个曲线的运动形状，其中P0和P3是开始和截止的位置，关键位置是P1和P2，那么P1的坐标(x,y)就对应了cubic-bezier(n,n,n,n)的前两个n的值，而P2的坐标对应了后两个n的值。然后将P0、P1连线、P2、P3连线，此时这两条线就是整条曲线的切线。
>
> 运动曲线怎么确定动画的速度：
> 可以把这个坐标系横轴的值当做时间，纵轴的值当做关键帧之间属性变化的数值。这样这条曲线的平陡程度就是动画快慢的反应，即越陡的部分动画反应出来就是越快，越平的部分当然动画反应的就是越慢了。

![image-20221222220843193](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222220843193.png)

**steps:**

> animation默认以ease方式过渡，它会在每个关键帧之间插入补间动画，所以动画效果是连贯性的除了ease，linear、cubic-bezier之类的过渡函数都会为其插入补间。但有些效果不需要补间，只需要关键帧之间的跳跃(逐帧动画)，这时应该使用steps过渡方式。
>
> steps 函数指定了一个阶跃函数，它的第一个参数指定了时间函数中的间隔数量（必须是正整数），第二个参数可选，接受 start 和 end 两个值，指定在每个间隔的起点或是终点发生阶跃变化，默认为 end。

![image-20221222220941803](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222220941803.png)

#### transition过渡动画

**transition:**

> transition 属性是 transition-property，transition-duration，transition-timing-function 和 transition-delay 的一个简写属性。
>
> 过渡可以为一个元素在不同状态之间切换的时候定义不同的过渡效果。比如在不同的伪元素之间切换，像是 :hover，:active 或者通过 JavaScript 实现的状态变化。
>
> transition属性可以被指定为一个或多个 CSS 属性的过渡效果，多个属性之间用逗号进行分隔。

`transition: margin-right 4s, color 1s;`

`transition: all 0.5s ease-out 1s;`

**transition-property:**

> transition-property 指定应用过渡属性的名称。如果指定简写属性（比如background），那么其完整版中所有可以动画的属性都会被应用过渡。

- none：没有过渡动画
- all：所有可被动画的属性都表现出过渡动画
- IDENT：属性名称

**transition-duration:**

> transition-duration 属性以秒或毫秒为单位指定过渡动画所需的时间。默认值为 0s ，表示不出现过渡动画。
>
> 可以指定多个时长，每个时长会被应用到由 transition-property 指定的对应属性上。如果指定的时长个数小于属性个数，那么时长列表会重复。如果时长列表更长，那么该列表会被裁减。两种情况下，属性列表都保持不变。

**transition-timing-function:**

> CSS属性受到 transition effect的影响，会产生不断变化的中间值，而 CSS transition-timing-function 属性用来描述这个中间值是怎样计算的。实质上，通过这个函数会建立一条加速度曲线，因此在整个transition变化过程中，变化速度可以不断改变。
>
> 你可以规定多个timing function,通过使用 transition-property属性，可以根据主列表(transition property的列表)给每个CSS属性应用相应的timing function.如果timing function的个数比主列表中数量少，缺少的值被设置为初始值（ease） 。如果timing function比主列表要多，timing function函数列表会被截断至合适的大小。这两种情况下声明的CSS属性都是有效的。

**transition-delay:**

> transition-delay属性规定了在过渡效果开始作用之前需要等待的时间。
>
> 值以秒（s）或毫秒（ms）为单位，表明动画过渡效果将在何时开始。取值为正时会延迟一段时间来响应过渡效果；取值为负时会导致过渡立即开始。
>
> 你可以指定多个延迟时间，每个延迟将会分别作用于你所指定的相符合的css属性（transition-property）。

`transition-delay: 2s, 4ms;`

#### keyframe关键帧动画

**animation:**

> animation 属性是 animation-name，animation-duration, 
> animation-timing-function，animation-delay，
> animation-iteration-count，animation-direction，
> animation-fill-mode 和 animation-play-state 属性的一个简写属性形式。

**animation-name:**

> animation-name属性指定应用的一系列动画，每个名称代表一个由@keyframes定义的动画序列。

- none：特殊关键字，表示无关键帧。可以不改变其他标识符的顺序而使动画失效，或者使层叠的动画样式失效。
- IDENT：标识动画的字符串。

**animation-duration:**

> animation-duration属性指定一个动画周期的时长。默认值为0s，表示无动画。

**animation-timing-function:**

> animation-timing-function属性定义CSS动画在每一动画周期中执行的节奏。
>
> 对于关键帧动画来说，timing function作用于一个关键帧周期而非整个动画周期，即从关键帧开始开始，到关键帧结束结束。
>
> 由于timing function作用于一个关键帧周期，因此可以为每个关键帧周期设置不同的timing function：

`animation-timing-function: ease, step-start, cubic-bezier(0.1, 0.7, 1.0, 0.1);`

**animation-delay:**

> animation-delay 属性定义动画于何时开始，即从动画应用在元素上到动画开始的这段时间的长度。
>
> 0s是该属性的默认值，代表动画在应用到元素上后立即开始执行。否则，该属性的值代表动画样式应用到元素上后到开始执行前的时间长度。
>
> 定义一个负值会让动画立即开始。但是动画会从它的动画序列中某位置开始。例如，如果设定值为-1s，动画会从它的动画序列的第1秒位置处立即开始。

**animation-iteration-count:**

> animation-iteration-count 属性定义动画在结束前运行的次数：

- infinite：无限循环播放动画。
- <number>：动画播放的次数；默认值为1。可以用小数定义循环，来播放动画周期的一部分：例如，0.5 将播放到动画周期的一半。不可为负值。

**animation-direction:**

> animation-direction 属性指示动画是否反向播放：

- normal：每个循环内动画向前循环，换言之，每个动画循环结束，动画重置到起点重新开始，这是默认属性。
- alternate：动画交替反向运行，反向运行时，动画按步后退，同时timing function也反向，比如，ease-in 在反向时成为 ease-out。
- reverse：反向运行动画，每周期结束动画由尾到头运行。
- alternate-reverse：反向交替， 反向开始交替。动画第一次运行时是反向的，然后下一次是正向，后面依次循环。

**animation-fill-mode:**

> animation-fill-mode 属性设置CSS动画在执行之前和之后如何将样式应用于其目标：

- none：当动画未执行时，动画将不会将任何样式应用于目标，而是已经赋予给该元素的 CSS 规则来显示该元素。这是默认值。
- forwards：目标将保留由执行期间遇到的最后一个关键帧计算值。 最后一个关键帧取决于animation-direction和animation-iteration-count的值。
- backwards：动画将在应用于目标时立即应用第一个关键帧中定义的值，并在animation-delay期间保留此值。 第一个关键帧取决于animation-direction的值。

![image-20221222223437253](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222223437253.png)

**animation-play-state:**

> animation-play-state 属性定义一个动画是否运行或者暂停。可以通过查询它来确定动画是否正在运行。另外，它的值可以被设置为暂停和恢复的动画的重放。
>
> 恢复一个已暂停的动画，将从它开始暂停的时候，而不是从动画序列的起点开始在动画。

- running：当前动画正在运行。
- paused：当前动画已被停止。

#### 过渡动画与关键帧动画的区别

- transition过渡动画是在属性发生变化时才会触发；
- animation关键帧动画则可以自动触发。

#### 逐帧动画

> animation-timing-function: steps(1) 就是为了逐帧动画而生的

```css
.container{
    width: 100px;
    height: 100px;
    border: 1px solid red;
    background: url(./animal.png) no-repeat;
    animation: run 1s infinite;
    animation-timing-function: steps(1, start);
}
@keyframes run{
    0%{
        background-position: 0 0;
    }
    12.5%{
        background-position: -100px 0;
    }
    25%{
        background-position: -200px 0;
    }
    37.5%{
        background-position: -300px 0;
    }
    50%{
        background-position: 0 -100px;
    }
    62.5%{
        background-position: -100px -100px;
    }
    75%{
        background-position: -200px -100px;
    }
    87.5%{
        background-position: -300px -100px;
    }
    100%{
        background-position: 0 0;
    }
}
```

![image-20221222223615813](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222223615813.png)

## CSS预处理器





### 预处理器框架

- =>  SASS - Compass；
  =>  Less - Lesshat / EST；
  =>  这些库提供现成的mixin，就类似于JS类库 
      存，封闭了一些常用的功能。

