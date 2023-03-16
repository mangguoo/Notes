## react18新特性：

> - 改进已有属性，如自动批量处理【setState】、改进Suspense、组件返回undefined不再报错等
>
> - 支持Concurrent模式，带来新的API，如useTransition、useDeferredValue等

### 创建项目：

```shell
// 此方案创建出来的项目使用为react18版本
$ npx create-react-app myapp 
$ npm init react-app myapp 
$ yarn create react-app myapp
```

注意：

- nodejs版本一定要为16.x及以上版本，如果你用的是win笔记本，则操作系统不能低于win10
- react18中的webpack版本为5版本

#### 入口文件的不同：

**react17：**

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

**react18：**

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

### setState优化：

> 在react18之后，setState都为异步，无论写在什么样的语法环境中

```jsx
import React, { Component } from 'react'
import { flushSync } from 'react-dom'

class App extends Component {
  state = {
    count: 100
  }

  addCount = () => {
    // Concurrent模式下所有useState都为异步执行（为了优化性能）
    this.setState(
      state => ({ count: state.count + 1 }),
      () => console.log(this.state.count)
    )
    // 此方案在react18之前，它里面的操作是同步的，但在react18中由于使用concurrent模式后都为异步
    setTimeout(() => {
      this.setState(state => ({ count: state.count + 1 }))
      console.log(this.state.count)
    }, 1)

    // 使用flushSync方法可以使setState方法变为同步【在生产中不建议使用】
    flushSync(() => {
      this.setState(state => ({ count: state.count + 1 }))
    })
    // setState放在flushSync方法中变成了同步执行，所以在此处可以得到最新的数据
    console.log(this.state.count)
  }

  render() {
    return (
      <div>
        <h3>{this.state.count}</h3>
        <button onClick={this.addCount}>累加count</button>
      </div>
    )
  }
}

export default App
```

### Suspense增强：

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

### useTransition：

> 可以用来降低渲染优先级。分别用来包裹计算量大的 function 和 value，降低优先级，减少重复渲染次数。
>
> 举个例子：搜索引擎的关键词联想。一般来说，对于用户在输入框中输入都希望是实时更新的，如果此时联想词比较多同时也要实时更新的话，这就可能会导致用户的输入会卡顿。这样一来用户的体验会变差，这并不是我们想要的结果。
>
> 我们将这个场景的状态更新提取出来：一个是用户输入的更新；一个是联想词的更新。这个两个更新紧急程度显然前者大于后者。
>
> 以前我们可以使用防抖的操作来过滤不必要的更新，但防抖有一个弊端，当我们长时间的持续输入（时间间隔小于防抖设置的时间），页面就会长时间都不到响应。而 startTransition 可以指定 UI 的渲染优先级，哪些需要实时更新，哪些需要延迟更新。
>
> useTransition 接受传入一个毫秒的参数用来修改最迟更新时间，返回一个过渡期的 pending 状态和 startTransition 函数。

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

### useDeferredValue：

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

