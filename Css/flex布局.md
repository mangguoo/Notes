# flex布局

## 容器与项目

> - 采用flex布局的元素被称作容器
> - 在flex布局中的子元素被称作项目
>
> **即父级元素采用flex布局，则父级元素为容器，全部子元素自动成为项目**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826161110660.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size

## 容器的一些属性

> - flex-direction
> - flex-wrap
> - flew-flow
> - justify-content
> - align-items
> - align-content

### flex-direction属性

> **flex-direction 属性设置容器主轴的方向**

```css
.wrap{
    flex-direction:row | row-reverse | column | column=reverse;
}
```

- row: 默认值，表示沿水平方向，由左到右

- row-reverse: 表示沿水平方向，由右到左

- column: 表示垂直方向，由上到下

- column-reverse: 表示垂直方向，由下到上

### flex-wrap属性

> **flex-wrap属性用于设置当项目在容器中一行无法显示的时候如何处理**

```css
.wrap{
    flex-wrap:nowrap | wrap | wrap-reverse;
}
```

- nowrap: 表示不换行(设置的项目的宽度就失效了，强行在一行显示)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826162931372.png)

- wrap：正常换行，第一个位于第一行的第一个

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826164654292.png)

- wrap-reverse：向上换行，第一行位于下方

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826164616862.png)

### flex-flow属性

> **flex-flow属性是flex-deriction和flex-wrap属性的简写，默认值为[row nowrap];**

- 第一个属性值为flex-direction的属性值

- 第二个属性值为flex-wrap的属性值

### justify-content属性

> **justify-content属性用于设置项目在容器中的对齐方式**

```css
.wrap{
    justify-content: flex-start | flex-end | center |space-between | space-around
}
```

- flex-start：默认值，左对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826165647428.png)

- flex-end：右对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826165705105.png)

- center：居中对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826165917704.png)

- space-between：两端对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826165942170.png)

- space-evenly: 项目之间间距与项目与容器间距相等，相当于除去项目宽度，平均分配了剩余宽度作为项目左右margin

![v2-7b1d442bea60e27638c348419777bc4d_720wfdsf](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/v2-7b1d442bea60e27638c348419777bc4d_720wfdsf.jpg)

- space-around: 项目之间间距为左右两侧项目到容器间距的2倍

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826170003803.png)

### align-items属性

> **align-items定义了项目在交叉轴上是如何对齐显示的**

```css
.wrap{
    align-items:flex-start | flex-end | center | baseline | stretch
}
```

- flex-start:交叉轴的起点对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826170642812.png)

- flex-end 交叉轴的终点对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826170726171.png)

- center 交叉轴居中对齐

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826170850724.png)

- baseline 项目的第一行文字的基线对齐

![v2-71ff4a27b4e71fc646ed5d60ebba33f9_720wfdsafdfs](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/v2-71ff4a27b4e71fc646ed5d60ebba33f9_720wfdsafdfs.jpg)

- stretch：默认值：如果项目未设置高度或者高度为auto，将占满整个容器的高度

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826171005498.png)

### align-content属性

> **align-content属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用**

```css
.box {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

- flex-start：与交叉轴的起点对齐
- flex-end：与交叉轴的终点对齐
- center：与交叉轴的中点对齐
- space-between：与交叉轴两端对齐，轴线之间的间隔平均分布
- space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍
- stretch（默认值）：轴线占满整个交叉轴
- space-evenly: 项目之间间距与项目到容器之间间距相等

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826190133439.png)

![v2-918244a22a79e6a81030d13b374ac031_720wfdsafasdfas4564654](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/v2-918244a22a79e6a81030d13b374ac031_720wfdsafasdfas4564654.jpg)

### gap属性

> 在以前，使用 flex 布局一般通过 `margin` 来设置边距，而且需要考虑换行的情况，也就是设置 `flex-wrap` 为 `wrap` 的情况下，下一行的元素会紧贴着上一行的元素，要是想给这两行元素同时设置列间距或行间距的话，那么需要设置上边距和左边距。这样就会有一个问题，使用 `margin` 设置边距，根据边距的位置，第一个或最后一个元素会有多余的边距。
>
> 而如果使用 gap 属性可以避免这种情况，我们可以直接像 css grid 布局中一样，给 flex 布局设置一个 gap 属性，比如说 24 像素，那么 flex 布局下边的每个元素之间，就会有一个 24 像素的空隙，它的两边也不会有多余的边距。并且，如果有折行的话，每行之间也会有同样的间距

gap 属性是用来设置网格行与列之间的间隙，该属性是row-gap and column-gap的简写形式

- row-gap: 设置网格布局中行之间的间隙大小

- column-gap: 设置网格布局中列之间的间隙大小

```css
gap: row-gap column-gap;
```

## 项目的一些属性

> - order

### order属性

> order属性设置项目排序的位置，默认值为0，数值越小越靠前

```css
.item{
    order:<Integer>;
}
.green-item{
    order:-1;
}
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826171929418.png)

### flex-grow属性

> **默认0，用于决定项目在有剩余空间的情况下是否放大，默认不放大；注意，即便设置了固定宽度，也会放大**
>
> - 假设默认三个项目中前两个个项目都是0，最后一个是1，最后的项目会沾满剩余所有空间
> - 假设只有第一个项目默认为0，后面两个项目flex-grow均为1，那么后两个项目平分剩余空间
> - 假设第一个项目默认为0，第二个项目为flex-grow:2，最后一个项目为1，则第二个项目在放大时所占空间是最后项目的两倍

```css
.green-item{
    order:-1;
    flex-grow:2;
}
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826183920378.png)

### flex-shrink属性

> **默认1，用于决定项目在空间不足时是否缩小，默认项目都是1，即空间不足时大家一起等比缩小；注意，即便设置了固定宽度，也会缩小，但如果某个项目flex-shrink设置为0，则即便空间不够，自身也不缩小**

### flex-basis属性

> **默认auto，用于设置项目宽度，默认auto时，项目会保持默认宽度，或者以width为自身的宽度，但如果设置了flex-basis，权重会比width属性高，因此会覆盖widtn属性**

### flex属性

> **flex属性是 flex-grow属性、flex-shrink属性、flex-basis属性的简写。默认值为：0 1 auto；**

```css
.item{
    flex:(0 1 auto) | auto(1 1 auto) | none (0 0 auto)
}
```

### align-self属性

> **align-self属性表示当前项目可以和其他项目拥有不一样的对齐方式。它有六个可能的值。默认值为auto**　

- auto：和父元素align-self的值一致

- flex-start：顶端对齐

- flex-end：底部对齐

- center：竖直方向上居中对齐

- baseline：item第一行文字的底部对齐

- stretch：当item未设置高度时，item将和容器等高对齐

```css
.item{
    align-self: flex-start | flex-end | center | baseline | stretch 
}
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20200826184505238.png)