# Grid布局

> - **容器和项目**：我们通过在元素上声明`display：grid`或`display：inline-grid`来创建一个网格容器。一旦我们这样做，这个元素的所有直系子元素将成为网格项目。比如上面`.wrapper`所在的元素为一个网格容器，其直系子元素将成为网格项目
>
> - **网格轨道**：`grid-template-columns`和`grid-template-rows`属性来定义网格中的行和列。容器内部的水平区域称为行，垂直区域称为列。图中`One`、`Two`、`Three`组成了一行，`One`、`Four`则是一列
>
> - **网格单元**：一个网格单元是在一个网格元素中最小的单位，从概念上来讲其实它和表格的一个单元格很像。图中`One`、`Two`、`Three`、`Four`都是一个个的网格单元
>
>- **网格线**：划分网格的线，称为"网格线"。应该注意的是，当定义网格时，我们定义的是网格轨道，而不是网格线。Grid会创建编号的网格线来来定位每一个网格元素。m列有m + 1根垂直的网格线，n行有n + 1跟水平网格线。图示例中就有4根垂直网格线。一般而言，是从左到右，从上到下，1，2，3进行编号排序。当然也可以从右到左，从下到上，按照-1，-2，-3...顺序进行编号排序
>

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591934e1560~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="网格线" style="zoom:67%;" />

`Grid`布局属性可以分为两大类，一类是容器属性，一类是项目属性

## 容器属性

### grid-template-columns/rows

`grid-template-columns`属性设置列宽，`grid-template-rows`属性设置行高，这两个属性在`Grid`布局中尤为重要

#### 固定的列宽和行高

```css
.wrapper {
  display: grid;
  /*  声明了三列，宽度分别为 200px 100px 200px */
  grid-template-columns: 200px 100px 200px;
  grid-gap: 5px;
  /*  声明了两行，行高分别为 50px 50px  */
  grid-template-rows: 50px 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591c0fc1214~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="固定行高和列宽" style="zoom:67%;" />

#### repeat函数

> 可以使用repeat函数简化重复的值。该函数接受两个参数，第一个参数是重复的次数，第二个参数是所要重复的值
>
> 比如上面的例子中item的高度是相同的，那么就可以通过repeat函数简化上面例子

```css
.wrapper-1 {
  display: grid;
  grid-template-columns: 200px 100px 200px;
  grid-gap: 5px;
  /*  2行，而且行高都为 50px  */
  grid-template-rows: repeat(2, 50px);
}
```

#### auto-fill关键字

> 表示自动填充，让一行（或者一列）中尽可能的容纳更多的单元格。`grid-template-columns: repeat(auto-fill, 200px)`表示列宽是200px，但列的数量是不固定的，只要浏览器能够容纳得下，就可以放置元素

```css
.wrapper-2 {
  display: grid;
  grid-template-columns: repeat(auto-fill, 200px);
  grid-gap: 5px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591c300e81a~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:67%;" />

#### fr关键字

> `Grid`布局还引入了一个另外的长度单位来帮助我们创建灵活的网格轨道。`fr`单位代表网格容器中可用空间的一等份。`grid-template-columns: 200px 1fr 2fr`表示第一个列宽设置为200px，后面剩余的宽度分为两部分，宽度分别为剩余宽度的1/3和2/3

```css
.wrapper-3 {
  display: grid;
  grid-template-columns: 200px 1fr 2fr;
  grid-gap: 5px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591ccc256d1~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:67%;" />

#### minmax函数

>我们有时候想给网格元素一个最小和最大的尺寸，`minmax()`函数产生一个长度范围，表示长度就在这个范围之中都可以应用到网格项目中。它接受两个参数，分别为最小值和最大值。`grid-template-columns: 1fr 1fr minmax(300px, 2fr)`的意思是，第三个列宽最少也是要300px，但是最大不能大于第一第二列宽的两倍

```css
.wrapper-4 {
  display: grid;
  grid-template-columns: 1fr 1fr minmax(300px, 2fr);
  grid-gap: 5px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591dc05edac~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:67%;" />

#### auto关键字

> 由浏览器决定长度。通过`auto`关键字，我们可以轻易实现三列或者两列布局。`grid-template-columns: 100px auto 100px`表示第一第三列为100px，中间由浏览器决定长度

```css
.wrapper-5 {
  display: grid;
  grid-template-columns: 100px auto 100px;
  grid-gap: 5px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591f2146e1d~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:67%;" />

### gap

> `row-gap`属性、`grid-column-gap`属性分别设置行间距和列间距。` grid-gap`属性是两者的简写形式

```css
.wrapper {
  display: grid;
  grid-template-columns: 200px 100px 100px;
  gap: 10px 20px;
  grid-auto-rows: 50px;
}
```

```css
.wrapper-1 {
  display: grid;
  grid-template-columns: 200px 100px 100px;
  grid-auto-rows: 50px;
  row-gap: 10px;
  column-gap: 20px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389591f78de6f2~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="img" style="zoom: 50%;" />

### grid-template-areas

> `grid-template-areas`属性用于定义区域，一个区域由一个或者多个单元格组成
>
> 一般这个属性跟网格元素的`grid-area`一起使用，我们在这里一起介绍。`grid-area`属性指定项目放在哪一个区域

```CSS
.wrapper {
  display: grid;
  grid-gap: 10px;
  grid-template-columns: 120px  120px  120px;
  grid-template-areas:
    ". header  header"
    "sidebar content content";
  background-color: #fff;
  color: #444;
}
```

上面代码表示划分出6个单元格，其中值得注意的是`.`符号代表空的单元格，也就是没有用到该单元格

```css
.sidebar {
  grid-area: sidebar;
}
.content {
  grid-area: content;
}
.header {
  grid-area: header;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895920bbe824a~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="img" style="zoom: 50%;" />

### grid-auto-flow

>`grid-auto-flow`属性控制着自动布局算法怎样运作，精确指定在网格中被自动布局的元素怎样排列。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行，即下图英文数字的顺序`one`,`two`,`three`...
>
>这个顺序由`grid-auto-flow`属性决定，默认值是`row`

```css
.wrapper {
  display: grid;
  grid-template-columns: 100px 200px 100px;
  grid-auto-flow: row;
  grid-gap: 5px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895921548265c~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="img" style="zoom: 50%;" />

如果想要使item尽量的填充空白区域，可以设置`grid-auto-flow: row dense`

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895923612a19b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom: 50%;" />

以下是`grid-auto-flow: column`（表示先列后行）的效果

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895923f11dd83~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom: 50%;" />

### justify/align-items

> `justify-items`属性设置单元格内容的水平位置（左中右），`align-items`属性设置单元格的垂直位置（上中下）

```css
.container {
  justify-items: start | end | center | stretch;
  align-items: start | end | center | stretch;
}
```

- start：对齐单元格的起始边缘

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/1738959244947d96~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

- end：对齐单元格的结束边缘

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389592560e3fc2~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

- center：单元格内部居中

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895925bd879fa~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

- stretch：拉伸，占满单元格的整个宽度（默认值）

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/1738959270057d0c~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

### justify/align-content

> `justify-content`属性是整个内容区域在容器里面的水平位置（左中右），`align-content`属性是整个内容区域的垂直位置（上中下）

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;  
}
```

- start - 对齐容器的起始边框
- end - 对齐容器的结束边框
- center - 容器内部居中

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895926d20f5d6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

- space-around - 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与容器边框的间隔大一倍
- space-between - 项目与项目的间隔相等，项目与容器边框之间没有间隔
- space-evenly - 项目与项目的间隔相等，项目与容器边框之间也是同样长度的间隔
- stretch - 项目大小没有指定时，拉伸占据整个网格容器

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895927ba770c4~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

### grid-auto-columns/rows

> **隐式和显示网格**：显式网格包含了你在`grid-template-columns`和`grid-template-rows`属性中定义的行和列。如果你在网格定义之外又放了一些东西，或者因为内容的数量而需要的更多网格轨道的时候，网格将会在隐式网格中创建行和列
>
> 假如有多余的网格（也就是上面提到的隐式网格），那么它的行高和列宽可以根据`grid-auto-columns`属性和`grid-auto-rows`属性设置。它们的写法和` grid-template-columns`和`grid-template-rows`完全相同。如果不指定这两个属性，浏览器完全根据单元格内容的大小，决定新增网格的列宽和行高

```css
.wrapper {
  display: grid;
  grid-template-columns: 200px 100px;
/*  只设置了两行，但实际的数量会超出两行，超出的行高会以 grid-auto-rows 算 */
  grid-template-rows: 100px 100px;
  grid-gap: 10px 20px;
  grid-auto-rows: 50px;
}
```

`grid-template-columns`属性和`grid-template-rows`属性只是指定了两行两列，但实际有九个元素，就会产生隐式网格。通过`grid-auto-rows`可以指定隐式网格的行高为 50px

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895927d99af1c~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="img" style="zoom:50%;" />

## 项目属性

### grid-column/row-start/end

> - grid-column-start 属性：左边框所在的垂直网格线
> - grid-column-end 属性：右边框所在的垂直网格线
> - grid-row-start 属性：上边框所在的水平网格线
> - grid-row-end 属性：下边框所在的水平网格线

```css
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 20px;
  grid-auto-rows: minmax(100px, auto);
}
.one {
  grid-column-start: 1;
  grid-column-end: 2;
  background: #19CAAD;
}
.two { 
  grid-column-start: 2;
  grid-column-end: 4;
  grid-row-start: 1;
  grid-row-end: 2;
  /*   如果有重叠，就使用 z-index */
  z-index: 1;
  background: #8CC7B5;
}
.three {
  grid-column-start: 3;
  grid-column-end: 4;
  grid-row-start: 1;
  grid-row-end: 4;
  background: #D1BA74;
}
.four {
  grid-column-start: 1;
  grid-column-end: 2;
  grid-row-start: 2;
  grid-row-end: 5;
  background: #BEE7E9;
}
.five {
  grid-column-start: 2;
  grid-column-end: 2;
  grid-row-start: 2;
  grid-row-end: 5;
  background: #E6CEAC;
}
.six {
  grid-column: 3;
  grid-row: 4;
  background: #ECAD9E;
}
```

上面代码中，类`.two`所在的网格项目，垂直网格线是从2到4，水平网格线是从1到2。其中它跟`.three`（垂直网格线是从3到4，水平网格线是从1到4）是有冲突的。可以设置`z-index`去决定它们的层级关系

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/173895928bc7e88e~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="img" style="zoom:50%;" />

### grid-area

> `grid-area`属性指定项目放在哪一个区域，区域名则是由`grid-template-areas`属性进行指定

### justify/align-self

> `justify-self`属性设置单元格内容的水平位置（左中右），跟`justify-items`属性的用法完全一致，但只作用于单个项目
>
> `align-self`属性设置单元格内容的垂直位置（上中下），跟`align-items`属性的用法完全一致，也是只作用于单个项目

```css
.item {
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
}
```

## 实现响应式布局

### fr实现等分响应式

> `fr`等分单位，可以将容器的可用空间分成想要的多个等分空间。利用这个特性，能够轻易实现一个等分响应式

```css
.wrapper {
  margin: 50px;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-gap: 10px 20px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389592bf7e44dd~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom: 50%;" />

### repeat+auto-fill 固定列宽，动态列数量

> 等分布局并不只有`Grid`布局才有，像`flex`布局也能轻松实现，接下来看看更高级的响应式
>
> 上面例子的始终都是三列的，但是需求往往希望我们的网格能够固定列宽，并根据容器的宽度来改变列的数量。这个时候，可以用到上面提到`repeat()`函数以及`auto-fit`关键字。`grid-template-columns: repeat(auto-fill, 200px)`表示固定列宽为 200px，数量是自适应的，只要容纳得下，就会往上排列

```css
.wrapper {
  margin: 50px;
  display: grid;
  grid-template-columns: repeat(auto-fit, 200px);
  grid-gap: 10px 20px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389592c297495a~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:67%;" />

### repeat+auto-fill+minmax去掉右侧空白

> 上面看到的效果中，右侧通常会留下空白。而`minmax()`函数可以实现列的宽度在某个范围内自适应。将`grid-template-columns: repeat(auto-fit, 200px)`改成`grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))`表示列宽至少200px，如果还有空余则一起等分

```css
.wrapper {
  margin: 50px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-gap: 10px 20px;
  grid-auto-rows: 50px;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389592cc3c2bf9~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="auto-auto-minmax.gif" style="zoom: 50%;" />

### repeat+auto-fit+minmax+dense解决空缺问题

> 上面的布局看起来没什么问题，但是每个网格元素的长度可能不相同，这也简单，通过`span`关键字进行设置网格项目的跨度，`grid-column-start: span 3`，表示这个网格项目跨度为3

```css
.item-4, .item-6 {
  grid-column-start: span 3;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389592f9da3763~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom:50%;" />

但是现在右侧又会出现空白，这个时候就就需要用到`dense`关键字。`grid-auto-flow: row dense`表示尽可能填充，而不留空白

```css
.wrapper {
  margin: 50px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-gap: 10px 20px;
  grid-auto-rows: 50px;
  grid-auto-flow: row dense;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/17389593009f7fe7~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp" alt="image" style="zoom: 50%;" />
