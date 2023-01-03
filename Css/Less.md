# Less

## 编译工具

- 安装less编译工具：

`npm install less -D`

- 编译less文件：

`npx lessc ct.less>ct.css`

## less嵌套

> 嵌套选择器前面加上&表示同级，这样可以方便我们写伪类选择器

```less
body{
    padding:0;
    margin:0;
}

.wrapper{
    background:white;

    .nav{
        font-size: 12px;
    }
    .content{
        font-size: 14px;
        &:hover{
            background:red;
        }
    }
}
```

```css
body {
  padding: 0;
  margin: 0;
}
.wrapper {
  background: white;
}
.wrapper .nav {
  font-size: 12px;
}
.wrapper .content {
  font-size: 14px;
}
.wrapper .content:hover {
  background: red;
}
```

## less变量

> lighten是less自带的运算颜色的函数，less中自带了很多类似这样的函数

```less
@fontSize: 12px;
@bgColor: red;

body{
    padding:0;
    margin:0;
}

.wrapper{
    background:lighten(@bgColor, 40%);

    .nav{
        font-size: @fontSize;
    }
    .content{
        font-size: @fontSize + 2px;
        &:hover{
            background:@bgColor;
        }
    }
}
```

```css
body {
  padding: 0;
  margin: 0;
}
.wrapper {
  background: #4dff4d;
}
.wrapper .nav {
  font-size: 16px;
}
.wrapper .content {
  font-size: 18px;
}
.wrapper .content:hover {
  background: green;
}
```

## less mixin

> less使用类选择器来定义mixin函数，如果定义mixin函数时指定需要传入参数，那么他就只做为一个mixin函数，而如果定义mixin函数时没有指定需要传入参数，那么他就既可以做为一个mixin函数，又可以做为类选择器

```less
@fontSize: 12px;
@bgColor: red;

.boxA{
    color:green;
}

.boxB(@fontSize){
    font-size: @fontSize;
    border: 1px solid #ccc;
    border-radius: 4px;
}

.box1{
    .boxB(@fontSize + 3px);
    line-height: 2em;
}
.box2{
    .boxA();
    line-height: 3em;
}
```

```css
.boxA {
  color: green;
}
.box1 {
  font-size: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  line-height: 2em;
}
.box2 {
  color: green;
  line-height: 3em;
}
```

## less extend

> extend与mixin的功能相同，都是为了避免开发时写重复代码。但是mixin对于重复的代码采用的解决方案是复制，而extend采用的方案是提取到一起

```less
@fontSize: 12px;
@bgColor: red;

.block{
    font-size: @fontSize;
    border: 1px solid #ccc;
    border-radius: 4px;
}

.wrapper{
    background:lighten(@bgColor, 40%);
    .nav:extend(.block){
        color: #333;
    }
    .content{
        color: #333;
        &:extend(.block);
        &:hover{
            background:red;
        }
    }
}
```

```css
.block,
.wrapper .nav,
.wrapper .content {
  font-size: 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.wrapper {
  background: #ffcccc;
}
.wrapper .nav {
  color: #333;
}
.wrapper .content {
  color: #333;
}
.wrapper .content:hover {
  background: red;
}
```

## less loop

> 可以使用when语法给mixin函数加上一个条件，加上条件之后，下次调用这个mixin函数时，就只有满足条件才会执行。
>
> 这样就可以递归调用这个带条件的mixin函数，从而形成类似于for循环的效果。

```less
.gen-col(@n) when (@n > 0){
    .gen-col(@n - 1);
    .col-@{n}{
        width: 1000px/12*@n;
    }
}

.gen-col(12);
```

```css
.col-1 {
  width: 1000px/12*1;
}
.col-2 {
  width: 1000px/12*2;
}
.col-3 {
  width: 1000px/12*3;
}
.col-4 {
  width: 1000px/12*4;
}
......
.col-11 {
  width: 1000px/12*11;
}
.col-12 {
  width: 1000px/12*12;
}
```

## less import

> 通过@import导入的css文件最后会和当前文件合并到一起，这样就可以把页面中不同模块的css样式分开文件来写，最后再编译合并为一个文件，这样既避免了多个css文件增加http请求，又方便区别不同模块的样式

- **lessImport.less**

```less
@import "./variable";
@import "./module";
```

- **variable.less**

```less
@themeColor: blue;
@fontSize: 14px;
```

- **module.less**

```less
.module{
    .box{
        font-size:@fontSize + 2px;
        color:@themeColor;
    }
    .tips{
        font-size:@fontSize;
        color:lighten(@themeColor, 40%);
    }
}
```

## 预处理器框架

> Lesshat / EST: 这些库提供现成的mixin，就类似于JS类库，封装了一些常用的功能
