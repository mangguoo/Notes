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

> lighten是sass自带的运算颜色的函数，sass中自带了很多类似这样的函数

```scss
$fontSize: 12px;
$bgColor: red;
body{
    padding:0;
    margin:0;
}
.wrapper{
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

## sass mixin

> sass的mixin函数的定义方式与less有所不同，在sass中，必须通过@mixin的方式显式定义mixin函数。
> 除了定义方式，调用方式也不同，在sass中，mixin函数必须以@include语法来调用mixin函数。

```scss
$fontSize: 12px;
$bgColor: red;
@mixin boxA{
    color:green;
}
@mixin boxB($fontSize){
    font-size: $fontSize;
    border: 1px solid #ccc;
    border-radius: 4px;
}
.box1{
    @include boxB($fontSize + 3px);
    line-height: 2em;
}
.box2{
    @include boxA();
    line-height: 3em;
}
```

```css
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

![image-20221220200449265](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221220200449265.png)

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

