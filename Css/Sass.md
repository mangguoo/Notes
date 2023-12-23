# sass

## 编译工具

- 安装sass编译工具：

`npm install node-sass -D`

- 编译less文件：

`npx node-sass ct.scss>ct.css`

## sass嵌套

### 选择器嵌套

> 嵌套选择器前面加上&表示同级，这样可以方便我们写伪类选择器

```scss
div {
  li {
    color: white;
  }
  > span {
    color: green;
  }
  + p {
    color: red;
  }
  &:hover {
    color: blue;
  }
}
```

```css
div li {
  color: white;
}
div > span {
  color: green;
}
div + p {
  color: red;
}
div:hover {
  color: blue;
} 
```

### 属性嵌套

```scss
div {
  border: {
    left: {
      width: 100px;
      color: red;
    }
  }
}
```

```css
div {
  border-left-color: red;
  border-left-width: 100px;
}
```

## sass变量

> Sass变量的名字可以使用连字符和下划线
>
> 除了SassScript支持的数据类型之外，任何其他有效的CSS值都可以分配给变量。SassScript支持七种数据类型：
>
> - 数字
> - 带引号和不带引号的字符串
> - 颜色
> - 布尔值
> - 空值
> - list
> - map

```scss
$fontSize: 12px;
$bgColor: red;
body{
    padding:0;
    margin:0;
}
.wrapper{
    // lighten是sass自带的运算颜色的函数，sass中自带了很多类似这样的函数
    background:lighten($bgColor, 40%);
    .nav{
        font-size: $fontSize;
    }
    .content{
        font-size: $fontSize + 2px;
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
  background: #ffcccc;
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

### 变量的作用域

> sass变量是有作用域的。定义在全局，则是全局变量，全局可用。定义在选择器内部，则是本地变量。本地变量只在嵌套的选择内部可用:

```scss
$primary:red;
.link{
  $primary green;
  color:$primary;
  &:hover{
    color:$primary;
  }
}
p{
  color:$primary;
}
```

```scss
.link{
  color:green;
}
.link:hover{
  color:green;
}
p{
  color:red;
}
```

### 变量覆盖

> 在同一个作用域定义同一个变量，变量被覆盖。术语叫重载

```scss
$primary:red;
.link{
  color:$primary
}
$primary:green;
.other-link{
  color:$primary;
}
```

```scss
.link{
  color:red;
}
.other-link{
  color:green;
}
```

### !global关键字

> !global关键字用来提升局部变量的权限，将局部变量提升到全局

```scss
$primary:red;
.link{
    $primary:green !global;
    color:$primary;
}
.other-link{
    color: $primary;
}
```

```scss
.link {
  color: green; 
}

.other-link {
  color: green; 
}
```

### 变量运算

> scss中定义的变量可以进入加减乘除以及模运算

```scss
$color1:blue;
$color2:red;
$distance:10px;
.operations {
    color: $color1 + $color2;
    width: $distance * 100;
    height: ($distance+10) * 5;
    font-family: sans- +'serif';
}
```

```scss
.operations {
  color: magenta;
  width: 1000px;
  height: 100px;
  font-family: sans-serif; 
}
```

### !default关键字

> !default关键字用来定义默认属性，想要覆盖掉默认属性，我们只需要重新定义个相同的属性名。我们可以定义一些默认的变量，然后通过@import指令导入进来，从而让代码的重用行变得更好

```scss
$color:red;
$color:green !default;

.green{
    color: $color;
}
```

```scss
.green {
  color: red; 
}
```

## sass mixin

> sass的mixin函数的定义方式与less有所不同，在sass中，必须通过@mixin的方式显式定义mixin函数。
> 除了定义方式，调用方式也不同，在sass中，mixin函数必须以@include语法来调用mixin函数。

```scss
$bgColor: red;
@mixin boxBase{
    color:green;
}
.box {
    @include boxA();
    line-height: 3em;
}
```

```css
.box {
  color: green;
  line-height: 3em; 
}
```

### mixin传值

```scss
$fontSize: 12px;
@mixin boxBase($fontSize){
    font-size: $fontSize;
    border: 1px solid #ccc;
    border-radius: 4px;
}
.box{
    @include boxBase($fontSize + 3px);
    line-height: 2em;
}
```

```scss
.box {
  font-size: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  line-height: 2em; 
}
```

### @content的使用

```scss
@mixin tablet {
  @media (min-width: #{$tablet-width}) {
    @content;
  }
}

@mixin desktop {
  @media (min-width: #{$desktop-width}) {
    @content;
  }
}

@mixin mobile {
  @media (max-width: #{$tablet-width}) {
    @content;
  }
}
```

```scss
.container {
  @include mobile {
    display: none;
  }
  @include tablet {
    display: none;
  }
  @include desktop {
    display: flex;
  }
}
```

## sass extend

> extend与mixin的功能相同，都是为了避免开发时写重复代码。但是mixin对于重复的代码采用的解决方案是复制，而extend采用的方案是提取到一起。

```scss
$fontSize: 12px;
$bgColor: red;
.block{
    font-size: $fontSize;
    border: 1px solid #ccc;
    border-radius: 4px;
}
.wrapper{
    background:lighten($bgColor, 40%);
    .nav{
        @extend .block;
        color: #333;
    }
    .content{
        @extend .block;
        color: #333;
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

## sass占位符

```scss
%toolbelt {
  box-sizing: border-box;
  border-top: 1px rgba(#000, .12) solid;
  padding: 16px 0;
  width: 100%;

  &:hover { border: 2px rgba(#000, .5) solid; }
}

.action-buttons {
  @extend %toolbelt;
  color: #4285f4;
}

.reset-buttons {
  @extend %toolbelt;
  color: #cddc39;
}
```

上面的代码中用%toolbet定义了一个占位符，首先这个%toolbet的内容不会出现在最终的css输出文件中，但是我们可以定义新的css类，然后使用@extend去扩展这个占位符

与exntend直接拓展一个class name比较,这种拓展方式不会生成class name

## sass function

> @function指令创建的函数不会生成到编译后的css文件中，它只是返回一个值，下面是一个px转rem做个例子

```scss
$browser-context:16;
@function px_to_rem($pixels,$context:$browser-context){
    @return ($pixels/$browser-context)*1rem;
}
h1{
    font-size: px_to_rem(20);
}
```

```scss
h1 {
  font-size: 1.25rem; 
}
```

## sass loop

> 在sass中可以使用和less中类似的方式实现循环，但是这样的写法完全没有必要，因为sass中是有循环语法的。

```scss
@mixin gen-col($n){
    @if $n > 0 {
        @include gen-col($n - 1);
        .col-#{$n}{
            width: 1000px/12*$n;
        }
    }
}

@include gen-col(12);
```

```scss
@for $i from 1 through 12 {
    .col-#{$i} {
        width: 1000px/12*$i;
    }
}
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

## sass import

> 通过@import导入的css文件最后会和当前文件合并到一起，这样就可以把页面中不同模块的css样式分开文件来写，最后再编译合并为一个文件，这样既避免了多个css文件增加http请求，又方便区别不同模块的样式。

- **sassImport.scss**

```scss
@import "./variable";
@import "./module";
```

- **variable.scss**

```scss
$themeColor: blue;
$fontSize: 14px;
```

- **module.scss**

```scss
.module{
    .box{
        font-size:$fontSize + 2px;
        color:$themeColor;
    }
    .tips{
        font-size:$fontSize;
        color:lighten($themeColor, 40%);
    }
}
```

```css
.module .box {
  font-size: 16px;
  color: blue;
}
.module .tips {
  font-size: 14px;
  color: #ccccff;
}
```

## 插值语法

> scss中的插值语法与js中的概念大致相同，无非是写法略有不同

```scss
$name:class;
$direction:left;
$units:px;
.#{name}{
  margin-#{$direction}:20#{units};
}
```

```scss
.class {
  margin-left: 20px; 
}
```

## partial sass文件

> 当在sass文件名前面加上“_”时，除非把它导入到另一个不是partial的sass文件中，否则它不会被生成到CSS中,这是一个用来保持样式被分隔成逻辑部分很好的方法

**来自Sass语言指南：**
您可以创建包含CSS小片段的部分Sass文件，这些CSS小片段可以包含在其他Sass文件中。这是模块化CSS的好方法，有助于使内容更易于维护。部分就是一个以下划线开头的Sass文件。您可以将其命名为_partial.scss。下划线让Sass知道这个文件只是一个部分文件，不应该生成CSS文件。Sass部分文件与@import指令一起使用

**注意:**

在导入partial sass文件的时候,不需要写_

```scss
@import 'variables'
```



## 预处理器框架

>  Compass: 这个库提供现成的mixin，就类似于JS类库，封装了一些常用的功能

# Scss使用技巧

## 设置元素左右间距

> 在我们使用margin-left给元素设置间距的时候，不能给第一个设置，这时候就可以使用一些技巧：通过兄弟选择器选择同classname的兄弟，这样就不会选择到第一个元素

```scss
.footerNavListItem {
  &+.footerNavListItem {
    margin-left: getVw(144);
  }
}
```

## 设置元素上下间距

> 在没有相同classname的情况下，想给元素设置上下间距，就需要给每个需要手动给每个需要设置margin-top的元素设置相同classname，这样很麻烦，我们可以通过编程的方式来简化这个操作，以react为例：

- **index.tsx**

```tsx
import React from "react";
import classNames from "classnames";
import styles from "./style.module.scss";

type IProps = {
  data: {
    ... ...
  }[]
}

const comp = (props:IProps) => {
  return <div>
    {props.data.map((item, index) => {
      return (
        <div 
          className={
            classNames({[styles.marginTop50]: index > 0})
          } 
          key={item.title}>
            ... ...
          </div>
        </div>
      )
    })}
  </div>
}

export default NavMessageModule;
```

- **style.module.scss**

```scss
.marginTop50 {
  margin-top: 60px;
}
```

## px转为vw

```scss
@function getVw($n) {
  @return (100 / (1920 / $n)) + vw;
}
```

## 切换风格

> 下面以一个按钮组件为例

```tsx
import React, { CSSProperties, memo, ReactNode, useCallback } from 'react'
import { useDebounceFn } from 'ahooks'
import classNames from 'classnames'
import styles from './Button.module.scss'
import { useRouter } from 'next/router'
// 这个方法，用于给数字或字符数字加'px'
import { pxUnit } from '../../utils'
import { UrlObject } from 'url'

export interface ButtonProps {
  // 类型
  type?: 'primary' | 'text' | 'default' | 'white' | 'rect'
  // 幽灵按钮
  plain?: boolean
  // 点击事件
  onClick?: () => void
  // 是否禁用
  disabled?: boolean
  // 路由跳转
  to?: UrlObject | string
  // 目标，to 属性存在时生效
  target?: string
  // 是否载入中
  loading?: boolean
  // 大小
  size?: 'big' | 'middle' | 'small'
  // 宽度
  width?: string | number
  // 高度
  height?: string | number
  // 类名
  className?: string
  // children
  children?: ReactNode
  // 节流时间
  throttleTime?: number
  // 是否通栏按钮
  inline?: boolean
  // 按钮样式
  buttonStyle?: CSSProperties
  // 文字颜色
  color?: string
  // 外链
  href?: string
}

const Button = (props: ButtonProps) => {
  const {
    type = 'primary',
    disabled = false,
    plain = false,
    loading = false,
    onClick = () => {},
    throttleTime = 200,
    size = 'middle',
  } = props

  const clickFn = useDebounceFn(onClick, {
    wait: throttleTime,
    leading: true,
    trailing: false,
  })

  const router = useRouter()

  // 处理点击事件
  const handleClick = useCallback(() => {
    if (disabled || loading) {
      return
    }
    if (props.to) {
      router.push(props.to)
      return
    }
    if (props.href) {
      window.open(props.href)
      return
    }
    clickFn.run()
  }, [disabled, loading, props.to, props.href, clickFn, router])

  return (
    <div
      style={{
        width: !props.inline ? pxUnit(props.width) : undefined,
        height: pxUnit(props.height),
        color: props.color,
        ...props.buttonStyle,
      }}
      className={classNames(props.className, styles.button, {
        [styles.buttonBig]: size === 'big',
        [styles.buttonMiddle]: size === 'middle',
        [styles.buttonSmall]: size === 'small',
        [styles.buttonPrimary]: type === 'primary',
        [styles.buttonDefault]: type === 'default',
        [styles.buttonWhite]: type === 'white',
        [styles.buttonRect]: type === 'rect',
        [styles.buttonText]: type === 'text',
        [styles.isPlain]: plain,
        [styles.isDisabled]: disabled,
        [styles.isInline]: props.inline,
      })}
      onClick={handleClick}
    >
      {props.children}
    </div>
  )
}

export default memo(Button)
```

```scss
@import "styles/theme";
@import "styles/common";
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: $border-radius;
  font-size: 14px;
  color: #fff;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.15s;
  user-select: none;
  box-sizing: border-box;
  &+.button {
    margin-left: getVw(24);
  }
  &Big {
    height: getVw(60);
    font-size: getVw(20);
    padding: 0 getVw(36);
    &+.buttonBig {
      margin-left: getVw(32);
    }
  }
  &Middle {
    height: getVw(40);
    padding: 0 getVw(16);
    font-size: getVw(16);
  }
  &Small {
    height: 30px;
    padding: 0 30px;
    font-size: 12px;
  }
  &Default {
    background: #f6f8fb;
    color: $text-1;
  }
  &Primary {
    background: $main;
    &:hover:not(.is-loading):not(.is-disabled) {
      background: $main-hover;
      border-color: transparent;
    }
    &.isPlain {
      color: $main;
      background: transparent;
      border: 1px solid $main;
      &:hover:not(.is-disabled) {
        background: $main;
        color: #fff;
        border-color: transparent;
      }
    }
  }
  &White {
    background: #fff;
    color: $main;
    &:hover {
      background: rgba(#fff,0.9);
    }
    &.isPlain {
      color: #fff;
      background: transparent;
      border: 1px solid #fff;
      &:hover:not(.is-disabled) {
        background: #fff;
        color: $main;
        border-color: transparent;
      }
    }
  }
  &Text {
    color: $text-1;
    padding: 0 4px;
    min-width: 0;
    height: auto;
    &:hover {
      opacity: 0.8;
    }
  }
  &.isDisabled {
    cursor: not-allowed;
    opacity: 0.5;
  }
  &.isInline {
    width: 100%;
  }
}
```

## 巧用nth-type

![image-20221220200449265](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/image-20221220200449265.png)

```scss
.institutionItem {
  padding: getVw(8) getVw(16) getVw(8) getVw(30);
  font-size: getVw(18);
  margin-top: getVw(24);
  border-radius: 4px;
  position: relative;
  &:after {
    content: '';
    width: getVw(8);
    height: getVw(8);
    border-radius: 50%;
    top: 50%;
    transform: translateY(-50%);
    left: getVw(16);
    position: absolute;
  }
  & + .institutionItem {
    margin-left: getVw(28);
  }
  &:nth-of-type(4n - 3) {
    background: rgba($main, 0.06);
    color: $main;
    &:after {
      background: rgba($main, 0.4);
    }
  }
  &:nth-of-type(4n - 2) {
    background: rgba($success, 0.06);
    color: $success;
    &:after {
      background: rgba($success, 0.4);
    }
  }
  &:nth-of-type(4n - 1) {
    background: rgba($info, 0.06);
    color: $info;
    &:after {
      background: rgba($info, 0.4);
    }
  }
  &:nth-of-type(4n) {
    background: rgba($tip, 0.06);
    color: $tip;
    &:after {
      background: rgba($tip, 0.4);
    }
  }
}
```

## 巧用nth-child

> 通过n-2给顶层的两个div加margin-top

```html
<div class="container">
    <div></div>
    <div></div>
    <div></div>
    <div></div>
</div>
```

```scss
.container {
   display: flex;
   width: 100px;
   flex-wrap: wrap;
   &>div {
      width: 50px;
      height: 50px;
      &:nth-child(-n + 2) {
        margin-top: 0;
      }
   } 
}

```

# .module.scss下的特殊标识

## :global

> 在React项目中，样式语言无论是用scss或less，如果想让样式仅作用在某个组件，而不影响全局，一般都会把样式文件进行模块化，即打包后每个class名都会被自动加上一串唯一的序列号
>
> 假如 **想让某个class不加序列号，可以作用到其他组件，那么就写到 :global { ... } 中即可**

```scss
.main {
  width: 100px;
  :global {
    .ant-popover-title{
        color: red;
    }
  }
}
```

```css
.main__3D0Xe{ width: 100px; }
 
.main__3D0Xe .ant-popover-title{
    color: red;
}
```

可以看到，**main后面自动追加了一串唯一的序列号，而ant-popover-title则没有**，这样在main元素中其他任何组件，只要用到ant-popover-title这个class，字体都会变成红色

## :export

> :export用于在.module.scss文件中导出公共变量

- **theme.scss**

```scss
$menuText:#bfcbd9;
$menuActiveText:#409EFF;
$subMenuActiveText:#f4f4f5;
```

如果这些变量是定义在.scss文件中，那么它可以直接被.scss或者.module.scss文件导入并使用：

- **global.scss**

```scss
@import theme.scss
 
body {
	color: $menuText;
}
```

而如果这些变量是定义在.module.scss文件中的，则这些变量必须使用:export关键字导出后，才能在其它地方引用它们：

- **theme.module.scss**

```scss
$menuText:#bfcbd9;
$menuActiveText:#409EFF;
$subMenuActiveText:#f4f4f5;

:export {
  menuText: $menuText;
  menuActiveText: $menuActiveText;
  subMenuActiveText: $subMenuActiveText;
}
```

- **global.scss**

```scss
@import theme.module.scss
 
body {
	color: $menuText;
}
```

- **index.js**

```js
import style from 'theme.module.scss'

console.log(style.menuText)
```

