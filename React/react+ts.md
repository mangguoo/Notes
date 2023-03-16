## react+ts

### 初始化react为ts环境：

```shell
$ npx create-react-app my-app --template typescript
```

用上述命名创建出来的react脚手架，与react+js创建出来的脚手架几乎相同，只是增加了几个文件：

- /src/react-app-env.d.ts：这是一个类型声明文件，与常规ts项目中的/src/types.d.ts文件的功能相同，他就是在ts文件中导入js文件时，ts会在这个文件中查找js代码的类型声明
- /tsconfig.json：这个文件会做为tsc编译器的配置文件（`jsconfig.json`是tsconfig.json的后代，`tsconfig.json` 是 TypeScript 的配置文件。`jsconfig.json`相当于`tsconfig.json`把`"allowJs"`属性设置为`true`）

**注意：**

- 在react+ts工程中，如果是组件或方法有jsx语法，则文件扩展名一定要是.tsx扩展名，否则编译报错（在react+js的项目中，组件.jsx也可以写成.js）
- 在react+ts项目中的所有.ts文件都会被编译为.js文件，然后再进行打包，因此如果要在react+ts项目中备份文件，它的扩展名尽量不要写.ts .tsx，因为项目中虽然没有引用它，但是它仍然会被编译，这会导致编译速度变慢，甚至可以导致报错

### tsx写类组件：

> 组件的children属性就是在双标签组件中写的内容，可以给显示注解ReactElement或any类型。也可以使用react提供的一个泛型类型PropsWithChildren对props的类型进行包裹，这个泛型类型会对传入的类型添加children属性的类型。

```tsx
import React, { Component, ReactElement, PropsWithChildren } from 'react'

// 给props添加自定义属性的类型
interface IProps {
  num: number
  setNum: () => void
  // children: ReactElement
}

// 通过泛型给当前属性添加自定义属性的props类型，相当于在react+js中的prop-types定义
// 今后在react+ts项目中，如果有自定义属性的存在， 请一定要提前在定义组件时把属性的类型指定好
// class Child extends Component<IProps> {
class Child extends Component<PropsWithChildren<IProps>> {
  render() {
    return (
      <div>
        <h3>Child组件 -- {this.props.num}</h3>
        <button onClick={() => this.props.setNum()}>+++</button>
        <hr />
        {this.props.children}
      </div>
    )
  }
}

export default Child
```

```tsx
import React, { Component } from 'react'
import Child from './components/Child'

interface IState {
  num: number
}
interface IProps {}

// state类型可以用泛型传入或断言指定，可以分情况来决定怎么写
// 比如这个组件中没有要接收的props，那么就可以使用断言的方式，不然没有props也得给它定义类型来占位
class App extends Component<IProps, IState> {
  // 断言来指定当前成员属性它的对象的类型
  // state = {
  //   num: 100
  // } as IState

  state = {
    num: 100
  }

  setNum = () => this.setState(state => ({ num: state.num + 1 }))

  render() {
    return (
      <div>
        <h3>App类组件</h3>
        <Child num={this.state.num} setNum={this.setNum}>
          我是child中的内容
        </Child>
      </div>
    )
  }
}

export default App
```

### tsx写函数组件：

```tsx
import React, { FC, ReactNode, PropsWithChildren } from 'react'

// interface IProps {
//   num: number
//   setNum: (n: number) => void
//   // children: ReactNode  // 使用ReactNode来注解children
// }

// 使用PropsWithChildren来包裹props类型，用于获取children的类型
type IProps = PropsWithChildren<{ 
  num: number
  setNum: (n: number) => void
}>

// 方案1：直接指定props类型，缺点就是无法使用解构语法
// const Child = (props: IProps) => {
// 方案2：使用react提供的FS泛型类型来注解组件
const Child: FC<IProps> = ({ num, setNum, children }) => {
  return (
    <div>
      <h3>Child组件 -- {num}</h3>
      <button onClick={() => setNum(100)}>++++</button>
      {children}
    </div>
  )
}

export default Child
```

```tsx
import React, { useState } from 'react'
import Child from './components/Child'

const App = () => {
  // useState它会自动推导出对应的初始类型，无须指定
  const [num, setNum] = useState(100)
  const addNum = (n: number) => setNum(v => v + n)

  return (
    <div>
      <Child num={num} setNum={addNum}>
        <h1>插槽内容</h1>
      </Child>
    </div>
  )
}

export default App
```

### forwardRef

```tsx
import React, { forwardRef, useImperativeHandle, type Ref } from 'react'
import { Input, Form } from 'antd'

interface IProps {
  range: [number, number]
  decimal: number
}

// 导出这个类型，可以在引入组件中当做useRef的泛型参数useRef<TRef>(null)
export interface TRef {
  rules: string[]
}

const HyInput = (props: IProps, _ref: Ref<TRef>) => {
  const { range, decimal } = props
  const messageApi = useContext(context)

  const rules = []

  useImperativeHandle(_ref, () => {
    return { rules }
  })

  return (
    <></>
  )
}

export default forwardRef<TRef, IProps>(HyInput)
```

### 其它类型

#### ReactNode

> ReactNode可以是一个ReactElement，一个ReactFragment，一个string类型，一个number类型，或者是null，或者是boolean，或者是undefined，或者一个ReactNode的数组

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;

interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;

type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```

#### ReactElement

> ReactElement 是一个具有type和element的object

```ts
type Key = string | number

type JSXElementConstructor<P> =
    | ((props: P) => ReactElement | null)
    | (new (props: P) => Component<P, any>);

interface ReactElement<P = any, T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>> {
    type: T;
    props: P;
    key: Key | null;
}
```

#### JSX.Element

> JSX.Element 是一个 ReactElement，props和type的泛型类型为 any。它存在是因为各种库可以以自己的方式实现 JSX，因此 JSX 是一个global namespace，然后由库设置:

```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> { }
  }
}
```

#### DetailedHTMLProps

> 继承这个类型就相当于可以让组件后接收原生标签可以接收的属性

```tsx
interface DINTextProps
  extends React.DetailedHTMLProps<
    React.HTMLAttributes<HTMLDivElement>,
    HTMLDivElement
  > {
  children?: React.ReactNode;
  fontWeight?: 100 | 200 | 300 | 400;
}

const DINText = (props: DINTextProps) => {
  const { className = [], fontWeight = 100 } = props;
  const fontFamily = useMemo(() => {
    const weightMap = {
      100: "DINPro-Regular",
      200: "DINPro-Medium",
      300: "DINPro-Bold",
      400: "DINPro-Black",
    };
    return weightMap[fontWeight];
  }, [fontWeight]);

  return (
    <div
      {...props}
      className={classNames(styles.gDinproText, className)}
      style={{ fontFamily, ...props.style }}
    >
      {props.children}
    </div>
  );
};
```
