# React

## TS

### 初始化react为ts环境

```shell
$ npx create-react-app my-app --template typescript
```

用上述命名创建出来的react脚手架，与react+js创建出来的脚手架几乎相同，只是增加了几个文件：

- /src/react-app-env.d.ts：这是一个类型声明文件，与常规ts项目中的/src/types.d.ts文件的功能相同，他就是在ts文件中导入js文件时，ts会在这个文件中查找js代码的类型声明
- /tsconfig.json：这个文件会做为tsc编译器的配置文件（`jsconfig.json`是tsconfig.json的后代，`tsconfig.json`是TypeScript的配置文件。`jsconfig.json`相当于`tsconfig.json`把`"allowJs"`属性设置为`true`）

**注意：**

- 在react+ts工程中，如果是组件或方法有jsx语法，则文件扩展名一定要是.tsx扩展名，否则编译报错（在react+js的项目中，组件.jsx也可以写成.js）
- 在react+ts项目中的所有.ts文件都会被编译为.js文件，然后再进行打包，因此如果要在react+ts项目中备份文件，它的扩展名尽量不要写.ts .tsx，因为项目中虽然没有引用它，但是它仍然会被编译，这会导致编译速度变慢，甚至可以导致报错

### 类组件

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

### 函数组件

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

### ReactNode

> ReactNode可以是一个ReactElement，一个ReactFragment，一个string类型，一个number类型，或者是null，或者是boolean，或者是undefined，或者一个ReactNode的数组

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;

interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;

type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```

### ReactElement

> ReactElement是一个具有type和element的object

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

### JSX.Element

> JSX.Element是一个ReactElement，props和type的泛型类型为any。它存在是因为各种库可以以自己的方式实现JSX，因此JSX是一个global namespace，然后由库设置:

```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> { }
  }
}
```

### DetailedHTMLProps

> 这个一个类型，在我们想要自定义组件的时候会用得到，例如当我们把组件的props定义为这样的类型时，就可以给他传递任意div标签上可以存在的属性

```tsx
interface IProps
  extends React.DetailedHTMLProps<
    React.HTMLAttributes<HTMLDivElement>,
    HTMLDivElement
  > {
  children?: React.ReactNode;
  ... ...
}
  
const CustomComponent = (props: IProps) => {
  ... ...
};
```

**例子：**

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

## React静态方法属性

### React.cloneElement

> 在`react`中有这样一个方法`cloneElement，用于克隆一个元素。以element元素为样板克隆并返回新的React元素。config中应包含新的props，key或ref。返回元素的props是将新的props与原始元素的props浅层合并后的结果。新的子元素将取代现有的子元素，如果在config中未出现key或ref，那么原始元素的key和ref将被保留
>
> ```js
> React.cloneElement(
>  element,
>  [config],
>  [...children]
> )
> ```

**通过这个方法就可以实现传递ref给children**

```jsx
export default (props: IProps) => {
  const { height, children, style, className } = props;
  const childrenRef = useRef(null);
  const getCls = setCompCommonCls("custom", "scroll", "bar");

  useEffect(() => {
    console.log(childrenRef.current);
  }, [childrenRef.current]);

  // 向children传递ref
  const createChildren = useCallback(
    () =>
      React.cloneElement(children, {
        ref: childrenRef,
      }),
    [children]
  );
  return (
    <div
      className={classNames(getCls("container"), className)}
      style={{
        height,
        ...style,
      }}
    >
      {createChildren()}
    </div>
  );
};
```

### 案例(tabs组件)

- **tabs.tsx**

```tsx
import React, { useState, useEffect, useRef, cloneElement, useMemo } from "react";
import type { MouseEvent } from "react";
import { TabsContainer } from "./TabsStyle";
import Tab, { IExpose } from "./Tab";

type IProps = {
  children: React.ReactElement[];
};

function Tabs(props: IProps) {
  const [activeTab, setActiveTab] = useState(0);
  const curTabRef = useRef<IExpose>();
  const prevTabRef = useRef<IExpose>();

  let [tabs, setTabs] = useState(() => {
    return props.children.map((tab, index) => {
      return cloneElement(tab, {
        ref: curTabRef,
      });
    });
  });

  const titles = useMemo(() => {
    return tabs.map((tab: any) => tab.props.title);
  }, [tabs]);

  const switchTabHandler = (evt: MouseEvent, index: number) => {
    setActiveTab(index);
  };

  const addTabHandler = (e: MouseEvent) => {
    setTabs((tabs: any) => {
      return [
        ...tabs,
        React.createElement(Tab, { ref: curTabRef, title: Date.now().toString(16).slice(-3), children: "新内容" }),
      ];
    });
  };

  useEffect(() => {
    curTabRef.current!.activated();
    if (prevTabRef.current) prevTabRef.current.deactivated();
    prevTabRef.current = curTabRef.current;
  }, [activeTab]);

  return (
    <TabsContainer>
      <div className="title">
        {titles.map((title: string, index: number) => (
          <div
            key={title}
            onClick={(evt) => switchTabHandler(evt, index)}
            className={activeTab === index ? "activeTab" : ""}
          >
            {title}
          </div>
        ))}
        <div onClick={addTabHandler}>+</div>
      </div>
      <div className="content">{tabs[activeTab]}</div>
    </TabsContainer>
  );
}

export default Tabs;
```

- **tab.tsx**

```tsx
import React, { forwardRef, useImperativeHandle } from "react";

type IProps = {
  title: string;
  children: (React.ReactElement | string)[] | React.ReactElement | string;
};

function Tab(props: IProps, _ref: any) {
  const activated = () => {
    console.log(props.title + "activated");
  };

  const deactivated = () => {
    console.log(props.title + "deactivated");
  };

  useImperativeHandle(_ref, () => ({
    activated,
    deactivated,
  }));

  return <div>{props.children}</div>;
}

export type IExpose = {
  activated: () => void;
  deactivated: () => void;
};

export default forwardRef(Tab);
```

### React.createElement

> JSX是React.createElement(component, props, ...children)函数的语法糖

**React.createElement**

React.createElement(component, props, ...childre)返回一个 react 元素对象，包含：

- This tag allows us to uniquely identify this as a React Element

- - $$typeof：Symbol.for('react.element')（暂时只涉及react element）

- Built-in properties that belong on the element

- - type：原生标签类型（h1、p、div等）或者 function（类组件、函数组件等）(还有别的类型，比如React.Fragment、React.Suspense，这里暂不涉及)
  - key：列表中标识元素的唯一性
  - ref：同 [react ref ](https://link.zhihu.com/?target=https%3A//react.docschina.org/docs/refs-and-the-dom.html)属性
  - props：元素的 props 属性（包含 jsx 中写在元素标签上的属性和 children 属性）

### React.Children

> props.children 指向的是当前 react element 的子节点，这个子节点非常灵活，可以是字符串、函数、react element等等
>
> 灵活的 props.children 为组件开发也带来了一定麻烦，比如要遍历子节点、限制子节点的种类和数量（细化到组件类型）、强制使用某一种子节点（比如 Col 组件必须做为 Row 组件的第一层子节点出现），基于这些考虑，React 提供了 React.Children 的一系列方法，方便开发者进行操作

#### React.Children.map

`React.Children.map(children, function[(thisArg)])`

在 children 里的每个直接子节点上调用一个函数，并将 this 设置为 thisArg。如果 children 是一个数组，它将被遍历并为数组中的每个子节点调用该函数。如果子节点为 null 或是 undefined，则此方法将返回 null 或是 undefined，而不会返回数组

#### React.Children.forEach

`React.Children.forEach(children, function[(thisArg)])`

与 React.Children.map() 类似，但它不会返回一个数组

#### React.Children.count

`React.Children.count(children)`

返回 children 中的组件总数量，等同于通过 map 或 forEach 调用回调函数的次数

#### React.Children.only

`React.Children.only(children)`

验证 children 是否只有一个子节点（一个 React 元素），如果有则返回它，否则此方法会抛出错误

### 应用

#### 实现自定义触发事件元素

举个例子，比如Form组件，支持触发提交的Button自定义，思路是遍历Form的children，找到类型为Button的子节点，为此子节点添加onClick事件

```js
const renderSubmitButton: (submitButton: any) => any = submitButton => {
  return React.Children.map(props.children, child => {
    if (['boolean', 'undefined', 'string', 'number'].includes(typeof child) || child === null) {
      return child
    }
    if (child.type.name === 'Button') {
      return React.cloneElement(child, {
          onClick: () => {
            if (child.props.onClick === undefined
              || typeof child.props.onClick === 'function'
              && child.props.onClick() !== false
            ) {
              triggerSubmit()
            }
          }
        }) 
    }
    if (child.props.children) {
      return React.cloneElement(
        child,
        {},
        renderSubmitButton(child.props.children)
      )
    }
    return child
  })
}
```

#### 组件限制子元素类型及数量

```js
render() {
  return React.Children.map(this.props.children, child => {
    if (child.type.name !=='Col' || child.type.dispalyName !== 'Col') {
      throw new Error('the children of component Row muse be component Col')
    }
  }) 
}


render() {
  return React.Children.only(this.props.children)
} 
```

#### 组件灵活修改children的属性

```js
const renderCustomElement = () => {
  return React.children.map(child => {
     // do something, for example, add event to props or style etc.
     return React.cloneElement(child, config)
  })
}
```

> 在 React 的世界中，有容器组件和 UI 组件之分，在 React Hooks 出现之前，UI 组件我们可以使用函数组件，无状态组件来展示 UI，而对于容器组件，函数组件就显得无能为力，我们依赖于类组件来获取数据，处理数据，并向下传递参数给 UI 组件进行渲染。React在**v16.8** 的版本中推出了 React Hooks 新特性，Hook是**一套工具函数的集合**,它增强了函数组件的功能，**hook不等于函数组件，所有的hook函数都是以use开头。**
>
> 使用 React Hooks 相比于从前的类组件有以下几点好处：
>
> - 代码可读性更强，原本同一块功能的代码逻辑被拆分在了不同的生命周期函数中，通过 React Hooks 可以将功能代码聚合，方便维护
>
> - 组件树层级变浅，在原本的代码中，我们经常使用 HOC/render/Props 等方式来复用组件的状态，增强功能等，无疑增加了组件树层数及渲染，而在 React Hooks 中，这些功能都可以通过强大的自定义的 Hooks 来实现

**使用hook限制：**

- hook只能用在函数组件中，class组件不行

- 普通函数不能使用hook，默认只能是函数组件才能用

  例外：普通函数名称以**use**开头也可以,(自定义的函数以use开头，称为自定义hook)

- 函数组件内部的函数也不能使用hook

- hook函数一定要放在函数组件的第一层，别放在if/for中(块级作用域)

- 要求函数组件名称必须首字母大写

## React Hooks

### useState

> 保存组件状态，功能类似于类组件中的this.state状态管理

```jsx
import React, { useState } from 'react';

const App = () => {
  // 相当于在App函数组件是定义一个state数据，变量名为count,修改此count的值的方法为setCount
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count + 1)
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

注意：在使用useState的set函数时，尽量不要使用++运算符，因为它有可能导致一些怪异行为：

> 如果这里使用++，会发现点击再次按钮数值才会增加一次，这里因为++是后++，因此第一次setCount的其实还是100，但是count确实是++变成101了。而由于count值没有改变，因此函数组件也不会重新渲染，count通过闭包的形式保存了下来
>
> 在下一次点击时，setCount的值就是101了，而count也变成了102，由于count值发生了变化，因此组件重新渲染，count值也变成了101，这样下次再点击的时候就会重新开始上面的流程

```jsx
import React, { useState } from 'react';

const App = () => {
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count++)
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

**useState并发处理：**

> 如果使用下面这种写法，虽然在addCount方法中进行了三次+1操作，但是实际上count只会被+1，这是因为在setCount后react不会立即更新组件状态，而是会先把它放进一个队列中，而后在合适的时机执行它们，但由于count在这里一直是100（组件状态没有被更新），这就导致其实是重复执行了3次setCount(101)。

```jsx
import React, { useState } from 'react';

const App = () => {
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count + 1)  // 100
    setCount(count + 1)  // 100
    setCount(count + 1)  // 100
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

> 想要解决这个问题，就需要使用回调函数，这样它在每次setCount的时候，就能把当前的count通过回调函数的参数告诉你，从而实现并发操作

```jsx
const addCount = () => {
    setCount(v => v + 1)
    setCount(v => v + 1)
    setCount(v => v + 1)
}
```

**使用useState完成自定义hook：**

- /src/app.js

```jsx
import React from 'react';
import useInput from './hook/useInput';

const App = () => {
  let usernameInput = useInput('')
  let passwordInput = useInput('')

  return (
    <div>
      <input type="text" {...usernameInput} />
      <input type="text" {...passwordInput} />
    </div>
  );
}

export default App;
```

- /src/hooks/useInput.js

```jsx
import { useState } from 'react';

// 在react中，定义的函数是以use开头，则认为它就是一个自定义hook函数
// 在自定义hook函数中，可以调用内置hook
const useInput = (initialValue = '') => {
  let [value, setValue] = useState(initialValue)
  return { value, onChange: e => setValue(e.target.value.trim()) }
}

export default useInput
```

**使用useState获取dom:**

> 通常情况下，我们获取dom的方式是通过useRef，用它获取dom的优势在于useRef不会每次重新渲染都重新获取实例。但是他也有缺点，那就是useRef的值在更新后不会触发更新，因此想要将useRef获取到的元素实例用于传递给其它hook做为参数就变得不可能了。这里就可以使用useState来获取：

```tsx
import React, { type ReactNode, useState } from 'react'
import { usePopper } from 'react-popper'
import styles from './Popper.module.scss'
import ReactDOM from 'react-dom'
import { BrowserView, MobileView } from 'react-device-detect'
import { Alert } from '@/components/common/Alert'
import { type Placement } from '@popperjs/core/lib/enums'

interface PopperProps {
  children: ReactNode | string
  content: string | ReactNode
  placement?: Placement
}

const Popper = (props: PopperProps) => {
  const [referenceElement, setReferenceElement] = useState<HTMLDivElement | null>(null)
  const [popperElement, setPopperElement] = useState<HTMLDivElement | null>(null)
  const [arrowElement, setArrowElement] = useState<HTMLDivElement | null>(null)
  const { styles: popperStyles, attributes } = usePopper(referenceElement, popperElement, {
    placement: props.placement,
    modifiers: [
      { name: 'arrow', options: { element: arrowElement } },
      {
        name: 'offset',
        options: {
          offset: [0, 10]
        }
      }
    ]
  })

  const [visible, setVisible] = useState(true)

  const handleOpenTip = () => {
    Alert({
      content: props.content,
      hasCancel: false
    })
  }

  return (
    <>
      <BrowserView>
        <div
          ref={setReferenceElement}
          onMouseLeave={() => {
            setVisible(true)
          }}
          onMouseEnter={() => {
            setVisible(false)
          }}
          className={styles.popperButton}
        >
          {props.children}
        </div>
        {ReactDOM.createPortal(
          <div
            ref={setPopperElement}
            style={popperStyles.popper}
            {...attributes.popper}
            className={styles.popperContent}
            data-popper-reference-hidden={visible}
          >
            {props.content}
            <div ref={setArrowElement} style={popperStyles.arrow} className={styles.popperArrow} />
          </div>,
          // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
          document.querySelector('#root')!
        )}
      </BrowserView>
      <MobileView>
        <div className={styles.popperButton} onClick={handleOpenTip}>
          {props.children}
        </div>
      </MobileView>
    </>
  )
}

export default Popper
```

### useEffect

> useEffect是副作用钩子：用于处理组件中的副作用，用来取代生命周期函数

```tsx
useEffect(()=>{//副作用函数
    return ()=>{ // 返回函数
    }
},[依赖参数])
```

**useEffect的作用：**

- 挂载阶段：
  从上向下执行函数，如果碰到 useEffect 就执行并将 useEffect 传入的副作用函数推入一个队列(链表)，在组件挂载完成之后，将队列（链表）中的副作用函数执行，并将副作用函数的返还函数，推入一个新的队列（链表）

- 更新阶段不同于其他阶段，这个阶段中对应的函数是否要执行，取决于依赖参数：
  从上向下执行函数，如果碰到 useEffect 就执行并将 useEffect 传入的副作用函数推入一个队列(链表)，在组件更新完成之后，找出之前的返回函数队列，依次准备执行，在执行前会判断该 useEffect 的依赖参数，如果依赖参数改变就执行，否则跳过当前项去看下一项，然后再执行副作用队列，执行时同样判断依赖是否变化，来决定其是否执行，如果执行，就重新获取其对应的返回函数

- 卸载阶段：
  组件即将卸载时，将副作用函数返还函数队列中的函数依次执行

**useEffect的常用情况：**

- 没有依赖参数的情况：

> 这个情况下副作用函数会在组件挂载和组件更新时执行（componentDidMount componentDidUpdate）

```tsx
useEffect(() => {
  console.log('App -- useEffect');
})
```

- 依赖项为空数据：

> 这个情况下他只会执行一次（componentDidMount）

```js
useEffect(() => {
  console.log('App -- useEffect');
}, [])
```

- 传入依赖项：

> 如果对依赖项进行传值，只要依赖项中的值发生改变（componentDidMount componentDidUpdate），就会执行相应的副作用函数（可以传入多个依赖项）

```js
let [count, setCount] = useState({ num: 100 })
const addCount = () => setCount(v => ({ num: v.num + 1 }))

useEffect(() => {
  console.log('App -- useEffect');
}, [count])
```

- 返回一个函数：

> 如果副作用函数中返回一个函数，那个这个函数会在组件卸载时被调用（componentWillUnmout）。
>
> 下面这种写法中，由于它的依赖项是一个空值，所以他只会在挂载时被调用，但是他的返回函数会被收集，在组件卸载时被调用。

```js
useEffect(() => {
  return () => {
    console.log('componentWillUnmout');
  }
}, [])
```

**使用useEffect进行网络请求：**

> 在之前的react版本中，useEffect的副作用回调函数只能返回一个函数，用于模拟生命周期componentWillUnmout
> 因此useEffect的参数不能使用异步函数。因为async函数都会默认返回一个隐式的promise。但是，useEffect不应该返回任何内容。这就是为什么会在控制台日志中看到以下警告：
> Warning: useEffect function must return a cleanup function or nothing. Promises and useEffect(async () => …) are not supported, but you can call an async function inside an effect

- 如果useEffect顶层不支持 async/await可以使用如下的解决方案：

```js
useEffect(() => {
   ;(async function() {
      let ret = await get('/api/swiper')
      setFilms(ret.data)
   })()
}, [])
```

- 发送网络请求：

```jsx
import React, { useState, useEffect } from 'react';
import { get } from '@/utils/http'

const App = () => {
  let [films, setFilms] = useState([])
  useEffect(async () => {
    let ret = await get('/api/swiper')
    setFilms(ret.data)
  }, [])

  return (
     <ul>
        {
          films.map(item => (
            <li key={item.id} > {item.title}</li>
          ))
        }
     </ul>
  );
}

export default App;
```

**useEffect封装自定义hook：**

- /src/hooks/useGoodFood.js

```js
import { useEffect, useState } from 'react';
import { get } from '@/utils/http'

const useGoodFood = (initialValue = []) => {
  let [data, setData] = useState(initialValue)
  let [page, setPage] = useState(1)
  // 暴露请求数据的接口，实现翻页，或数据懒加载
  const loadData = async () => {
    let ret = await get('/api/goodfood?page=' + page)
    if (ret.data.length > 0) {
      // setData(v => [...v, ...ret.data]) // 懒加载
      setData(ret.data) // 翻页
      setPage(v => v + 1)
    }
  }
  // 组件挂载时请求数据，并把请求到的数据(data)暴露出去
  useEffect(() => {
    loadData()
  }, [])
  return [data, loadData]
}

export default useGoodFood
```

- /src/app.js

```jsx
import useGoodFood from "./hooks/useGoodFood";

const App = () => {
  let [data, loadData] = useGoodFood()
  return (
    <div>
      <ul>
        {
          data.map(item => (
            <li key={item.id} > {item.name}</li>
          ))
        }
      </ul>
      <hr />
      <button onClick={() => {
        loadData()
      }}>下一页</button>
    </div >
  );
}

export default App;
```

### useReducer

> useReducer它就是一个小型的redux，它几乎和redux是一样的，也可以管理数据，唯一缺点的就是无法使用redux提供的中间件
>
> 使用hook函数后，一般情况下面，一个组件组中的数据我们可以用useReducer来管理，而不是用redux来完成，redux一般存储公用数据，不需要把所有的数据都存储在里面，redux它是一个单一数据源
>
> useReducer返回一个数组，这个数组有两项，分别是：
>
> - state：它就是初始化后的状态对象
> - dispatch：方法，用来发送指令让reducer来修改state中的数据
>
> useReducer有三个参数：
>
> - reducer，它必须就是一个纯函数，用来修改initState初始化后数据状态函数
>
> - initState，表示state状态的初始化数据
>
> - 第3参数是一个回调函数，要求返回一个对象数据，或直接直接返回一个值，也是表示初始化state的初始化数据
>
>   如果useReducer它有第3个参数，则第2个参数就没有意义，会以第3个参数优先，虽然他们都是初始化数据，但是第3个参数是惰性初始化，可以提升性能（一般用不到）

```jsx
import React, { useReducer } from 'react'

const initState = {
  count: 100
}

const reducer = (state, { type, payload }) => {
  if (type === 'add') {
    return { ...state, count: state.count + payload }
  }
  return state
}

const App = () => {
  const [state, dispatch] = useReducer(reducer, null, () => ({ count: 2000 }))

  const addCount = () => {
    dispatch({ type: 'add', payload: 2 })
  }

  return (
    <div>
      <h3>App组件 -- {state.count}</h3>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

**案例：todoList**

- /reducer.js

```js
export const initState = {
  todos: []
}

export const reducer = (state, { type, payload }) => {
  if ('add' === type) return { ...state, todos: [...state.todos, payload] }
  if ('del' === type) return { ...state, todos: state.todos.filter(item => item.id != payload) }
  return state
}
```

- /index.js

```jsx
import React, { useReducer } from 'react'
import Form from './components/Form'
import List from './components/List'
import { initState, reducer } from './reducer'

const Todo = () => {
  const [{ todos }, dispatch] = useReducer(reducer, initState)
  return (
    <div>
      <Form dispatch={dispatch} />
      <List dispatch={dispatch} todos={todos} />
    </div>
  )
}

export default Todo
```

- /components/List.js

```jsx
import React from 'react'

const List = ({ todos, dispatch }) => {
  const del = id => {
    dispatch({ type: 'del', payload: id })
  }

  return (
    <div>
      <h3>任务项</h3>
      <hr />
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            <span>{item.title}</span>
            <span onClick={() => del(item.id)}>删除</span>
          </li>
        ))}
      </ul>
    </div>
  )
}

export default List
```

- /components/Form.js

```jsx
import React from 'react'

const Form = ({ dispatch }) => {
  // 回车事件
  const onEnter = evt => {
    if (evt.keyCode === 13) {
      const title = evt.target.value.trim()
      // todo每项中的数据
      const todo = {
        id: Date.now(),
        title,
        done: false
      }
      dispatch({ type: 'add', payload: todo })
      evt.target.value = ''
    }
  }

  return (
    <div>
      <input onKeyUp={onEnter} />
    </div>
  )
}

export default Form
```

### useContext

> 在函数组件中，可以使用useContext来获取context中的数据。但是使用它不是必须的，它只是可以方便调用

- /context/appContext.js

```js
import {createContext} from 'react'
const context = createContext()

export const {Provider,Consumer} = context

export default context
```

- /app.jsx

```jsx
import React, { useState, useContext } from "react";
import ctx, { Provider, Consumer } from "./context/appContext";

const Child1 = () => {
  return (
    <div>
      <h3>Child1</h3>
      {/* 不一定非要使用useContext，也可以直接使用组件来调用context中的数据 */}
      <Consumer>{(contextState) => <div>{contextState.name}</div>}</Consumer>
      <hr />
    </div>
  );
};

const Child2 = () => {
  const contextState = useContext(ctx);
  return (
    <div>
      <h3>Child2 --- {contextState.name}</h3>
      <button
        onClick={() => {
          contextState.changeName(Date.now() + "");
        }}
      >
        修改姓名
      </button>
    </div>
  );
};

const App = () => {
  const [name, setName] = useState("张三");
  const changeName = (name) => setName(name);
  return (
    <div>
      <Provider value={{ name, changeName }}>
        <Child1 />
        <Child2 />
      </Provider>
    </div>
  );
};

export default App;
```

### useMemo

> 记忆组件，可以理解为计算属性(它是类组件中的memoizeOne组件是一样的，只不过，useMemo是内置功能，无需安装) 
>
> useMemo它可以对于【数值】进行缓存，如果你的依赖项没有发生改变，则下一次调用时，读取上一次的缓存，从而提升性能
>
> 如果useMemo参数2为一个空数据，则它只会执行一次，后续它所计算的变量的值发生的改变，它也不会重新计算，因为它没有依赖。如果参数2中的数组为多个元素，则其中任意一个元素发生了改变，则它都会重新计算一次

```jsx
import React, { useMemo, useState } from 'react'

const App = () => {
  const [num1, setNum1] = useState(1)
  const [num2, setNum2] = useState(2)

  const sum = useMemo(() => {
    return num1 + num2
  }, [num1, num2])

  return (
    <div>
      <div>{sum}</div>
      <button onClick={() => setNum1(v => v + 1)}>num1+1</button>
      <button onClick={() => setNum2(v => v + 1)}>num2+1</button>
    </div>
  )
}

export default App
```

### useCallback

> 这个hook用来缓存函数。它的第一个参数是一个函数，表示要被缓存的函数。它的第二个参数表示依赖项，如果依赖项是一个空数组，则表示没有依赖项，则可以完全的缓存一个函数，如果有依赖项，则依赖项中的数值发生改变，则它会重新赋值一次函数

```jsx
import React, { useCallback, useState, useEffect } from 'react'

const Child = ({ addCount }) => {
  // 查看addCount是否被更新了（使用了useCallback后只会更新一次）
  useEffect(() => {
    console.log('更新了 addCount')
  }, [addCount])

  return (
    <div>
      <h3>Child组件</h3>
      <button
        onClick={() => {
          addCount(1)
        }}
      >
        修改count值 -- Child组件
      </button>
    </div>
  )
}

const App = () => {
  const [count, setCount] = useState(100)

  // 在没有缓存此函数时，子组件中的按钮被点击，count增加，会导致组件重新渲染
  // 这样App组件重新执行后，它会再次赋值一次
  const addCount = n => {
    console.log('addcount')
    setCount(v => v + n)
  }

  // 但是使用了useCallback后，该函数会被缓存，只要依赖项没有发生变化
  // 那么useCallback不会重新赋值该函数，而是会把上次的函数返回
  const addCount = useCallback(n => {
    setCount(v => v + n)
  }, [])

  return (
    <div>
      <h3>App -- {count}</h3>
      <Child addCount={addCount} />
    </div>
  )
}

export default App
```

### React.memo

> 此方法是一个React顶层Api方法，给函数组件来减少重复渲染，类似于PureComponent父和shouldComponentUpdate方法的集合体。
>
> shouldComponentUpdate必须要有一个返回值，true则表示继续渲染，false停止渲染，而React.memo与它相同，参数2返回值如果为true则表示停止渲染，false继续渲染
>
> 并且参数是一个回调函数，一般情况下都在props传过来的数据为引用类型才需要手动来判断，如果是基本类型则不需要写参数2来手动判断，memo会帮助我们做这个工作，这就类似于PureComponent

```jsx
import React, { useState, memo } from 'react'
import _ from 'lodash'

const Child = memo(
  ({ count }) => {
    console.log('child')
    return (
      <div>
        <h3>child组件 -- {count.n}</h3>
      </div>
    )
  },
  (prevProps, nextProps) => _.isEqual(prevProps, nextProps)
)

const App = () => {
  let [count, setCount] = useState({ n: 100 })
  let [name, setName] = useState('张三')
  return (
    <div>
      <h3>App -- {count.n}</h3>
      <input type="text" value={name} onChange={e => setName(e.target.value)} />
      <button
        onClick={() => {
          setCount({ n: Date.now() })
        }}
      >
        ++++
      </button>
      <Child count={count} />
    </div>
  )
}

export default App
```

### useRef与createRef

> useRef返回一个可变的ref对象，该对象只有个current属性，初始值为传入的参数(initialValue，如果没有传入初始值，则该ref对象的current属性为undefined)
>
> useRef返回的ref对象在组件的整个生命周期内保持不变，当更新current值时并不会reRender，这就是它与useState不同的地方
>
> 更新useRef是side effect (副作用)，所以一般写在useEffec 或event handler里

**useRef与createRef的区别：**

1. createRef可以用在类组件和函数组件中，声明时不能赋初始值

2. useRef只能使用在函数组件中，可以在声明时给初始值

3. createRef每次重新渲染时都会创建一个ref对象，但是在类组件它可以定义在生命周期函数中，所以它一般不会被重新创建

4. useRef第1次渲染时创建之后，并重新渲染时，如果发现这个对象已经存在则不会再创建，它的性能更好一些，在函数组件中推荐使用useRef

**验证useRef不会重复创建：**

> 当button被点击触发点击事件时，count会被改变，从而触发reRender，多次点击时，就会发现，"createRef"会被多次打印，而"useRef"只会被打印一次

```jsx
import React, { createRef, useRef, useEffect, useState } from "react";

const App = () => {
  const usernameCreateRef = createRef();
  const usernameUseRef = useRef();
  const [count, setCount] = useState(100);

  if (!usernameCreateRef.current) {
    console.log("createRef");
    usernameCreateRef.current = count;
  }

  // 如果你第一次创建成功后，第二次使用之前的对象，条件就不成立
  // 以此就可以验证useRef是不是只会创建一次
  if (!usernameUseRef.current) {
    console.log("useRef");
    usernameUseRef.current = count;
  }

  return (
    <div>
      <h3>useState中的count值：{count}</h3>
      <h3>createRef：{usernameCreateRef.current}</h3>
      <h3>useRef：{usernameUseRef.current}</h3>
      <button onClick={() => setCount(v => v + 1)}>+++++++</button>
    </div>
  );
};

export default App;
```

**使用useRef获取dom：**

```jsx
import React, { createRef, useRef, useEffect, useState } from "react";

const App = () => {
  const usernameRef = createRef();
  return (
    <div>
      <input type="text" ref={usernameRef} />
      <button
        onClick={() => {
          console.log(usernameRef.current.value);
        }}
      >
        获取表单项中的数据
      </button>
    </div>
  );
};

export default App;
```

### forwardRef

> 如果ref对象绑定在自定义类组件中，则得到当前自定义类组件的实例，从而可以实现组件通信。但是如果ref直接绑定在自定义函数组件中，则不行，因为函数组件没有实例，所以会报错，因此函数组件如果需要绑定ref，则需要通过react提供的一个顶层api方法来增强函数组件从而实现函数组件也可以绑定ref对象
>
> Rect.forwardRef(render)的返回值是react组件，接收的参数是一个render函数，函数签名为render(props, ref)，第二个参数将其接收的ref属性转发到render返回的组件中

```jsx
import React, { useRef, forwardRef } from 'react'

// 使用forwardRef方法完成对于函数组件中绑定ref对象
const Child = forwardRef((props, _ref) => {
  return (
    <h3 ref={_ref}>child组件</h3>
  )
})

const App = () => {
  let childRef = useRef()
  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => console.log(childRef.current)}>
        ++++
      </button>
    </div>
  )
}

export default App
```

- **ts类型**

```tsx
import React, { forwardRef, useImperativeHandle, type ForwardedRef } from 'react'
import { Input, Form } from 'antd'

interface IProps {
  range: [number, number]
  decimal: number
}

// 导出这个类型，可以在引入组件中当做useRef的泛型参数useRef<TRef>(null)
export interface TRef {
  rules: string[]
}

const HyInput = (props: IProps, _ref: ForwardedRef<TRef>) => {
  const { range, decimal } = props
  const messageApi = useContext(context)
  const rules = []

  useImperativeHandle(_ref, () => {
    return { rules }WalletEditListRef
  })

  return <></>
}

export default forwardRef<TRef, IProps>(HyInput)
```

- 将ref绑定在指定元素上

```tsx
import React, { forwardRef } from 'react'

const CurrentItem = (
  props: IProps,
  ref: React.LegacyRef<HTMLDivElement> | undefined
) => {
  return (
    <div {...props} ref={ref}>
      ... ...
    </div>
  )
}

const CurrentItemRef = forwardRef(CurrentItem)
```

### useImperativeHandle

> 使用它可以透传Ref，因为函数组件没有实例，所以在默认自定义函数组件中不能使用ref属性，使用为了解决此问题，react提供了一个hook和一个高阶组件来帮助函数组件能够使用ref属性
>
> useImperativeHandle有两个参数：
>
> - 参数1：父组件传过来的ref对象
> - 参数2：回调函数，返回一个对象，穿透给父组件的数据

```jsx
import React, { useRef, forwardRef, useState, useImperativeHandle } from 'react'

const Child = forwardRef((props, _ref) => {
  let [name, setName] = useState('张三')
  const setNameFn = name => setName(name)
  const userRef = useRef()

  useImperativeHandle(_ref, () => {
    return { name, setNameFn, userRef }
  })

  return (
    <div>
      <h3>child组件 -- {name}</h3>
      <input type="text" ref={userRef} />
    </div>
  )
})

const App = () => {
  let childRef = useRef()
  return (
    <div>
      <Child ref={childRef} />
      <button
        onClick={() => {
          console.log(childRef.current.name)
          childRef.current.setNameFn(Date.now() + '')
          childRef.current.userRef.current.value = 'abc'
        }}
      >
        ++++
      </button>
    </div>
  )
}

export default App
```

### useLayoutEffect

> useLayoutEffect与useEffect非常类似，他们都接受一个函数和一个数组，只有在数组里面的值改变时才会再次执行effect
>
> useLayoutEffect与useEffect的差异：
>
> - `useEffect`是异步执行的，而`useLayoutEffect`是同步执行的
> - `useEffect`的执行时机是浏览器完成渲染之后，而`useLayoutEffect`的执行时机是浏览器把内容真正渲染到界面之前，和`componentDidMount`等价

**例子：**

> 下面两个例子中，上面使用useEffect来处理副作用，而下面的例子使用useLayoutEffect来处理副作用，从运行的结果中可以直观的看出，使用useEffect的例子中，dom元素有明显的闪烁，而使用的useLayoutEffect的例子中则没有
>
> 这里因为useEffect是渲染完之后异步执行的，所以会导致hello world先被渲染到了屏幕上，再变成world hello，就会出现闪烁现象。而useLayoutEffect是渲染之前同步执行的，所以会等它执行完再渲染上去，就避免了闪烁现象。也就是说我们最好把操作dom的相关操作放到useLayouteEffect中去，避免导致闪烁

```jsx
import React, { useEffect, useLayoutEffect, useState } from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  const [state, setState] = useState("hello world")
  useEffect(() => {
    let i = 0;
    while(i <= 100000000) {
      i++;
    };
    setState("world hello");
  }, []);

  return (
    <>
      <div>{state}</div>
    </>
  );
}

export default App;
```

```jsx
import React, { useEffect, useLayoutEffect, useState } from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  const [state, setState] = useState("hello world")
  useLayoutEffect(() => {
    let i = 0;
    while (i <= 100000000) {
      i++;
    }
    setState("world hello");
  }, []);

  return (
    <>
      <div>{state}</div>
    </>
  );
}

export default App;
```

## setupProxy

> 1. 在src目录下创建一个setupProxy.js文件，react脚手架会读取这个文件，把根据它设置代理
> 2. 安装代理间件模块：npm install -D http-proxy-middleware

- /mock/user.js

```js
module.exports = app => {
  app.get('/api/users', (req, res) => {
    res.send([
      { id: 1, name: '张三' },
      { id: 2, name: '英子' },
    ])
  })
}
```

- /src/setupProxy.js

```js
const { createProxyMiddleware: proxy } = require('http-proxy-middleware')

const userMockFn = require('../mock/user')

module.exports = app => {
  userMockFn(app) // mock接口
    
  app.use('/api', proxy({
    target: 'https://api.iynn.cn/film',
    changeOrigin: true,
    pathRewrite: {
      "^/api": ""
    }
  }))
}
```

## React补充知识

### 组件的key属性

> react中的key属性，它是一个特殊的属性，它是出现不是给开发者用的，而是给React自己使用，有了key属性后，就可以与组件建立了一种对应关系，react利用key来识别组件，他是一种身份标识。每个key 对应一个组件，相同的key react认为是同一个组件，这样后续相同的key对应组件都不会被创建。在render函数执行的时候，新旧两个虚拟DOM会进行对比，如果两个元素有不同的key，那么在前后两次渲染中就会被认为是不同的元素，这时候旧的那个元素会被销毁，新的元素会被创建。如果提供了唯一的标识key且是相同的key，且元素类型相同， 若元素属性有所变化，则React只更新组件对应的属性，这种情况下，性能开销会相对较小

利用这个特性，我们就可以强制重新渲染某个组件，原理就是给组件传入一个与上次不同的key，下面这个组件用于显示一个食品列表，列表到达底部时就会调用父组件传给他的loadData方法，在这个方法中会加载新的数据，然后通过data再传给他，从而触发他的更新，但是如果不想让他更新，而是让他重新渲染，就可以每次给他一个不同的Key，比如说页码，这样就会触发他的重新创建。

```jsx
<GoodFood 
  key={'goodfood_' + this.state.page} 
  data={this.state.goodfood} 
  loadData={this.loadData} 
/>
```

### 组件懒加载

#### Suspense

> Suspense的中文意思是悬而不定，顾名思义指的是数据加载完成之前页面呈现的内容
> 在Suspense内的任何子组件（数据）只要还在pending或者初始状态，则显示的是fallback的内容。等所有的数据加载完成才显示子组件，避免多次更新

#### lazy

> 我们都知道使用webpack打包后会将所有的资源打包到一个入口文件中，这样可以减少请求的资源数量
>
> 当然我们可以使用mini-css-extract-plugin插件将CSS单独分割出来。但是所有的项目打包到一个js文件难免造成体积过大加载缓慢的情况
>
> 使用React.lazy可以将组件打包到单独的chunk文件中，并且实现按需加载。比如在加载首页时只加载首页所需要的资源文件，这样就可以提升加载速度
>
> React.lazy是一个函数。它接收一个Promise，该Promise需要resolve一个defalut export的React组件，因此刚好可以和import()完美配合

#### 案例展示

```jsx
import React, { lazy, Suspense, Fragment } from "react";
import { Switch, Route } from "react-router-dom";
import Admin from "@/views/Admin";

const Login = lazy(() => import(/* webpackChunkName: 'login' */ "@/views/Login"));

const RouterView = () => {
  return (
    <Fragment>
      <Suspense fallback={<h3>加载中...</h3>}>
        <Switch>
          <Route path="/login" component={Login} />
          <Route path="/" component={Admin} />
        </Switch>
      </Suspense>
    </Fragment>
  );
};

export default RouterView;
```

### esModuleInterop标识

#### 问题引入

> JS引入react是这样的：
>
> ```js
> import React from 'react'
> ```
>
> 而TS却是这样的：
>
> ```ts
> import * as React from 'react'
> ```
>
> 如果直接在TS里改成JS一样的写法，在安装了@types/react的情况下，编辑器会抛出一个错误，根据提示，在tsconfig.json中设置 compilerOptions.esModuleInterop为true，报错就消失了

要搞清楚这个问题的原因，首先需要知道JS的模块系统。常用的JS的模块系统有三个：

- CommonJS
- ES module
- UMD

- AMD

babel、TS等编译器更加偏爱cjs。默认情况下，代码里写的esm都会被babel、TS转成cjs。这个原因有以下几点：

1. cjs出现得比esm更早，所以已有大量的npm库是基于cjs的（数量远高于 esm），比如react
2. cjs有着非常成熟、流行、使用率高的runtime：Node.js，而esm的runtime目前支持非常有限（浏览器端需要高级浏览器，node需要一些稀奇古怪的配置和修改文件后缀名）
3. 有很多npm库是基于UMD的，UMD兼容cjs，但因为esm是静态的，UMD无法兼容esm

打开react库的index.js，：

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/v2-13b243bb852d872316ef4f2d7fd73e51_720w.webp" alt="v2-13b243bb852d872316ef4f2d7fd73e51_720w" style="zoom: 67%;" />

可以看到react是基于cjs的，相当于：

```js
module.exports = {
  Children: Children,
  Component: Component
}
```

而在index.ts中，写一段：

```js
import React from "react";
console.log(React);
```

默认情况下，经过tsc编译后的代码为：

```js
"use strict";
exports.__esModule = true;
var react_1 = require("react");
console.log(react_1["default"]);
```

显然，打印出来的结果为undefined，因为react的module.exports中根本就没有default和这个属性。所以后续获取React.createElement、React.Component自然都会报错

这个问题引申出来的问题其实是，目前已有的大量的第三方库大多都是用UMD/cjs写的（或者说，使用的是他们编译之后的产物，而编译之后的产物一般都为cjs），但现在前端代码基本上都是用esm来写，所以esm与cjs需要一套规则来兼容

- esm导入esm

- - 两边都会被转为cjs
  - 严格按照esm的标准写，一般不会出现问题

- esm导入cjs

- - 引用第三方库时最常见，比如react
  - 兼容问题的产生是因为esm有default这个概念，而cjs没有。任何导出的变量在cjs看来都是module.exports这个对象上的属性，esm的default导出也只是cjs上的module.exports.default属性而已
  - 导入方esm会被转为cjs

- cjs导入esm （一般不会这样使用）

- cjs导入cjs

- - 不会被编译器处理
  - 严格按照cjs的标准写，不会出现问题

#### TS默认编译规则

> TS对于import变量的转译规则为：

```js
// 对于 import 导入默认导出的模块，TS 在读这个模块的时候会去读取上面的 default 属性 
import React from 'react'; // before
console.log(React)
var React = require('react'); // after
console.log(React['default'])

// 对于 import 导入非默认导出的变量，TS 会去读这个模块上面对应的属性
import {Component} from 'react'; // before
console.log(Component);
var React = require('react'); // after
console.log(React.Component)
 
// 对于 import *，TS 会直接读该模块
import * as React from 'react'; // before 
console.log(React);
var React = require('react'); // after
console.log(React);
```

> TS、babel对export变量的转译规则为：（代码经过简化）

```js
// 对于 export default 的变量，TS 会将其放在 module.exports 的 default 属性上
// 对于 export 的变量，TS 会将其放在 module.exports 对应变量名的属性上
export const name = "esm";  // before
export default {
   name: "esm default",
};

// 额外给 module.exports 增加一个 __esModule: true 的属性，用来告诉编译器，这本来是一个 esm 模块
exports.__esModule = true;  // after
exports.name = "esm";
exports["default"] = {
   name: "esm default"
}
```

#### TS开启esModuleInterop后的编译规则

> esModuleInterop这个属性默认为false。改成true之后，TS对于import的转译规则会发生一些变化（export的规则不会变）：

```js
 // before
 import React from 'react';
 console.log(React);
 // after（代码经过简化）
 var react = __importDefault(require('react'));
 console.log(react['default']);

 // before
 import {Component} from 'react';
 console.log(Component);
 // after（代码经过简化）
 var react = require('react');
 console.log(react.Component);
 
 // before
 import * as React from 'react';
 console.log(React);
 // after（代码经过简化）
 var react = _importStar(require('react'));
 console.log(react);
```

可以看到，对于默认导入和namespace（*）导入，TS使用了两个helper函数来帮忙（代码经过简化）

```js
// 如果目标模块是 esm，就直接返回目标模块；否则将目标模块挂在一个对象的 defalut 上，返回该对象
var __importDefault = function (mod) {
  return mod && mod.__esModule ? mod : { default: mod };
};

// 如果目标模块是 esm，就直接返回目标模块。否则将目标模块上所有的导出属性（除了default）挪到 result 上，最后将目标模块自己挂到 result.default 上
var __importStar = function (mod) {
  if (mod && mod.__esModule) {
    return mod;
  }

  var result = {};
  for (var k in mod) {
    if (k !== "default" && mod.hasOwnProperty(k)) {
      result[k] = mod[k]
    }
  }
  result["default"] = mod;

  return result;
};
```

### React Hooks心智模型

> $React Hooks心智模型：必须按顺序、不能在条件语句中调用的规则$
>
> react hooks有一个限制，不要在循环，条件或嵌套函数中调用Hook，确保总是在React函数的最顶层以及任何return之前调用他们。遵守这条规则，就能确保Hook在每一次渲染中都按照同样的顺序被调用。这让React能够在多次的`useState`和`useEffect`调用之间保持hook状态的正确

#### useState的工作原理

> useState hook背后的思想是，你可以使用hook函数返回的数组的第二个数组项作为setter函数，并且该setter将控制由hook管理的状态。下面以一个例子来理解useState：

```jsx
function RenderFunctionComponent() {
    const [firstName, setFirstName] = useState("Rudi");
    const [lastName, setLastName] = useState("Yardley");
 
    return (
        <Button onClick={() => setFirstName("Fred")}>Fred</Button>
    );
}
```

#### useState的具体实现

- **初始化：**创建两个空数组：`setters`和`state`，并将cursor设置为0

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111013758-191510416.png" alt="img" style="zoom: 67%;" />

- **第一次渲染：**首次运行组件函数，每次useState()调用，在第一次运行时，都会将一个setter函数推送到setters数组上，然后将一些状态推送到state数组上

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111040565-1891855337.png" alt="img" style="zoom:67%;" />

- **后续渲染：**每次后续渲染都会重置cursor，并且仅从每个数组中读取这些值

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111057882-1329150053.png" alt="img" style="zoom:67%;" />

- **事件处理：**每个setter都有对其cursor的引用，因此通过触发对setter的调用，setter它将更改状态数组中该位置的状态值

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111131036-1171682237.png" alt="img" style="zoom:67%;" />

#### 简单代码实现

```jsx
const state = [];
const setters = [];
let cursor = 0;
 
function createSetter(cursor) {
    return function setterWithCursor(newVal) {
        state[cursor] = newVal;
    };
}
 
export function useState(initVal) {
    if (state[cursor] === undefined && setters[cursor] === undefined) {
        state.push(initVal);
        setters.push(createSetter(cursor));
    }
 
    const setter = setters[cursor];
    const value = state[cursor];
 
    cursor++;
    return [value, setter];
}
 
function RenderFunctionComponent() {
    const [firstName, setFirstName] = useState('Rudi'); // cursor: 0
    const [lastName, setLastName] = useState('Yardley'); // cursor: 1
 
    return (
        <div>
            <button onClick={() => setFirstName('Richard')}>Richard</button>
            <button onClick={() => setLastName('Fred')}>Fred</button>
        </div>
    );
}
 
function MyComponent() {
    cursor = 0; // resetting the cursor
    return <RenderFunctionComponent />; // render
}
 
console.log(state); // Pre-render: []
MyComponent();
console.log(state); // First-render: ['Rudi', 'Yardley']
MyComponent();
console.log(state); // Subsequent-render: ['Rudi', 'Yardley']
 
// click the 'Richard' button
console.log(state); // After-click: ['Richard', 'Yardley']
```

#### 为什么顺序很重要

> 现在，如果我们根据某些外部因素甚至组件状态更改渲染周期的钩子顺序会发生什么？

```jsx
let firstRender = true;
 
function RenderFunctionComponent() {
  let initName;
 
  if(firstRender){
    [initName] = useState("Rudi");
    firstRender = false;
  }
  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");
 
  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```

- **Bad Component第一次渲染**

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111310884-1555363583.png" alt="img" style="zoom:67%;" />

- **Bad Component第二次渲染：**`firstName`和`lastName`发生了错位，我们的状态存储变得不一致了。这就是为什么保持正确顺序的重要性

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/532971-20210729111330977-325690002.png" alt="img" style="zoom:67%;" />

### 闭包陷阱

#### useState的闭包陷阱

> 在函数组件中，如果我们在回调函数中使用了state的值，那么闭包就会产生。闭包在函数创建时产生，他会缓存创建时的state的值

```jsx
import React, { useState } from "react";
const LikeButton: React.FC = () => {
  const [like, setLike] = useState(0);
  function handleAlertClick() {
    setTimeout(() => {
      alert(`you clicked on ${like}`);
      //形成闭包，所以弹出来的是当时触发函数时的like值
    }, 3000);
  }
  return (
    <>
      <button onClick={() => setLike(like + 1)}>{like}赞</button>
      <button onClick={handleAlertClick}>Alert</button>
    </>
  );
};
export default LikeButton;
```

在like为6的时候, 点击alert, 再继续增加like到10, 弹出的值为6, 而非10。当我们更改状态的时候，React会重新渲染组件，每次的渲染都会拿到独立的like值，并重新定义个handleAlertClick函数，每个handleAlertClick函数体里的like值也是它自己的，所以当like为6时，点击alert，触发了handleAlertClick，此时的like是6，哪怕后面继续更改like到10，但alert时的like已经定下来了

![7de34cc101cdde7088a15dec436d7659](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/7de34cc101cdde7088a15dec436d7659.gif)

**解决方法**

- **采用全局变量：**

```jsx
import React from "react";
let like = 0;
const LikeButton: React.FC = () => {
  function handleAlertClick() {
    setTimeout(() => {
      alert(`you clicked on ${like}`);
    }, 3000);
  }
  return (
    <>
      <button
        onClick={() => {
          like = ++like;
        }}
      >
        {like}赞
      </button>
      <button onClick={handleAlertClick}>Alert</button>
    </>
  );
};
export default LikeButton;
```

由于like变量是定义在组件外，所以不同渲染间是可以共用该变量,所以3秒后获取的like值就是最新的like值。但是改变like并不会触发组件重新渲染，因此看起来like一直是0

![c8706c7482867dde33f0bb23ce4695b3](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/c8706c7482867dde33f0bb23ce4695b3.gif)

- **使用useRef：**

```jsx
import React, { useRef } from "react";
const LikeButton: React.FC = () => {
  // 定义一个实例变量
  let like = useRef(0);
  function handleAlertClick() {
    setTimeout(() => {
      alert(`you clicked on ${like.current}`);
    }, 3000);
  }
  return (
    <>
      <button
        onClick={() => {
          like.current = like.current + 1;
        }}
      >
        {like.current}赞
      </button>
      <button onClick={handleAlertClick}>Alert</button>
    </>
  );
};
export default LikeButton;
```

采用useRef,作为组件实例的变量，保证获取到的数据肯定是最新的。但是useRef中current更改并不会触发re-render。但是useRef是定义在实例基础上的，如果代码中有多个相同的组件，每个组件的ref只跟组件本身有关，跟其他组件的ref没有关系

![c8706c7482867dde33f0bb23ce4695b3](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/c8706c7482867dde33f0bb23ce4695b3.gif)

#### useEffect的闭包陷阱

```jsx
import { useEffect, useState } from 'react';

const App = () => {
    const [count,setCount] = useState(0);

    useEffect(() => {
        setInterval(() => {
            setCount(count + 1);
        }, 500);
    }, []);

    useEffect(() => {
        setInterval(() => {
            console.log(count);
        }, 500);
    }, []);

    return <div>count: {count}</div>;
}

export default App;
```

上面组件的打印结果一直是1。这是因为useEffect的第二个参数为空数组，所以只会在组件加载后仅执行一次，我们知道组件每次render的时候都会生成一个新的state对象，对应一个快照，上述代码中，因为useEffect只执行了一次，所以定时器中的`count`一直是最初快照里的`count`，那么页面中`count`的显示肯定不会改变。闭包陷阱产生的原因其实就是 useEffect的函数里引用了某个state，形成了闭包

**解决方法**

- **解法一：**

> 使用useEffect的第二个参数，count变化时，重新执行`setInterval`，并且在useEffect的清理函数中执行`clearInterval`，这样我们就可以在页面上看到变化的count了

```jsx
import { useEffect, useState } from 'react';

const App = () => {
    const [count,setCount] = useState(0);

    useEffect(() => {
      const timer = setInterval(() => {
        setCount(count + 1);
      }, 1000);
      return () => clearInterval(timer)
    }, [count]);

    useEffect(() => {
      const timer = setInterval(() => {
        console.log(count);
      }, 1000);
      return () => clearInterval(timer)
    }, [count]);

    return <div>count: {count}</div>;
}

export default App;
```

但是这种方法有一定的缺点，因为每次count变了都要重置定制器，这样可能会导致计时不准确。所以这种把依赖的state添加到deps里的方式是能解决闭包陷阱，但是定时器不能这样做

- **解法二：**

> 最主要的是`setCount(count => count +1)`，使用函数作为参数，接受一个旧的state，得到新的state
> 使用`useRef`来保存回调函数，在`useEffect`中从`ref.current`来取函数再调用，在`useLayoutEffect`中给`ref`赋值新的fn，这个fn里的state是最新的

```jsx
import { useEffect, useLayoutEffect, useRef } from "react";

const App = () => {
  const [count, setCount] = useState(0);

  const fn = () => {
    console.log(count);
  };

  const ref = useRef(() => {});

  useEffect(() => {
    // 最关键的一步，使用函数，接受一个旧的state，得到新的state
    setInterval(() => {
      setCount((count) => count + 1);
    }, 1000);
  }, []);

  // 每次在render前都给ref赋值新的fn，这个fn里的state是最新值
  // 这里是利用了useLayoutEffect在useEffect之前执行的特性
  useLayoutEffect(() => {
    ref.current = fn;
  });

  useEffect(() => {
    // 如果直接使用fn的话，那相当于一直使用同一个闭包，这个闭包中的count是固定的，一直是第一次创建时赋值的0，所有需要每次更新时执行新的函数
    setInterval(() => ref.current(), 1000);
  }, []);

  return <div>count: {count}</div>;
};

export default App;
```

**以上这个代码可以封装成`useInterval`(自定义hook)：**

```js
import { useEffect, useLayoutEffect, useRef } from 'react';

const useInterval = (fn: Function, delay: number)=>{
    const ref = useRef<Function>(()=>{})

    useLayoutEffect(()=>{
        ref.current = fn
    })

    useEffect(()=>{
        setInterval(()=>{
            ref.current()
        }, delay)
    }, [])
}

export default useInterval
```

```js
import useInterval from './useInterval';

const App = () => {
    const [count,setCount] = useState(0);
    useInterval(()=>{
        setCount(count => count+1)
    }, 1000)
    useInterval(()=>{
        console.log(count, 'count')
    }, 1000)
    
    return <div>count: {count}</div>;
}

export default App;
```

**扩展知识**

- 使用`useEffect`时，若有多个副作用，则应该调用多个`useEffect`，而不是写在一个里面
- `useEffect`第一个参数可以返回一个函数，这个函数会在组件卸载时（也就是render了，生成新的快照时）执行，可以用来清除副作用里的操作
- `useLayoutEffect`是在render前同步执行的（和`componentDidMount`等价），`useEffect`是在render后异步执行的

### React18的useEffect会执行两次

> 这是 React18 才新增的特性。它仅在开发模式("development")下，且使用了严格模式("Strict Mode")下会触发。生产环境("production")模式下和原来一样，仅执行一次
> react之所以要把useEffect的回调执行两次，是因为react其实是在组件挂载后，还 "额外"模拟执行了一次组件的卸载和挂载。这样可以帮助开发者提前发现重复挂载造成的 Bug 的代码
> 同时，这也是为了以后 React的新功能做铺垫。未来会给 React 增加一个特性，允许 React 在保留状态的同时，能够做到仅仅对UI部分的添加和删除。为了让开发者能够提前习惯和适应，做到组件的卸载和重新挂载之后，重复执行 useEffect 的时候不会影响应用正常运行

#### 如何应对

> 对于这个问题，官方文档上面有一句原话：正确的问题不是“怎么样让 Effect 执行一次”，而是“怎样修复我的 Effect，让它在(重复)挂载之后正常工作”
>
> 毕竟在 React 的未来版本中做离屏渲染的时候 useEffect 肯定会多次执行的。而且，即使是当前版本，在做页面的前进后退也会面临触发多次 useEffect。所以，解决办法其实就是重复挂载卸载之后使得应用能够正常工作

每次组件渲染时，React 都会更新页面 UI，然后运行 useEffect 中的代码，Effect 在屏幕更新之后的 rendering 进程结束的时候执行。因此，对于某些“副作用”的渲染，比如异步接口请求，事件绑定等操作我们通常都放在 useEffect 中执行

useEffect 支持返回一个函数，在组件卸载的时候就会执行该函数。因此正确解法就是在useEffect返回的函数中实现清理副作用

- **清理事件监听**

```js
useEffect(() => {
  function handleScroll(e) {
    console.log(e.clientX, e.clientY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

- **重置页面数据**

```js
// 清除属性状态：对于一些页面属性的变更，在返回函数内部将其变更的属性进行还原
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

```jsx
// 涉及到元素状态的，比如播放器之类，需要对（元素）播放器的状态进行重置
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

```js
// 如果是默认弹窗类，这种也算是元素状态，同样需要对其（弹出）状态进行重置
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

- **处理异步数据请求**

在useEffect中进行异步请求时，由于react会在开发环境中额外模拟一次组件的卸载和重新挂载，因此第一次异步执行网络请求时，组件其实已经卸载了，再通过网络请求的数据进行更改组件状态是会报错的

因此这里在返回函数中把ignore设为true，这样第一次执行的时候就不会执行setTodos(json)了，它其实是在第二次执行的时候才会触发

并且这里使用useRef对网络请求到的数据进行了缓存，下次请求的时候如果已经有缓存数据了就直接用，无须再次发起请求

```js
const cache = useRef(null);
useEffect(() => {
  let ignore = false;
  async function startFetching() {
    if (!cache.current) {
      cache.current = await fetchTodos(userId);
    }
    if (!ignore) {
      setTodos(cache.current);
    }
  }
  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

## react18新特性

### 入口文件的不同

**react17**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

// 这个方式不支持Concurrent模式
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
)
```

**react18**

```jsx
import React from "react";
// react18它引入ReactDOM类的位置发生改变
import ReactDOM from "react-dom/client";
import App from "./App";

// 把虚拟dom挂载到真实dom中的方法也发生了改变 由原来的render方法，变为 createRoot(dom节点).render(<App/>)
// 支持Concurrent模式[批处理，让setState都为异步] -- 提升性能
ReactDOM.createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <Router>
      <App />
    </Router>
  </Provider>
);
```

### setState优化

> 在react18之后，setState都为异步，无论写在什么样的语法环境中

**自动批处理：**

批处理是指React将多个状态更新，聚合到一次render中执行，以提升性能

```ts
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React只会re-render一次，这就是批处理
}
```

在React18之前，React只会在事件回调中使用批处理，而在Promise、setTimeout、原生事件等场景下，是不能使用批处理的

```ts
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 会 render 两次，每次 state 变化更新一次
}, 1000);
```

而在React18中，所有的状态更新，都会自动使用批处理，不关心场景

```ts
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会 re-render 一次，这就是批处理
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会 re-render 一次，这就是批处理
}, 1000);
```

如果你在某种场景下不想使用批处理，你可以通过`flushSync`来强制同步执行（比如：你需要在状态更新后，立刻读取新DOM上的数据等）

```ts
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React 更新一次 DOM
  flushSync(() => {
    setFlag(f => !f);
  });
  // React 更新一次 DOM
}
```

### Concurrent Mode

> Concurrent Mode（以下简称 CM）翻译叫并发模式，它是一个底层设计，它使React能够同时准备多个版本的 UI

<img src="https://img-blog.csdnimg.cn/img_convert/531364eef7025d6d303ba2b975dd2890.png" alt="531364eef7025d6d303ba2b975dd2890.png" style="zoom:67%;" />

在以前，React在状态变更后，会开始准备虚拟DOM，然后渲染真实DOM，整个流程是串行的。一旦开始触发更新，只能等流程完全结束，期间是无法中断的

<img src="https://img-blog.csdnimg.cn/img_convert/a7da13e8ca0ce1d683cc73d79be75e34.png" alt="a7da13e8ca0ce1d683cc73d79be75e34.png" style="zoom:50%;" />

在CM模式下，React在执行过程中，每执行一个Fiber，都会看看有没有更高优先级的更新，如果有，则当前低优先级的的更新会被暂停，待高优先级任务执行完之后，再继续执行或重新执行

CM模式有点类似计算机的多任务处理，处理器在同时进行的应用程序之间快速切换，也许React应该改名叫ReactOS了

这里举个例子：我们正在看电影，这时候门铃响了，我们要去开门拿快递。在React 18以前，一旦我们开始看电影，就不能被终止，必须等电影看完之后，才会去开门。而在React18 CM模式之后，我们就可以暂停电影，等开门拿完快递之后，再重新继续看电影

不过对于普通开发者来说，我们一般是不会感知到CM的存在的，在升级到React 18之后，我们的项目不会有任何变化

我们需要关注的是基于CM实现的上层功能，比如Suspense、Transitions、streaming server rendering（流式服务端渲染）等等

### Suspense增强

> Suspense在react18中支持异步组件（可以用它来完成异步数据获取时，组件的等待切换效果），以下是Suspense处理异步的基本流程：
>
> - 在组件中写入一个异步请求请求数据，我们要根据异步请求的结果进行渲染
>
> - 异步请求完成前，我们可以抛出一个错误，这个错误应该是一个promise
> - 如果react发现我们抛出了一个promise，则它会等待这个promise解决
>
> - 当这个promise解决后，react会重新渲染这个组件
>
> ==渲染组件->发现有异步请求->悬停，等待异步请求结果->再渲染展示数据==

**旧写法：**

> 使用条件渲染的方法，在没有数据时就显示加载中，当获取到数据，状态被修改触发组件重新渲染时，就可以正确的把数据渲染出来。

```jsx
import React, { useEffect, useState } from 'react'
import User from './pages/User'

const fetchUser = async () => {
  let ret = await (await fetch('/users.json')).json()
  return ret
  // [
  //   { "id": 1, "name": "张三" },
  //   { "id": 2, "name": "英子" },
  //   { "id": 3, "name": "乐乐" }
  // ]
}

const App = () => {
  let [data, setData] = useState([])

  useEffect(() => {
    fetchUser().then(ret => setData(ret))
  }, [])

  return (
    <div>
      {/* 条件渲染 */}
      {data.length == 0 ? <div>加载中...</div> : <User data={data} />}
    </div>
  )
}

export default App
```

```jsx
import React from 'react'

const User = ({ data }) => {
  return (
    <div>
      <ul>
        {data.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  )
}

export default User
```

**新写法：**

```jsx
import React, { Suspense, useEffect, useState } from "react";
import User from "./pages/User";

// 网络请求
const fetchUser = async () => {
  let ret = await (await fetch("/users.json")).json();
  return ret;
};

// 创建一个用于解析promise中的数据
const wrapperPromise = (promise) => {
  let status = "pending";
  let result;
  const callbackPromise = promise.then(
    (ret) => {
      status = "success";
      result = ret;
    },
    (err) => {
      status = "error";
      result = err;
    }
  );

  return {
    read() {
      if (status === "pending") {
        // 抛一个异常，这样它就会再来执行，然后就会有上一次的结果
        throw callbackPromise;
      } else if (status === "success") {
        return result;
      } else if (status === "error") {
        return result;
      }
    },
  };
};

const App = () => {
  let [data, setData] = useState(wrapperPromise(fetchUser()));

  return (
    <div>
      <Suspense fallback={<div>加载中...</div>}>
        <User users={data} />
      </Suspense>
    </div>
  );
};

export default App;
```

```jsx
import React from 'react'

const User = ({ users }) => {
  // 通过此方法把promise中的数据读取出来,如果请求未完成则会抛出一个错误，会导致这个组件进入悬停状态，请求完成后，这个组件会被重新渲染
  let data = users.read()
  return (
    <div>
      <ul>
        {data.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  )
}

export default User
```

### useTransition

> 我们如果要主动发挥CM的优势，那就离不开startTransition
>
> React的状态更新可以分为两类：
>
> - 紧急更新（Urgent updates）：比如打字、点击、拖动等，需要立即响应的行为，如果不立即响应会给人很卡，或者出问题了的感觉
> - 过渡更新（Transition updates）：将UI从一个视图过渡到另一个视图。不需要即时响应，有些延迟是可以接受的
>
> 而CM模式不会自动帮我们区分不同优先级的更新，它只是提供了可中断的能力，默认情况下，所有的更新都是紧急更新

举个例子：搜索引擎的关键词联想。一般来说，对于用户在输入框中输入都希望是实时更新的，如果此时联想词比较多同时也要实时更新的话，这就可能会导致用户的输入会卡顿。这样一来用户的体验会变差，这并不是我们想要的结果

我们将这个场景的状态更新提取出来：一个是用户输入的更新；一个是联想词的更新。这个两个更新紧急程度显然前者大于后者

以前我们可以使用防抖的操作来过滤不必要的更新，但防抖有一个弊端，当我们长时间的持续输入（时间间隔小于防抖设置的时间），页面就会长时间都不到响应。而startTransition可以指定UI的渲染优先级，哪些需要实时更新，哪些需要延迟更新

useTransition接受传入一个毫秒的参数用来修改最迟更新时间，返回一个过渡期的pending状态和startTransition函数

```jsx
import React, { useState, useTransition } from 'react';
 
export default function Demo() {
  const [value, setValue] = useState('');
  const [searchQuery, setSearchQuery] = useState([]);
  const [loading, startTransition] = useTransition(2000);
 
  const handleChange = (e) => {
    setValue(e.target.value); // 立即更新
    startTransition(() => { // 延迟更新
      setSearchQuery(Array(20000).fill(e.target.value));
    });
  };
 
  return (
    <div className="App">
      <input value={value} onChange={handleChange} />
      {loading ? (
        <p>loading...</p>
      ) : (
        searchQuery.map((item, index) => <p key={index}>{item}</p>)
      )}
    </div>
  );
}
```

### useDeferredValue

> 这个钩子可以允许用户推迟屏幕更新优先级不高部分。如果某些渲染比较消耗性能，比如存在实时计算和反馈，我们可以使用这个`Hook`降低其计算的优先级，使得避免整个应用变得卡顿。

```js
import { useDeferredValue } from 'react';

const deferredValue = useDeferredValue(value, { timeoutMs: <some value> })
```

**案例：**

```jsx
import React, { useState, useDeferredValue } from 'react'
import Search from './pages/Search'

const App = () => {
  let [kw, setKw] = useState('')
  let deferredValue = useDeferredValue(kw, {timeoutMs: 300})

  return (
    <div>
      <h3>{kw}</h3>
      <input value={kw} onChange={e => setKw(e.target.value.trim())} />
      <Search kw={deferredValue} />
    </div>
  )
}

export default App
```

```JSX
import React from 'react'

const Search = ({ kw }) => {
  // 模拟有很多搜索结果  
  const data = Array(1000)
    .fill('')
    .map((_, index) => {
      return '搜索 -- ' + index + ' -- ' + kw
    })

  return (
    <div>
      <ul>
        {data.map(item => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  )
}

export default Search
```



