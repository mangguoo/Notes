# React入门

## 初体验：

> 1. 引入2个核心库
>    - react.js 创建虚拟dom和提供react开发操作的若干API方法 -- 跨平台
>    - react-dom.js 把虚拟dom转为真实的html的dom对象，渲染到视图节点中
> 2. 在页面中定义挂载的节点  id="root"
> 3. 定义要渲染的组件(虚拟dom)  React.createElement
> 4. 调用ReactDOM.render方法来把组件转为真实的dom，渲染到页面挂载点中

## react组件分类：

> - 类组件：又叫状态组件 、 容器组件
> - 函组件：又叫无状态组件、ui组件

## 组件定义:

### **函数组件**

> 如果安装了jsx插件，则可以通过 【rfc】 完成创建；
>
> 注意事项：
>
> 1. 函数名称，首字母大写
> 2. 必须要有jsx/null返回值，如果是jsx则一定要有顶层元素包裹

``` jsx
import { Component } from 'react'

const App = () => {
  return (
    <div>
      <h3>我是一个App函数组件</h3>
    </div>
  )
}

export default App
```

### **类组件**

> 如果安装了jsx插件，则可以通过 【rcc】来创建类组件
>
> 1. 要以es6的定义类来定义 ，必须以class定义类
> 2. 此类必须要继承父类[Component]
> 3. 此类必须要有一个成员方法 render，此方法中必须要返回jsx/null,如果是jsx必须要有顶层元素

```jsx
import { Component } from 'react'

class App extends Component {
  render() {
    return (
      <div>
        <h3>App之类组件</h3>
      </div>
    )
  }
}

export default App
```

## 组件传参：

### **函数组件**

> 通过自定义属性，可以向子组件传递数据 ，此数据它是单向数据流，子组件不能直接修改
>
> 子组件通过入参 props来接收父组件传过来的数据，props它是一个对象

```jsx
import { Component } from 'react'

const Child = ({ title = '默认值', fn }) => {
  console.log(fn())
  return (
    <div>
      <h3>Child组件 -- {title}</h3>
    </div>
  )
}

const App = () => {
  const fn = () => 'fn'
  return (
    <div>
      <Child title="hello" num={100} obj={{ id: 1 }} fn={fn} />
    </div>
  )
}

export default App
```

### **类组件**

> 类组件中，子组件它是通过成员属性this.props来接受父组件的传值

```jsx
import { Component } from 'react'

class Child extends Component {
  render() {
    let { title = '默认值', fn } = this.props
    console.log(fn())
    return (
      <div>
        <h3>Child组件 -- {title}</h3>
      </div>
    )
  }
}

class App extends Component {
  // 柯理化，在react中很常见
  fn = arg => () => 'fn@@@ -- ' + arg
  render() {
    return (
      <div>
        <h3>App</h3>
        <Child title="我是标题" fn={this.fn(200)} />
      </div>
    )
  }
}

export default App
```

## 事件绑定：

### **函数组件**

```jsx
import React, { Component } from 'react'

// 函数组件中没有this指向问题
const App = () => {
  let num = 1
  const clickFn1 = () => {
    console.log(num++)
  }
  const clickFn2 = () => () => {
    console.log(num++)
  }
  return (
    <div>
      {/* 普通的值，它不具备响应式，所以你修改数据后，视图是不会更新的 */}
      <h3>App组件 -- {num}</h3>
      {/* 一般情况下，绑定的实现方法是不会加小括号，如果要加小括号，则定义绑定方法时，这个方法应该返回一个函数 */}
      <button onClick={clickFn1}>++++</button>
      <button onClick={clickFn2()}>++++</button>
    </div>
  )
}

export default App
```

### **类组件**

```jsx
import React, { Component } from 'react'

class App extends Component {
  num = 100
  todos = []

  // 建议使用 箭头函数定义成员属性，因为这个方法被绑定给了button的click事件，如果使用function定义的函数，那么它的this一定会指向全局，而如果使用箭头函数，由于箭头没有this指向，因此它会向上去找this，从而指向正确的组件对象
  addNum = () => {
    // 同步修改成员属性中的数据，但是视图不更新
    this.num++
    // 类组件中提供一个方法，可以强制让视图更新
    this.forceUpdate()
  }

  // react会在没有写小括号的时候会主动的把event对象传给你
  onEnter = evt => {
    if (evt.keyCode === 13) {
      this.todos.push({
        id: Date.now(),
        title: evt.target.value.trim(),
        done: false
      })
      evt.target.value = ''
      this.forceUpdate()
    }
  }

  render() {
    return (
      <div>
        <h3>{this.num}</h3>
        <ul>
          {this.todos.map(item => (
            <li key={item.id}>{item.title}</li>
          ))}
        </ul>
        {/* 类组件中绑定方法，有this指向问题 */}
        <button onClick={this.addNum}>++++</button>
        <input type="text" onKeyUp={this.onEnter} />
      </div>
    )
  }
}

export default App
```

## $$typeof：

```jsx
<marquee bgcolor="#ffa7c4">hi</marquee>
```

jsx语法其实是一种语法糖，其中的每一个自定义标签都会被编译为一个React对象，通过一个叫createEement的方法。

```jsx
React.createElement(
  /* type */ 'marquee',
  /* props */ { bgcolor: '#ffa7c4' },
  /* children */ 'hi'
)
```

通过这个方法创建出来的React对象大概长这样：

```jsx
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

可是看到这个方法创建出来的React对象中有一个`$$typeof`属性，这个属性是干什么的呢？其实它是用于防止`XSS`攻击的。

如果你的服务器有允许用户存储任意 `JSON` 对象的漏洞，而前端需要一个字符串，这可能会发生一个问题，那就是用户可以写一个React对象，并写一些危险代码存储到服务器中，这样这个React对象就有可能被渲染到每一个访问这个网站的用户界面上，这就是`XSS`攻击。

因此React在迭代版本后给React元素中加入一个`$$typeof`属性，里面存储了一个`Symbol`值，因为JSON不支持 `Symbol` 类型。所以即使服务器存在用`JSON`作为文本返回安全漏洞，`JSON` 里也不包含`Symbol.for('react.element')`。而React 在渲染时这个元素中是否存在 `element.$$typeof`，如果没有，则会拒绝处理该元素，从而防止了`XSS`攻击。

# 基础语法

## React事件处理（合成事件）：

### **合成事件：**

React基于浏览器的事件机制自身实现了一套事件机制，包括事件注册、事件的合成、事件冒泡、事件派发等，在React中这套事件机制被称之为合成事件。**React合成事件**是React **模拟原生DOM事件所有能力** 的一个事件对象。根据 ***W3C规范*** 来定义合成事件，**兼容所有浏览器**，拥有与浏览器原声事件相同的接口。合成事件拥有和浏览器原生事件相同的接口，包括`stopPropagetion()`和`preventDefault()`。

在React中，所有事件都是合成的，不是原生DOM事件，可以通过 `e.nativeEvent` 属性获取原生DOM事件。合成事件不会映射到原生事件，例如：在`onMouseLeave`事件中`event.nativeEvent`将指向`mouseout`事件；

```js
const handleClick = (e) => console.log(e.nativeEvent);;
const button = <button onClick={handleClick}>Leo 按钮</button>
```

与原生事件直接在元素上注册的方式不同的是，React的合成事件不会直接绑定到目标DOM节点上，用事件委托机制，以队列的方式，从触发事件的组件向父组件回溯，直到Root节点。因此，React组件上声明的事件最终绑定到了Root对象（React17之前是Document）上。在Root节点，用一个统一的监听器去监听，这个监听器上保存着目标节点与事件对象的映射。当组件挂载或卸载时，只需在这个统一的事件监听器上插入或删除对应对象；当事件发生时（即Root上的事件处理函数被执行），在映射里按照冒泡或捕获的路径去组件中收集真正的事件处理函数，然后，由这个统一的事件监听器对所收集的事件逐一执行。

#### 在ts中注解事件对象：

```ts
import React, { KeyboardEvent } from "react";

const UserAdd = () => {
  const onEnter = (evt: KeyboardEvent) => {
    if ("Enter" === evt.key) {
      console.log((evt.target as HTMLInputElement).value.trim());
    }
  };

  return (
    <div>
      <div>
        <input type="text" onKeyUp={onEnter} />
      </div>
    </div>
  );
};

export default UserAdd;
```

### **合成事件的优势：**

- 对事件进行归类，可以在事件产生的任务上包含不同的优先级

- 提供合成事件对象，抹平浏览器的兼容性差异

- React引入**事件池**，在事件池中获取或释放事件对象，React事件对象不会被释放掉，而是**存入一个数组**中，当事件触发，就从这个数组中弹出，避免频繁地创建和销毁（不需要注册那么多的事件了，一种事件类型只在 Root上注册一次）

```jsx
export default class extends React.Component {
   fn = (num, evt) => {
      console.log(num, evt);
   }

    render(){
        return (
            <div>
                {* 手动传递合成事件的事件对象 *} 
                <button onClick={evt => this.fn(200, evt)}>click</button>
            </div>
        )
    }
}
```

### **合成事件与原生事件的区别：**

- 命名方式不同

  - 原生：onclick（纯小写）
  - React：onClick（小驼峰）

- 事件处理函数写法不同

  原生事件处理函数为**字符串**，React JSX语法中，传入一个**函数**作为事件处理函数

- 阻止默认行为方式的不同

  - 原生事件：通过返回`false`或显式`preventDefault()`来阻止默认行为
  - React：只能使用`preventDefault()`方法阻止

- 阻止事件泡的方式不同

- 支持的事件不同

  在react中，如果需要注册捕获阶段的事件处理函数，应该为事件名添加`Capture`，例如：处理捕获阶段的点击事件请使用 `onClickCapture` 而不是 `onClick`

### **合成事件与原生事件的执行顺序：**

> 在 React 中，合成事件会以事件委托方式绑定在Root对象上。这里我们手写一个简单示例来观察 React 事件和原生事件的执行顺序：

```js
import React from "react";
class App extends React.Component {
  parentRef;
  childRef;
  constructor(props) {
    super(props);
    this.parentRef = React.createRef();
    this.childRef = React.createRef();
  }
  componentDidMount() {
    document.addEventListener("click", (e) => {
        console.log("原生事件：document DOM 事件捕获---");
    }, true);
    this.parentRef.current?.addEventListener("click", () => {
        console.log("原生事件：父元素 DOM 事件捕获---");
    }, true);
    this.childRef.current?.addEventListener("click", () => {
        console.log("原生事件：子元素 DOM 事件捕获---");
    }, true);
    document.addEventListener("click", (e) => {
        console.log("原生事件：document DOM 事件冒泡！");
    }, false);
    this.parentRef.current?.addEventListener("click", () => {
        console.log("原生事件：父元素 DOM 事件冒泡！");
    }, false);
    this.childRef.current?.addEventListener("click", () => {
        console.log("原生事件：子元素 DOM 事件冒泡！");
    }, false);
  }
  parentClickFun = () => {
    console.log("React 事件：父元素事件冒泡！");
  };
  parentCapture = () => {
    console.log("React 事件：父元素事件捕获-----");
  };
  childClickFun = () => {
    console.log("React 事件：子元素事件冒泡！");
  };
  childCapture = () => {
    console.log("React 事件：子元素事件捕获-----");
  };
  render() {
    return (
      <div ref={this.parentRef} onClick={this.parentClickFun} onClickCapture={this.parentCapture}>
        <p ref={this.childRef} onClick={this.childClickFun} onClickCapture={this.childCapture}>
          分析事件执行顺序
        </p>
      </div>
    );
  }
}

export default App;
```

```js
原生事件：document DOM 事件捕获---
React 事件：父元素事件捕获-----
React 事件：子元素事件捕获-----
原生事件：父元素 DOM 事件捕获---
原生事件：子元素 DOM 事件捕获---
原生事件：子元素 DOM 事件冒泡！
原生事件：父元素 DOM 事件冒泡！
React 事件：子元素事件冒泡！
React 事件：父元素事件冒泡！
原生事件：document DOM 事件冒泡！
```

**通过上面流程，我们可以总结得出：**

- React 所有合成事件都委托在 root 对象上；

- 当真实 DOM 元素触发事件，会冒泡到 root 对象后，再处理 React 事件；

- 在捕获阶段，先注册的先执行，且React合成事件先于原生事件执行；冒泡阶段，先注册的后执行，且原生事件先于React事件执行

### **阻止事件冒泡：**

> React中使用e.stopPropagation()方法来阻止React的事件冒泡，但是他只能阻止合成事件间冒泡，即下层的合成事件不会冒泡到上层的合成事件。对于原生事件，不能通过stopPropagation来阻止。**因此，在react中，阻止冒泡的方式有三种：**
>
> 1. 阻止合成事件与非合成事件之间的冒泡(原生事件与合成事件混用的情况)，需要用到e.target 判断
>
> 2. 阻止合成事件与最外层root上的事件间的冒泡，用e.nativeEvent.stopImmediatePropagation()
>
> 3. 阻止合成事件间的冒泡，用e.stopPropagation()

```js
import React from "react";
class StopBubbling extends React.Component {
  componentDidMount() {
    document.getElementById("root").addEventListener("click", this.handleRootClick);
    document.body.addEventListener("click", this.handleBodyClick, false);
    document.addEventListener("click", this.handleDocumentClick, false);
  }
  handleDocumentClick = (e) => {
    console.log("handleDocumentClick！");
  };
  handleBodyClick = (e) => {
    console.log("handleBodyClick！ ");
  };
  handleRootClick = (e) => {
    console.log("handleRootClick！ ");
  };
  handleClickTestBox = (e) => {
    console.warn("handleClickTestBox！");
  };
  handleClickTestBox2 = (e) => {
    console.warn("handleClickTestBox2！");
  };
  handleClickTestBox3 = (e) => {
    console.warn("handleClickTestBox3！");
  };
  render() {
    return (
      <div className="test-box" onClick={this.handleClickTestBox} style={{ background: "red" }}>
        Box
        <div onClick={this.handleClickTestBox2} style={{ background: "green", margin: "10px" }}>
          Box2
          <div id="inner" onClick={this.handleClickTestBox3} style={{ background: "yellow", margin: "10px" }}>
            Box3
          </div>
        </div>
      </div>
    );
  }
}

export default StopBubbling;
```

当box3被点击时控制台中的打印：

```txt
handleClickTestBox3！
handleClickTestBox2！
handleClickTestBox！
handleRootClick！
handleBodyClick！ 
handleDocumentClick！
```

#### 阻止合成事件与非合成事件之间的冒泡

> 通过这种手动判断进行筛选的方式来阻止合成事件与原生事件之间的冒泡。**合成事件和原生事件最好不要混用。**
>
> 因为，React中的执行e.stopPropagation()可以阻止React合成事件冒泡，但是不能阻止原生事件冒泡；反之，在原生事件中执行e.stopPropagation()阻止冒泡行为，会导致所有元素的事件将无法冒泡到Root上，即所有的 React 事件都将无法被注册。

```js
handleBodyClick = (e) => {
  if (e.target && e.target === document.querySelector("#inner")) {
    return;
  }
  console.log("handleBodyClick！ ");
};
```

```js
handleClickTestBox3！
handleClickTestBox2！
handleClickTestBox！
handleRootClick！
handleDocumentClick！
```

#### 阻止合成事件与最外层root上的事件间的冒泡

> 由于合成事件是委托在root中的，在这里阻止原生事件冒泡，只是会阻止他不会继续向root元素之上冒泡而已。
>
> 但是这里handleRootClick！没有被打印是由于使用了stopImmediatePropagation方法，虽然事件已经冒泡到root上了，理应执行root中所有的处理函数，但是stopImmediatePropagation不仅会阻止冒泡，还会阻止当前元素的后续事件处理函数。

```js
handleClickTestBox3 = (e) => {
  e.nativeEvent.stopImmediatePropagation();
  console.warn("handleClickTestBox3！");
};
```

```js
handleClickTestBox3！
handleClickTestBox2！
handleClickTestBox！
```

> 只是阻止继续向上冒泡，不会阻止继续执行当前元素的其它事件处理函数

```js
handleClickTestBox3 = (e) => {
  e.nativeEvent.stopPropagation();
  console.warn("handleClickTestBox3！");
};
```

```js
handleClickTestBox3！
handleClickTestBox2！
handleClickTestBox！
handleRootClick！
```

#### 阻止合成事件间的冒泡

> 这里在box3的事件处理函数中阻止了合成事件冒泡，那么他就不会再向上层合成事件冒泡，但是这里除了react在root上面写了一个点击事件的委托之外，我们自己还增加了一个root元素的点击事件处理函数，因此这里他们都会被执行，所以handleRootClick！也会被打印。

```js
handleClickTestBox3 = (e) => {
  e.stopPropagation();
  console.warn("handleClickTestBox3！");
};
```

```js
handleClickTestBox3！
handleRootClick！
```

### **this指向问题**

1. 使用bind解决

```js
import React, { Component } from 'react';

class App extends Component {
  // 类的构造函数，它只执行1次，在初始化
  constructor(props) {
    super(props);
    this.num = 100
    this.addNum = this.addNum.bind(this)
  }

  addNum(evt, n = 1) {
    this.num += n
    this.forceUpdate()
  }

  render() {
    return (
      <div>
        <button onClick={this.addNum}>累加</button>
      </div>
    );
  }
}

export default App;
```

2. 使用箭头函数解决

```js
import React, { Component } from 'react';

class App extends Component {
  num = 100
  addNum = (evt, n = 1) => {
    this.num += n
    this.forceUpdate()
  }

  render() {
    return (
      <div>
        <h3>{this.num}</h3>
        <button onClick={evt => this.addNum(evt, 2)}>累加</button>
      </div>
    );
  }
}

export default App;
```

## state状态：

> state状态只在**class类组件才有**，函数组件没有此功能。成员属性state它是一个特殊的属性，它是当前类的私有数据，只有在当前的组件中才能操作里面的数据，针对于此属性，react提供一个专门的修改数据的方法  this.setState ，用来修改当前属性中的方法，该方法可以触发当前state中的数据变化，和视图更新。
>
> 1. 状态(state)即数据，是组件内部的**私有数据**，只能在组件内部使用,和vue中data差不多，不过它没有像vue中的data进行了数据劫持
> 2. state的值是**对象**，表示一个组件中可以有多个数据
> 3. 通过**this.state**来获取状态，react中没有做数据代理
> 4. state数据值同时让视图更新则需要调用专用的方法 **this.setState**
> 5. state它是类的一个成员属性

```js
import React, { Component } from 'react';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      num: 100
    }
  }

  addNum(evt, n = 1) {
    // 同步修改数据，它不会触发视图更新
    this.state.num += n
  }

  render() {
    return (
      <div>
        <h3>{this.state.num}</h3>
        <button onClick={evt => this.addNum(evt, 2)}>累加</button>
      </div>
    );
  }
}

export default App;
```

### **state状态的同步与异步改变**

> setState本身并不是异步，但是react为了性能优化，使用state在某些情况下体现为异步。比如在react的生命周期函数或者作用域下为异步，在原生的环境下为同步。
>
> 总结：setState既可以是异步也可以是同步。

setState语法（可以传入一个对象，也可以传入一个函数）：

```js
this.setState({key:value}|state=>({key:value}),()=>{})
```

当使用对象方式更新state状态时，react会把这个操作放进一个任务队列中，当这个任务被真正执行时，会执行传入setState中的回调函数：

```js
this.setState({
   num: this.state.num + 1
 }, () => {
   console.log(this.state.num);
})
```

如果执行了多次setState，react会把相同的操作进行合并，下面addNum被执行后num会被+3：

```js
addNum(evt, n = 1) {
  this.setState({
    num: this.state.num + 1,
  });
  this.setState({
    num: this.state.num + 2,
  });
  this.setState({
    num: this.state.num + 3,
  });
}
```

如果为了保证数据完整性，不想让他进行合并，可以使用函数语法：

```js
addNum(evt, n = 1) {
  this.setState((state) => ({
    num: state.num + 1,
  }));
  this.setState((state) => ({
    num: state.num + 2,
  }));
  this.setState((state) => ({
    num: state.num + 3,
  }));
}
```

在原生环境中，react不会把setState放进任务队列中，因此在原生环境中，setState操作是同步的：

```js
addNum(evt, n = 1) {
  setTimeout(() => {
    this.setState({
      num: this.state.num + 3,
    });
    console.log(this.state.num); // 可以直接读取正确数据
  }, 0);
}

componentDidMount() {
  document.onclick = () => {
    this.setState({
      num: this.state.num + 3,
    });
    console.log(this.state.num); // 可以直接读取正确数据
  };
}
```

> 小技巧：
>
> ```js
> // 如果你setState写了一个空对象，则它会更新视图一次
> this.setState({})
> // 如果你写了为null,setState不会做任何事件，相当于没有调用
> this.setState(null)
> ```

### **todoList案例**

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    todos: []
  }
  onEnter = evt => {
    if (evt.keyCode === 13) {
      let title = evt.target.value.trim()
      this.setState(state => ({
        todos: [...state.todos, { id: Date.now(), title, done: false }]
      }), () => evt.target.value = '')
    }
  }
  del = tid => () => {
    this.setState(state => ({
      todos: state.todos.filter(({ id }) => id != tid)
    }))
  }
  delIndex = index => () => {
    this.state.todos.splice(index, 1)
    this.setState({})
  }

  render() {
    return (
      <div>
        <div>
          <input type="text" onKeyUp={this.onEnter} />
        </div>
        <hr />
        <ul>
          {
            this.state.todos.map((item, index) => (
              <li key={item.id}>
                <span>{item.title}</span>
                <span onClick={this.del(item.id)}>删除</span>
                <span onClick={this.delIndex(index)}>删除</span>
              </li>
            ))
          }
        </ul>
      </div>
    );
  }
}

export default App;
```

### **购物车案例**

```js
import React, { Component } from 'react';

class App extends Component {
  state = {
    carts: [
      { id: 1, name: '水果手机14', price: 1, num: 1 },
      { id: 2, name: '大米手机14', price: 1, num: 2 },
      { id: 3, name: '一般手机14', price: 1, num: 3 },
    ]
  }
  setNum = (n, index) => {
    this.state.carts[index]['num'] += n
    if (this.state.carts[index]['num'] <= 1) this.state.carts[index]['num'] = 1
    if (this.state.carts[index]['num'] >= 5) this.state.carts[index]['num'] = 5
    this.setState({})
  }
  totalPrice = () => {
    return this.state.carts.reduce((p, c) => {
      p += c.num * c.price
      return p
    }, 0)
  }

  render() {
    return (
      <div>
        <table width='600' border='1'>
          <thead>
            <tr>
              <th>ID</th>
              <th>名称</th>
              <th>价格</th>
              <th>数量</th>
            </tr>
          </thead>
          <tbody>
            {
              this.state.carts.map((item, index) => (
                <tr key={item.id}>
                  <td>{item.id}</td>
                  <td>{item.name}</td>
                  <td>{item.price}</td>
                  <td>
                    <button onClick={() => this.setNum(-1, index)}>---</button>
                    <span>{item.num}</span>
                    <button onClick={() => this.setNum(1, index)}>+++</button>
                  </td>
                </tr>
              ))
            }
          </tbody>
        </table>
        <hr />
        <h3>{this.totalPrice()}</h3>
      </div>
    );
  }
}
export default App;
```

## props参数：

### **props.children属性**

> children属性，表示组件标签的子节点，当组件标签有子节点时，props就会有该属性，与普通的props一样，其值可以为任意类型。
>
> 在自定义组件标签中间写的内容，它都会通过 this.props.children属性获取，如果自定义组件标签中间只有一个子元素，则此属性返回一个对象，如果是多个子元素则返回一个数组，使用它就可以实现类似于vue中的插槽功能

```jsx
import React, { Component } from 'react';

class Child extends Component {
  render() {
    return (
      <div>
        <h3>我是child组件</h3>
        {
          this.props.children
            ?
            this.props.children
            :
            <div>我是默认值</div>
        }
      </div>
    );
  }
}

class App extends Component {
  render() {
    return (
      <div>
        <Child>
          <div>我是child组件元素中间包裹的内容1</div>
          <div>我是child组件元素中间包裹的内容2</div>
          <div>我是child组件元素中间包裹的内容3</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

如果想要改变父元素传过来children元素，比如给它添加样式，或者绑定事件，直接改变props.children.props是不行的，因为它是不可以直接进行修改的，因此React提供了一个cloneElement方法，用于克隆一个React对象，这样就可以在克隆的时候修React对象的props属性了：

```js
import React, { Component } from 'react';

class Child extends Component {
  state = {
    id: 1000
  }
  render() {
    let cloneElement = React.cloneElement(this.props.children, {
      style: { color: 'red' },
      uid: this.state.id,
      onClick: () => console.log('我是事件', this.state.id)
    })
    return (
      <div>
        <h3>我是child组件</h3>
        <hr />
        {cloneElement}
      </div>
    );
  }
}

class App extends Component {
  state = {
    title: '我是child组件元素中间包裹的内容'
  }
  render() {
    return (
      <div>
        <Child>
          <div>{this.state.title}</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

### 模拟具名插槽：

```jsx
import React, { Component } from "react";

class Child extends Component {
  state = {
    id: 1000,
  };
  render() {
    let cloneElement = React.Children.map(this.props.children, (child, index) => {
      return React.cloneElement(child, {
        style: { color: "red" },
        uid: this.state.id,
        onClick: () => console.log("我是事件", this.state.id),
      });
    });
    return (
      <div>
        {this.props.header}
        <hr />
        {cloneElement}
        <hr />
        {this.props.footer}
      </div>
    );
  }
}

class App extends Component {
  render() {
    return (
      <div>
        <Child header={<div>我是头部</div>} footer={<div>底部</div>}>
          <div>我是child组件元素中间包裹的内容1</div>
          <div>我是child组件元素中间包裹的内容2</div>
          <div>我是child组件元素中间包裹的内容3</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

### **props类型限制**

> 对于组件来说，props是外部传入的，无法保证组件使用者传入什么格式的数据，简单来说就是组件调用者可能不知道组件封装着需要什么样的数据，如果传入的数据不对，可能会导致程序异常，所以必须要对于props传入的数据类型进行校验。

数据类型校验包在15.5之后就移出了核心库存，因此需要单独安装：

```js
npm i -S prop-types
```

在组件中导入：

```js
import PropTypes from 'prop-types'
```

使用方法：

```js
# 函数组件
function App(){}
App.propTypes = {
    prop-name:PropTypes.string
}

# 类组件
class App extends Component{
}
App.propTypes = {
    prop-name:PropTypes.string
}
```

类型：

- 类型： array、bool、func、number、object、string

- React元素类型：element

- 必填项：types.number.isRequired
- 指定元素类型的数组类型：types.arrayOf(types.number)
- 多个类型：types.oneOfType([types.number, types.string])

- 特定结构的对象： shape({})

  ```js
  types.shape({
        id: types.oneOfType([types.number, types.string]),
        name: types.string,
        fn: types.func
   })
  ```

- 自定义规则：

  ```js
  phone: (props, key) => {
      let value = props[key]
      if (!/^1[3-9]\d{9}$/.test(value)) {
          // 如果不正确，一定要返回一个Error对象，这样就可以在控制台中看到信息,不要throw
          return new Error('有误')
       }
  }
  ```

### **props默认值：**

```jsx
import React, { Component } from 'react';
import types from 'prop-types'

class Child extends Component {
  static propTypes = { // 设置类型验证
    age: types.number,
  }
  static defaultProps = { // 设置默认值
    age: 2000
  }

  render() {
    return (
      <div>
        <h3>我是child组件</h3>
      </div>
    );
  }
}
```

## 表单处理：

### **受控组件**

> 将state与表单项中的value值绑定在一起，有state的值来控制表单元素的值，称为受控组件。
>
> 绑定步骤：
>
> - 在state中添加一个状态，作为表单元素的value值
>
> - 给表单元素绑定change事件，将表单元素的值设置为state的值

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    username: '',
    password: ''
  }
  setValue = name => evt => {
    this.setState({
      [name]: evt.target.value.trim()
    })
  }
  render() {
    let { username, password } = this.state
    return (
      <div>
        <div>
          <input type="text" value={username} onChange={this.setValue('username')} />
        </div>
        <div>
          <input type="text" value={password} onChange={this.setValue('password')} />
        </div>
      </div>
    );
  }
}

export default App;
```

> 另一种方法(简写)

```jsx
import React, { Component } from "react";

class App extends Component {
  state = {
    username: {
      value: "admin",
      onChange: (e) => this.setState((state) => ({ username: { ...state.username, value: e.target.value } })),
    },
    password: {
      value: "admin888",
      onChange: (e) => this.setState((state) => ({ password: { ...state.password, value: e.target.value } })),
    },
    num: 100,
  };
  render() {
    let { username, password } = this.state;
    return (
      <div>
        <div>
          <input type="text" {...username} />
        </div>
        <div>
          <input type="text" {...password} />
        </div>
      </div>
    );
  }
}

export default App;
```

> 单选框、多选框

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    sex: '女',
    lessons: ['html', 'js']
  }
  setLessonsFn = e => {
    if (e.target.checked) {
      this.setState(state => ({
        lessons: [...state.lessons, e.target.value]
      }))
    } else {
      this.setState(state => ({
        lessons: state.lessons.filter(item => item != e.target.value)
      }))
    }
  }
  render() {
    let { sex,lessons } = this.state
    return (
      <div>
        <div>
          {/* 单选 */}
          <input type="radio" name='sex' checked={sex === '男'} value='男' onChange={e => this.setState({ sex: e.target.value })} />男
          <input type="radio" name='sex' checked={sex === '女'} value='女' onChange={e => this.setState({ sex: e.target.value })} />女
        </div>
        <div>
          {/* 复选框 */}
          <div>
            <input type="checkbox" value='html' checked={lessons.includes('html')} onChange={this.setLessonsFn} />
            <input type="checkbox" value='css' checked={lessons.includes('css')} onChange={this.setLessonsFn} />
            <input type="checkbox" value='js' checked={lessons.includes('js')} onChange={this.setLessonsFn} />
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```

### **非受控组件**(ref)

> 非受控组件：表单项中的value或checked值，它不受state中的属性控制，可以通过dom对象来获取当前表单项中的value或checked值。
>
> 没有和state数据源进行关联的表单项，而是借助ref，使用元素DOM方式获取表单元素值的步骤：
>
> - 调用 React.createRef() 方法创建ref对象
>
> - 将创建好的 ref 对象添加到文本框中
>
> - 通过ref对象获取到文本框的值
>
> ref 如果绑定在普通的html元素中则返回dom对象
> ref 如果绑定在"类组件"中，则返回当前自定义类组件实例对象，可以用它来完成组件通信

```jsx
import React, { Component, createRef } from 'react'

class App extends Component {
  usernameRef = createRef()
  getUsernameInput = () => {
    console.log(this.usernameRef.current.value)
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.usernameRef} />
        <button onClick={this.getUsernameInput}>获取表单项中的数据</button>
      </div>
    )
  }
}

export default App
```

> 绑定多选框

```jsx
import React, { Component, createRef } from 'react'

class App extends Component {
  lessonsRefs = Array(3)
    .fill('')
    .map(_ => createRef())

  getUsernameInput = evt => {
    console.log(
      this.lessonsRefs
        .map(item => {
          if (item.current.checked) {
            return item.current.value
          } else {
            return null
          }
        })
        .filter(item => item != null)
    )
  }

  render() {
    return (
      <div>
        <input type="checkbox" value="html" ref={this.lessonsRefs[0]} />
        <input type="checkbox" value="css" ref={this.lessonsRefs[1]} />
        <input type="checkbox" value="js" ref={this.lessonsRefs[2]} />
        <button onClick={this.getUsernameInput}>获取表单项中的数据</button>
      </div>
    )
  }
}

export default App
```

