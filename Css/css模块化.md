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

## Styled-components

> CSS-in-JS是一种**技术**，而不是一个具体的库实现。简单来说CSS-in-JS就是将应用的CSS样式写在JavaScript文件里面，而不是独立为一些css，scss或less之类的文件，这样你就可以在CSS中使用一些属于JS的诸如模块声明，变量定义，函数调用和条件判断等语言特性来提供灵活的可扩展的样式定义。CSS-in-JS在React社区的热度是最高的，这是因为React本身不会管用户怎么去为组件定义样式的问题，而Vue有属于框架自己的一套定义样式的方案。
>
> styled-components 应该是CSS-in-JS最热门的一个库，通过styled-components，你可以使用ES6的标签模板字符串语法，为需要styled的Component定义一系列CSS属性，当该组件的JS代码被解析执行的时候，styled-components会动态生成一个CSS选择器，并把对应的CSS样式通过style标签的形式插入到head标签里面。动态生成的CSS选择器会有一小段哈希值来保证全局唯一性来避免样式发生冲突。

### **styled-components的原理**

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

### 安装styled-components

```shell
# 安装
$ npm install -S styled-components
```

### 入门示例

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
        // color和size属性不会被传递到DOM中，因为styled-components足够聪明，可以过滤非标准属性
				<ContentContainer color='#00f' size="20" className="aa">
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

加上类型，这样在写JSX时就有提示，防止写错

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

### 扩展样式

> 可以把一个styled组件做为基类，然后用其它的styled组件对其进行扩展：

```js
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;
```

### As属性

> styled组件可以接收一个as属性，它可以更改styled组件所渲染的标签类型，并且适用于自定义组件：

```jsx
const Button = styled.button`
  display: inline-block;
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
  display: block;
`;

const ReversedButton = props => <Button {...props} children={props.children.split('').reverse()} />

render(
  <div>
    <Button as="a" href="#">Link with Button styles</Button>
  </div>
);
```

### 样式化组件

> styled方法不仅可以扩展styled组件，并且还可以扩展自定义组件：

```jsx
const Link = ({ className, children }) => (
  <a className={className}>
    {children}
  </a>
);

const StyledLink = styled(Link)`
  color: palevioletred;
  font-weight: bold;
`;

render(
  <div>
    <Link>Unstyled, boring Link</Link>
    <br />
    <StyledLink>Styled, exciting Link</StyledLink>
  </div>
);
```

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

### &与&&

> 单个＆符号指代该组件的**所有实例**

```tsx
const Thing = styled.div.attrs((/* props */) => ({ tabIndex: 0 }))`
  color: blue;

  &:hover {
    color: red; // <Thing> when hovered
  }

  & ~ & {
    background: tomato; // <Thing> as a sibling of <Thing>, but maybe not directly next to it
  }

  & + & {
    background: lime; // <Thing> next to <Thing>
  }

  &.something {
    background: orange; // <Thing> tagged with an additional CSS class ".something"
  }

  .something-else & {
    border: 1px solid; // <Thing> inside another element labeled ".something-else"
  }
`

render(
  <React.Fragment>
    <Thing>Hello world!</Thing>
    <Thing>How ya doing?</Thing>
    <Thing className="something">The sun is shining...</Thing>
    <div>Pretty nice day today.</div>
    <Thing>Don't you think?</Thing>
    <div className="something-else">
      <Thing>Splendid.</Thing>
    </div>
  </React.Fragment>
)
```

> &&双符号指的是组件的特定某个**实例**

```tsx
const Input = styled.input.attrs({ type: "checkbox" })``;

const Label = styled.label`
  align-items: center;
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
`

const LabelText = styled.span`
  ${(props) => {
    switch (props.$mode) {
      case "dark":
        return css`
          background-color: black;
          color: white;
          ${Input}:checked + && {
            color: blue;
          }
        `;
      default:
        return css`
          background-color: white;
          color: black;
          ${Input}:checked + && {
            color: red;
          }
        `;
    }
  }}
`;

render(
  <React.Fragment>
    <Label>
      <Input defaultChecked />
      <LabelText>Foo</LabelText>
    </Label>
    <Label>
      <Input />
      <LabelText $mode="dark">Foo</LabelText>
    </Label>
    <Label>
      <Input defaultChecked />
      <LabelText>Foo</LabelText>
    </Label>
    <Label>
      <Input defaultChecked />
      <LabelText $mode="dark">Foo</LabelText>
    </Label>
  </React.Fragment>
)
```

> &&具有“优先级提升”的特殊行为，在处理混合着styled组件和普通的 CSS 环境的情况下，他会起到作用

```tsx
const Thing = styled.div`
   && {
     color: blue;
   }
 `

 const GlobalStyle = createGlobalStyle`
   div${Thing} {
     color: red;
   }
 `

 render(
   <React.Fragment>
     <GlobalStyle />
     <Thing>
       I'm blue, da ba dee da ba daa
     </Thing>
   </React.Fragment>
 )
```

### .attrs

> 为了避免重复的将props传递给要render的组件或元素，可以使用`.attrs`构造函数。它可以将传入的props传递到要render的元素或组件上。
>
> 这种方法可以将静态props附加到元素上，或者将activeClassName之类的第三方props传递给React Router的Link组件。
>
> 此外，`.attrs`对象还接受函数，这些函数接收组件接收的props，返回值也将合并到最终的props中。

```tsx
const Input = styled.input.attrs(props => ({
  // we can define static props
  type: "text",

  // or we can define dynamic ones
  size: props.size || "1em",
}))`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;

  /* here we use the dynamically computed prop */
  margin: ${props => props.size};
  padding: ${props => props.size};
`;

render(
  <div>
    <Input placeholder="A small text input" />
    <br />
    <Input placeholder="A bigger text input" size="2em" />
  </div>
);
```

在包装styled组件时，`.attrs`将从最里面的styled组件应用到最外面的styled组件。这就意味着包装器可以重写attrs，类似于后面在样式表中定义的 css 属性如何覆盖以前的声明

```tsx
const Input = styled.input.attrs(props => ({
  type: "text",
  size: props.size || "1em",
}))`
  border: 2px solid palevioletred;
  margin: ${props => props.size};
  padding: ${props => props.size};
`;

// Input's attrs will be applied first, and then this attrs obj
const PasswordInput = styled(Input).attrs({
  type: "password",
})`
  // similarly, border will override Input's border
  border: 2px solid aqua;
`;

render(
  <div>
    <Input placeholder="A bigger text input" size="2em" />
    <br />
    {/* Notice we can still use the size attr from Input */}
    <PasswordInput placeholder="A bigger password input" size="2em" />
  </div>
);
```

### keyframe helper

> 具有@keyframe的CSS动画不限于单个组件，但是如果想要避免命名冲突，可以使用styled-component导出的keyframe helper，它将生成一个实例，可以在整个应用程序中使用:

```tsx
// Create the keyframes
const rotate = keyframes`
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
`;

// Here we create a component that will rotate everything we pass in over two seconds
const Rotate = styled.div`
  display: inline-block;
  animation: ${rotate} 2s linear infinite;
  padding: 2rem 1rem;
  font-size: 1.2rem;
`;

render(
  <Rotate>&lt; 💅🏾 &gt;</Rotate>
);
```

keyframe在使用时会被lazy注入，这就是它们可以被代码分割的原因，所以必须使用`css helper`来处理共享的样式片段:

```tsx
const rotate = keyframes``

// ❌ This will throw an error!
const styles = `
  animation: ${rotate} 2s linear infinite;
`

// ✅ This will work as intended
const styles = css`
  animation: ${rotate} 2s linear infinite;
`
```

### createGlobalStyle helper

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

### css helper

> 用于从带插值的模板字符串生成CSS。当然它只在插值为一个只有styled-component才理解的函数时才需要。如果你要插值一个字符串，则不需要使用这个函数
>
> css helper返回一个插值数组，它是一个拍平的数据结构，可以将其作为插值本身进行传递

```tsx
import styled, { css } from 'styled-components'

const complexMixin = css`
  color: ${props => (props.whiteColor ? 'white' : 'black')};
`

const StyledComp = styled.div`
  /* 嵌套插值 */
  ${props => (props.complex ? complexMixin : 'color: blue;')};
`
```

### theme provider

> Styled-Component 通过导出`<ThemeProvider>`包装组件来提供完整的主题化支持。该组件通过context API为其下面的所有React组件提供一个主题。在render tree中，所有styled组件都可以访问所提供的主题

```tsx
// Define our button, but with the use of props.theme this time
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// We are passing a default theme for Buttons that arent wrapped in the ThemeProvider
Button.defaultProps = {
  theme: {
    main: "palevioletred"
  }
}

// Define what props.theme will look like
const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button>Normal</Button>

    <ThemeProvider theme={theme}>
      <Button>Themed</Button>
    </ThemeProvider>
  </div>
);
```

#### function theme

> theme属性可以是一个函数。此函数将接收父主题接收的theme props。这样主题本身就可以与上下文相联系

```tsx
// Define our button, but with the use of props.theme this time
const Button = styled.button`
  color: ${props => props.theme.fg};
  border: 2px solid ${props => props.theme.fg};
  background: ${props => props.theme.bg};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
`;

// Define our `fg` and `bg` on the theme
const theme = {
  fg: "palevioletred",
  bg: "white"
};

// This theme swaps `fg` and `bg`
const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg
});

render(
  <ThemeProvider theme={theme}>
    <div>
      <Button>Default Theme</Button>

      <ThemeProvider theme={invertTheme}>
        <Button>Inverted Theme</Button>
      </ThemeProvider>
    </div>
  </ThemeProvider>
);
```

#### 在styled component之外的地方使用theme

- 在类组件中使用

```tsx
import { withTheme } from 'styled-components'

class MyComponent extends React.Component {
  render() {
    console.log('Current theme: ', this.props.theme)
    // ...
  }
}

export default withTheme(MyComponent)
```

- 在函数组件中使用

```tsx
import { useContext } from 'react'
import { ThemeContext } from 'styled-components'

const MyComponent = () => {
  const themeContext = useContext(ThemeContext)

  console.log('Current theme: ', themeContext)
  // ...
}
```

- 在自定义hook中使用

```tsx
import { useTheme } from 'styled-components'

const MyComponent = () => {
  const theme = useTheme()

  console.log('Current theme: ', theme)
  // ...
}
```

#### 覆盖theme

```tsx
// Define our button
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// Define what main theme will look like
const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button theme={{ main: "royalblue" }}>Ad hoc theme</Button>
    <ThemeProvider theme={theme}>
      <div>
        <Button>Themed</Button>
        <Button theme={{ main: "darkorange" }}>Overridden</Button>
      </div>
    </ThemeProvider>
  </div>
);
```

