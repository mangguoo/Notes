# React静态方法属性

## React.cloneElement

> 在`react`中有这样一个方法`cloneElement，用于克隆一个元素。以 element 元素为样板克隆并返回新的 React 元素。config 中应包含新的 props，key 或 ref。返回元素的 props 是将新的 props 与原始元素的 props 浅层合并后的结果。新的子元素将取代现有的子元素，如果在 config 中未出现 key 或 ref，那么原始元素的 key 和 ref 将被保留
>
> ```js
> React.cloneElement(
>   element,
>   [config],
>   [...children]
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

## React.DetailedHTMLProps

> 这个一个类型，在我们想要自定义组件的时候会用得到，例如当我们把组件的props定义为这样的类型时，就可以给他传递任意div标签上可以存在的属性

```ts
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

## React.createElement

> JSX 仅仅只是 React.createElement(component, props, ...children) 函数的语法糖

**React.createElement**

React.createElement(component, props, ...childre)返回一个 react 元素对象，包含：

- This tag allows us to uniquely identify this as a React Element

- - $$typeof：Symbol.for('react.element')（暂时只涉及react element）

- Built-in properties that belong on the element

- - type：原生标签类型（h1、p、div等）或者 function（类组件、函数组件等）(还有别的类型，比如React.Fragment、React.Suspense，这里暂不涉及)
  - key：列表中标识元素的唯一性
  - ref：同 [react ref ](https://link.zhihu.com/?target=https%3A//react.docschina.org/docs/refs-and-the-dom.html)属性
  - props：元素的 props 属性（包含 jsx 中写在元素标签上的属性和 children 属性）

## React.Children

> props.children 指向的是当前 react element 的子节点，这个子节点非常灵活，可以是字符串、函数、react element等等
>
> 灵活的 props.children 为组件开发也带来了一定麻烦，比如要遍历子节点、限制子节点的种类和数量（细化到组件类型）、强制使用某一种子节点（比如 Col 组件必须做为 Row 组件的第一层子节点出现），基于这些考虑，React 提供了 React.Children 的一系列方法，方便开发者进行操作

### React.Children.map

`React.Children.map(children, function[(thisArg)])`

在 children 里的每个直接子节点上调用一个函数，并将 this 设置为 thisArg。如果 children 是一个数组，它将被遍历并为数组中的每个子节点调用该函数。如果子节点为 null 或是 undefined，则此方法将返回 null 或是 undefined，而不会返回数组

### React.Children.forEach

`React.Children.forEach(children, function[(thisArg)])`

与 React.Children.map() 类似，但它不会返回一个数组

### React.Children.count

`React.Children.count(children)`

返回 children 中的组件总数量，等同于通过 map 或 forEach 调用回调函数的次数

### React.Children.only

`React.Children.only(children)`

验证 children 是否只有一个子节点（一个 React 元素），如果有则返回它，否则此方法会抛出错误

## **应用**

### **实现自定义触发事件元素**

举个例子，比如 Form 组件，支持触发提交的 Button 自定义，思路是遍历 Form 的 children，找到类型为 Button 的子节点，为此子节点添加 onClick 事件

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

### **组件限制子元素类型及数量**

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

### **组件灵活修改 children 的属性**

```js
const renderCustomElement = () => {
  return React.children.map(child => {
     // do something, for example, add event to props or style etc.
     return React.cloneElement(child, config)
  })
}
```
