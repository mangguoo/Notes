# CSS模块化解决方案

> **在react项目中，可以让样式命名不冲突的方案**
>
> 1、定义样式名称时，就让它唯一
>
> 2、使用内置的cssModule
>
> 3、css-in-js 把css当作js来使用

## CSS Module

> 使用 `.css` 文件
>
> 这种方式可能会导致样式名冲突

```css
// style.css
.home-wrapper{
	background: #f00;
	width: 100%;
	height: 500px;
}
```

```jsx
import React from 'react';
import './style.css'

function Hello(props) {
  return (
    <div className='home-wrapper'></div>
  );
}

export default Hello;
```

> 使用 `.module.css` 文件

```css
// style.module.css
.homeWrapper{
	background: #f00;
	width: 100%;
	height: 500px;
}
```

```jsx
import React from 'react';
import styles from './style.module.css';

function Hello(props) {
  return (
    <div className={styles.homeWrapper}></div>
  );
}

export default Hello;
```

> 当然也可能配合scss来使用css module：

```scss
// style.module.scss
.wrapper {
    color: red;
    .title {
        color: blue;
    }
}
```

下面是打包后的样子，可以看到如果使用scss的话，引用样式就只需要引入一个就可以了（ .wrapper ）

``` css
.style_wrapper__WVUEi {
  color: red;
}
.style_wrapper__WVUEi .style_title__d1hTo {
  color: blue;
}
```

## css-in-js

> CSS-in-JS是一种**技术**，而不是一个具体的库实现。简单来说CSS-in-JS就是将应用的CSS样式写在JavaScript文件里面，而不是独立为一些css，scss或less之类的文件，这样你就可以在CSS中使用一些属于JS的诸如模块声明，变量定义，函数调用和条件判断等语言特性来提供灵活的可扩展的样式定义。CSS-in-JS在React社区的热度是最高的，这是因为React本身不会管用户怎么去为组件定义样式的问题，而Vue有属于框架自己的一套定义样式的方案。
>
> styled-components 应该是CSS-in-JS最热门的一个库，通过styled-components，你可以使用ES6的标签模板字符串语法，为需要styled的Component定义一系列CSS属性，当该组件的JS代码被解析执行的时候，styled-components会动态生成一个CSS选择器，并把对应的CSS样式通过style标签的形式插入到head标签里面。动态生成的CSS选择器会有一小段哈希值来保证全局唯一性来避免样式发生冲突。

### **styled-components原理**

> styled-components就是通过下面这种方式来拼接模板字符串的

```js
const tag = (strings, ...values) => {
  let index = 0
  let str = ''
  values.forEach((item) => {
    str += strings[index] + item
    index++
  })
  return str + strings[index]
}
const aa = '11'
const bb = '22'

const ret = tag`a${aa}b${bb}c` // 函数调用
```

> styled-components把样式当作组件来定义和使用，样式就是组件，组件就是样式。下面这个方法其实是返回了一个组件ContentContainer，它会被渲染为一个div，并且会应用其中包裹的样式。
>
> 这个组件可以接收参数，因此在写样式的时候是可以通过传入的参数来写动态样式的。

```js
export const ContentContainer = styled.div`
  color:${props => props.color || '#888'};
  font-size: ${props => props.size || 12}px;
`
```

```jsx
<ContentContainer color='#00f' size="20">
  我是内容
</ContentContainer>
```

### **styled-components使用**

```js
# 安装
npm install -S styled-components
```

```jsx
/src/components/Child/index.jsx
import React, { Component } from 'react'
import { 
    ChildContainer, 
    TitleContainer, 
    SubTitleContainer, 
    ontentContainer 
} from './style'

class Child extends Component {
  render() {
    return (
      <ChildContainer>
        <div className="title">我是一个child组件</div>
        <SubTitleContainer>我是一个副标题</SubTitleContainer>
        <hr />
		<ContentContainer color='#00f' size="20">
		  我是内容
		</ContentContainer>
      </ChildContainer>
    )
  }
}

export default Child
```

```js
/src/components/Child/style.js
import styled from 'styled-components'

export const ChildContainer = styled.div`
  font-size: 30px;
  color:#f0f;
  .title{
    font-size:18px;
  }
`
export const TitleContainer = styled.div`
  color:red;
  font-size:18px;
`
export const SubTitleContainer = styled(TitleContainer)`
  font-size:14px;
`
export const ContentContainer = styled.div`
  color:${props => props.color || '#888'};
  font-size: ${props => props.size || 12}px;
`
```

### 实现全局样式

```tsx
import type { AppProps } from "next/app";
import { createGlobalStyle } from "styled-components";
import { globalStyle } from "../styles";

const GlobalStyle = createGlobalStyle`
  ${globalStyle}
`;

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <>
      <GlobalStyle />
	  ... ...
    </>
  );
}

export default MyApp;
```

