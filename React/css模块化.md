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

### module.css

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

### module.scss

> - 如果想在react脚手架中使用scss，必须先安装scss(`npn i scss -D`)
> - 然后就可以创建module.scss文件，在组件中进行使用了

```jsx
import styles from './index.module.scss'
<div className={styles.css类名}></div>
```

**最佳使用方式：**在每个组件的根节点使用CSSModules形式的类名，其他所有的子节点，都使用普通的CSS类名:global

- Index.module.scss

```scss
.app-container {
  position: fixed;
  left: 10px;
  width: 200px;
  height: 200px;
  background-color: red;

  // 此处，使用 global 包裹其他子节点的类名。此时，这些类名就不会被处理，在 JSX 中使用时，就可以用字符串形式的类名
  // 如果不加 :global ，所有类名就必须添加 styles.title 才可以
  /* stylelint-disable-next-line selector-pseudo-class-no-unknown */
  :global {
    .title {
      font-size: 30px;
    }
  }
}
```

```css
.App_app-container__ilamV {
  position: fixed;
  left: 10px;
  width: 200px;
  height: 200px;
  background-color: red;
  /* stylelint-disable-next-line selector-pseudo-class-no-unknown */
}
.App_app-container__ilamV .title {
  font-size: 30px;
}
```

- Component.tsx

```tsx
import styles from './index.module.scss'

const component = () => {
 return (
   {/* （1） 根节点使用 CSSModules 形式的类名*/}
   <div className={styles['app-container']}>
     {/* （2） 所有子节点，都使用普通的 CSS 类名*/}
     <h1 className="title">
       hello world
     </h1>
   </div>
 )
}
```

在ts项目中，引入module.scss可能会报错(<u>找不到模块“./index.module.scss”或其相应的类型声明</u>)，解决方法如下：

- 首先在src同级目录创建`globals.d.ts`,并添加下面代码

```ts
declare module '*.scss';  
```

- 在tsconfig的include中添加`"./**/*.ts"`，和exclude添加`"node_modules/**/*"`

```json
"include": [
    "src",
    "./**/*.ts"
],
"exclude": ["node_modules/**/*"]
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

```shell
# 安装
$ npm install -S styled-components
```

- **/src/components/Child/index.jsx**

```jsx
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

- **/src/components/Child/style.js**

```js
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

- 加上类型，这样在写JSX时就有提示，防止写错

```tsx
interface ColumnStyleProps {
  spanHeight: boolean;
  maxWidth: number;
  center: boolean;
}

const SColumn = styled.div<ColumnStyleProps>`
  position: relative;
  width: 100%;
  height: ${({ spanHeight }) => (spanHeight ? "100%" : "auto")};
  max-width: ${({ maxWidth }) => `${maxWidth}px`};
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: ${({ center }) => (center ? "center" : "flex-start")};
`;
```

### styled高阶组件

```tsx
import React from "react";
import styled from "styled-components";
import * as React from "react";
import * as PropTypes from "prop-types";
import styled, { keyframes } from "styled-components";

const fadeIn = keyframes`
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
`;

interface WrapperStyleProps {
  center: boolean;
}

const SWrapper = styled.div<WrapperStyleProps>`
  will-change: transform, opacity;
  animation: ${fadeIn} 0.7s ease 0s normal 1;
  min-height: 200px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: ${({ center }) => (center ? `center` : `flex-start`)};
`;

interface WrapperProps extends WrapperStyleProps {
  children: React.ReactNode;
}

const Wrapper = (props: WrapperProps) => {
  const { children, center } = props;
  return (
    <SWrapper {...props} center={center}>
      {children}
    </SWrapper>
  );
};

Wrapper.propTypes = {
  children: PropTypes.node.isRequired,
  center: PropTypes.bool,
};

Wrapper.defaultProps = {
  center: false,
};

// 使用styled函数包裹组件，并通过断言规定类型，这样在使用SContent时就有提示了
export const SContent = styled(Wrapper as React.FC<WrapperProps>)`
  width: 100%;
  height: 100%;
  padding: 0 16px;
`;
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

