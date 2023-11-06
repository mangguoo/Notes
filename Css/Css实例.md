# Css实例

## 绘制三角形

### 使用border属性实现

> 给定一个宽度和高度都为 0 的元素，其 border 的任何值都会直接相交，我们可以利用这个交点来创建三角形。也就是说，border属性是三角形组成的，下面给每一边都提供不同的边框颜色

```css
.triangle {
    width: 0;
    height: 0;
    border: 100px solid;
    border-color: orangered skyblue gold yellowgreen;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-6027d98cccaf10b137525e1cf33deb22_1440w-20231013112202572.webp" alt="img" style="zoom:50%;" />

有了这个特性，就可以绘制出各种三角形，比如说朝下的三角形(通过只显示上边框实现):

```css
.triangle {
    width: 0;
    height: 0;
    border-top: 50px solid skyblue;
    border-right: 50px solid transparent;
    border-left: 50px solid transparent;
}
```

亦或者说一个等边三角形(左右边框宽度相同):

```css
.triangle {
  width: 0;
  height: 0;
  border-left: 69px solid transparent;  
  border-right: 69px solid transparent;  
  border-bottom: 120px solid skyblue; 
}
```

### 使用linear-gradient实现

> linear-gradient 需要结合 background-image 来实现三角形

```css
.triangle {
  width: 80px;
  height: 100px;
  background-repeat: no-repeat;
  outline: 1px solid skyblue;
  background-image: linear-gradient(45deg, orangered 50%, rgba(255, 255, 255, 0) 50%);
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-df8849851672f8f37f2764d8e2857272_1440w.webp" alt="img" style="zoom:50%;" />

使用linear-gradient的优势就在于，它其实设置的是背景图片，那么它就被背景图片相关的属性控制比如background-size，background-repeat等等。

有了这些属性，就可以更容易的去绘制一个三角形：

```css
.triangle {
  width: 80px;
  height: 100px;
  background-repeat: no-repeat;
  outline: 1px solid skyblue;
  background-image: linear-gradient(32deg, orangered 50%, rgba(255, 255, 255, 0) 50%), linear-gradient(148deg, orangered 50%, rgba(255, 255, 255, 0) 50%);
  background-position: top left, bottom left;
  background-size: 100% 50%;
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-046162d4025db32e98c814441b9558cb_1440w.webp" alt="img" style="zoom:50%;" />

### 使用clip-path绘制三角形

> 它是最精简和最可具扩展性的。不过目前其在浏览器兼容性不是很好，使用时要考虑浏览器是否支持
>
> Clip-path就是在作用元素中绘制多边形（或圆形、椭圆形等）并将其定位在元素内，实际上，浏览器不会绘制clip-path之外的任何区域，因此我们看到的是clip-path的边界

```css
.triangle{
  margin: 100px;
  width: 160px;
   height: 200px;
   background-color: skyblue;
  clip-path: polygon(0 0, 0% 100%, 100% 50%);
}
```

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-332ed2dda5ce998e14e3a6c773f2aacd_1440w.webp" alt="img" style="zoom:50%;" />
