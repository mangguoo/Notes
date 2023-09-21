# @supports

> CSS中的@supports用于检测浏览器是否支持CSS的某个属性。其实就是条件判断，如果支持某个属性可以写一套样式，如果不支持某个属性，可以提供另外一套样式作为替补。可以放在代码的顶层，也可以嵌套在任何其他条件组规则中
>
> @supports规则由一组样式声明和一条支持条件构成。支持条件由一条或多条使用逻辑与（and）、逻辑或（or）、逻辑非（not）结合的"名称-值"对。可以使用圆括号调整操作符的优先级

## CSS语法

### 常规语法

```css
@supports (property: value){
  element {
    property: value;
  }
}
```

### 非（not）逻辑声明

```css
@supports not (property: value){
  element {
    property: value;
  }
}
```

### 与（and）逻辑声明

```css
@supports (property: value) and (property: value) {
  element {
    property: value;
  }
}
```

### 或（or）逻辑声明

```css
@supports (property: value) or (property: value) {
  element {
    property: value;
  }
}
```

### 组合声明

```css
@supports ((property: value) or (property: value)) and (property: value) {
  element {
    property: value;
  }
}
```

## JS语法

> 在JavaScript中，使用**window.CSS.supports**来支持CSS的@supports

### 第一种写法

```js
CSS.supports(property, value)
```

### 第二种写法

```js
CSS.supports('property: value');
```

## 实例

> css3中有一个属性aspect-ratio，表示纵横比，但是这个属性不是所有浏览器都支持，那如果想要使用这个属性的效果，如果做兼容呢？

### aspect-radio

> 一般情况下只有某些替换的元素具有长宽比，尤其是图像。对于它们，如果仅指定宽度和高度之一，则可以使用固有长宽比从中计算出另一个。然而对于大多数元素是不具有此性质的，此属性允许为任何其他元素显式指定长宽比，以获得相似的行为

``` css
div {
  aspect-radio: 4 / 3;
}
```

### 之前的解决方案

> 而在没有该属性之前，一般是使用padding-top或者padding-bottom来解决

比如说要实现一个width为200px，而height为0的下方形，应该怎么写:

```css
div {
  wdith: 200px;
  height: 0px;
  padding-top: 100%;
}
```

如果按照上面的写法，那得到的大概率不是一个正方形，而是一个长方形，因为padding-top的百分比是基于父元素的width来计算的，故应该直接指定padding-top的值为200px，才能得到一个正方形

有了这个概念之后，想要实现一个自适应并且固定比例的矩形就不是不可能了，这个方案也被称为“ Padding-Top Hack”。该解决方案需要一个父容器和一个绝对放置的子容器。然后，可以将宽高比计算为百分比以设置为`padding-top`，例如：

- 1：1长宽比= 1/1 = 1 = `padding-top: 100%`
- 4：3长宽比= 3/4 = 0.75 = `padding-top: 75%`
- 3：2长宽比= 2/3 = 0.66666 = `padding-top: 66.67%`
- 16：9长宽比= 9/16 = 0.5625 = `padding-top: 56.25%`

```css
<body>
	<div class="container">
		<img class="media" src="1.jpg">
	</div>
</body>

.container {  
  position: relative;  
  width: 100%;  
  padding-top: 100%; /* 1:1 Aspect Ratio */ 
}  
.media {  
  position: absolute; /* 由于子容易内容全部被pading填充，因此需要绝对定位 */ 
  top: 0; 
}
```

### 兼容两种方案

```scss
@mixin aspect($width: 16, $height: 9) {
  position: relative;
  aspect-ratio: $width / $height;

  @supports not (aspect-ratio: $width / $height) {
    &::before {
      float: left;
      padding-top: calc((#{$height} / #{$width}) * 100%);
      content: '';
    }

    &::after {
      display: block;
      clear: both;
      content: '';
    }
  }
}
```

