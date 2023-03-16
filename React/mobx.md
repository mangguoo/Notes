# mobx

> mobx它是一个简单、可扩展的状态管理。作用和redux是一样的，但是它的写法要比redux要简单许多，也是现在react生态中非常流行的一个状态管理。MobX >=5 版本可以运行在任何支持 ES6 **Proxy** 的浏览器中
>
> - mobx5它是使用装饰器来完成的，对于函数组件支持不是太好，适合类组件
>
> - mobx6已不再使用装饰器，对于函数组件的支持更好

**mobx的运行流程：**

![fdsa23f1a](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fdsa23f1a.png)

## mobx5

> mobx5建议在17及以下版本的react中去使用,mobx5是通过装饰器来完成对于属性和方法的装饰来实现数据的响应式
>
> **安装mobx5：**
>
> ```shell
> $ npm i -S mobx@5 mobx-react@5 --force
> ```

### 单模块使用

- **/src/mobx/index.js**

```tsx
import { observable, computed, action, runInAction, configure } from 'mobx'

// 修改当前mobx的执行模式 always 为严格模式，在always模式下，异步请求的action中不能直接修改状态值，必须另外建立一个action为修改状态值
configure({ enforceActions: 'always' })

class Store {
  @observable num = 100 // observable让此属性变成一个响应式的属性
  @observable users = []
  // action让成员方法，变成一个可以操作响应数据的方法
  // action.bound可以解决this指向问题，这是为了考虑有可能出现解构后使用该action的情况
  @action.bound
  setNum(n) {
    this.num += n
  }
  @action.bound
  setUsers(data) {
    this.users = data
  }
  // 异步action，如果此方法中有异步操作，不建议直接在此方法中对响应属性进行修改,而是再定义一个方法专门来修改此响应属性(在always模式下，直接在异步action中修改状态值会报错)
  @action.bound
  async fetchUsers(page = 1) {
    let ret = await (await fetch('/api/users?page=' + page)).json()
    // 如果 把执行模式修改为严格 configure({ enforceActions: 'always' })，就不能在此处直接去修改响应数据值
    // this.users = ret.data

    // 定义一个action专门用来修改响应数据(推荐)
    // this.setUsers(ret.data)

    // 如果开启了强制模式，但是还想在当前的action中完成对于响应数据的修改(不推荐)
    runInAction(() => {
      this.users = ret.data
    })
  }
}

// 应该默认导出一个对象，对象的属性的key值就是inject注入的模块名
export default { store: new Store() }
```

- **/src/index.jsx**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

import { Provider } from 'mobx-react'
import store from './mobx'
import { BrowserRouter as Router } from 'react-router-dom'

ReactDOM.render(
  <Provider {...store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

- **/src/app.jsx**

```jsx
import React, { Component } from 'react'
import { observer, inject } from 'mobx-react'

@inject(['store']) // 把store模块注入到该组件中(映射到this.props.store上)
@observer // 让组件变成一个响应式的组件
class App extends Component {
  componentDidMount() {
    let { fetchUsers } = this.props.store // 解构使用也不会出现this指向问题，因为进行了this绑定(@action.bound)
    fetchUsers(2)
  }

  render() {
    let { num, setNum, users } = this.props.store
    return (
      <div>
        <h3>{num}</h3>
        <button onClick={() => setNum(1)}>
          加一
        </button>
        <ul>
          {users.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      </div>
    )
  }
}

export default App
```

### 模块化处理

- **/src/mobx/index.js**

```js
// 使用构造函数store实例
class Store {
  constructor() {
    const moduleFn = require.context('./modules', false, /\.js$/)
    moduleFn.keys().forEach(item => {
      let key = /\.\/(\w+)\.js/.exec(item)[1]
      // 注意一下new和.运算符的优先级
      let value = new (moduleFn(item).default)()
      this[key] = value
    })
  }
}

// 在入口文件中可以展开 {...store}，inject注入模块的名称就是当前成员属性的key
export default new Store()
```

- **/src/mobx/modules/user.js**

```js
import { observable, action, computed } from 'mobx'

class User {
  @observable users = []
  @action.bound
  setUsers(data) {
    this.users = data
  }
  @action.bound
  async fetchUsers(page = 1) {
    let ret = await (await fetch('/api/users?page=' + page)).json()
    this.setUsers(ret.data)
  }
}

export default User
```

- **/src/mobx/modules/count.js**

```js
import { observable, action, computed } from 'mobx'

class Count {
  @observable num = 100
  @action.bound
  setNum(n) {
    this.num += n
  }
}

export default Count
```

- **/src/app.jsx**

```jsx
import React, { Component } from 'react'
import { Switch, Route } from 'react-router-dom'
import User from './pages/User'
import Count from './pages/Count'

class App extends Component {
  render() {
    return <Switch>
      <Route path='/user' component={User} />
      <Route path='/count' component={Count} />
    </Switch>
  }
}

export default App
```

- **/src/pages/User.jsx**

```jsx
import React, { Component } from 'react'
import { observer, inject } from 'mobx-react'

@inject('user')
@observer
class User extends Component {
  componentDidMount() {
    this.props.user.fetchUsers(2)
  }
  render() {
    let { users } = this.props.user
    return (
      <div>
        <ul>
          {users.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      </div>
    )
  }
}

export default User
```

- **/src/pages/Count.jsx**

```jsx
import React, { Component } from 'react'
import { observer, inject } from 'mobx-react'

@inject('count','user') // 可以注入多个
@observer
class Count extends Component {
  render() {
    let { num, setNum } = this.props.count
    return (
      <div>
        <h3>{num}</h3>
        <button onClick={() => setNum(1)}>+++</button>
      </div>
    )
  }
}

export default Count
```

## mobx6

> mobx6中已经不用装饰器了，使用mobx内置方法来完成对于方法和属性数据的响应式处理
>
> **安装mobx6：**
>
> ```js
> npm i -S mobx@6 mobx-react@7
> ```

### 单模块使用

- **/src/mobx/index.js**

```js
import { makeObservable, observable, action, flow, computed, configure } from 'mobx'

// 开启always模式，与mobx5相同
configure({ enforceActions: 'always' })

class Store {
  constructor() {
    this.num = 100
    this.users = [{ id: 1, name: '111111' }]
    // 定义响应属性与action，不再使用装饰器，而是在makeObservable中统一定义
    makeObservable(this, {
      num: observable,
      users: observable,
      showNum: computed,
      setNum: action.bound,
      setUsers: action.bound,
      fetchUsers1: action.bound,
      fetchUsers2: flow.bound // flow使用了generator来解决异步
    })
  }
  // 如果不在makeObservable中进行定义，则它只是个普通方法
  setNum(n) {
    this.num += n
  }
  // 计算属性
  get showNum() {
    return this.num <= 130 ? this.num : '保密'
  }
  setUsers(data) {
    this.users = data
  }
  // 异步操作，使用async/await
  async fetchUsers1() {
    let ret = await (await fetch('/api/users')).json()
    this.setUsers(ret.data)
  }
  // mobx6内置了generator方案，可以用它来完成异步问题
  *fetchUsers2() {
    let response = yield fetch('/api/users')
    let json = yield response.json()
    this.setUsers(json.data)
  }
}

// 直接导出store，可以在组件中直接导入该store进行使用(mobx-react不是必须的，它只是让我们的写法更加优雅)
export default new Store()
```

- **/src/app.jsx**

> 组件中数据的获取与修改：

```jsx
import React, { useEffect } from 'react'
import { observer } from 'mobx-react' // observer可以让组件响应式
import store from '@/mobx' // 直接引入store进行操作，不使用mobx-react

const App = () => {
  useEffect(() => { // 组件挂载完成时请求网络数据
    store.fetchUsers1()
  })
  return (
    <div>
      <h3>{store.num} -- {store.showNum}</h3>
      <button onClick={() => store.setNum(10)}>num加一</button>
      <ul>
        {store.users.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  )
}

// 使用observer 包裹，让当前组件变成响应式组件
export default observer(App)
```

> useObserver 也可以让组件响应式，但是与observer写法不同：

```jsx
import React, { useEffect } from 'react'
import { useObserver } from 'mobx-react'
import store from '@/mobx'

const App = () => {
  useEffect(() => {
    store.fetchUsers()
  })
  return useObserver(() => (
    <div>
      <h3>{store.num} -- {store.showNum}</h3>
      <button onClick={() => store.setNum(10)}>num加一</button>
      <ul>
        {store.users.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  ))
}

export default App
```

> Observer组件也可以让组件实现响应式，它的使用更加灵活：

```jsx
import React, { useEffect } from 'react'
import { Observer } from 'mobx-react'
import store from '@/mobx'

const App = () => {
  useEffect(() => {
    store.fetchUsers()
  })
  return (
    <div>
      {/* 使用Objserver，让子元素变成响应式 */}
      <Observer>
        {() => (
          <h3>
            App组件 -- {store.num} -- {store.showNum}
          </h3>
        )}
      </Observer>
      <button onClick={() => store.setNum(10)}>num加一</button>
      <hr />
      <ul>
        {/* 这里就不是响应式的，user数据修改后，这里不会改变 */}
        {store.users.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  )
}

export default App
```

### 模块化处理

- **/src/mobx/index.js**

```js
import User from './modules/user'

class Store {
  constructor() {
    this.user = new User() // inject注入的名称就是成员属性的key
  }
}

// 导出一个对象，这样方便在provider中去提供属性 {...store}
export default new Store()
```

- **/src/mobx/modules/user.js**

```js
import { makeObservable, observable, action } from 'mobx'
import axios from 'axios'

class User {
  constructor() {
    this.users = []
    makeObservable(this, {
      users: observable,
      setUsers: action.bound,
      fetchUsers: action.bound
    })
  }
  setUsers(data) {
    this.users = data
  }
  async fetchUsers() {
    let ret = await (await fetch('/api/users')).json()
    this.setUsers(ret.data)
  }
}

export default User
```

- **/src/index.jsx**

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import { Provider } from 'mobx-react'
import store from './mobx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider {...store}>
    <App />
  </Provider>
)
```

- **/src/app.jsx**

```jsx
import React, { useEffect } from 'react'
import { observer, inject } from 'mobx-react'

// 对props进行了解构  props.user => { user }
const App = ({ user }) => {
  useEffect(() => {
    user.fetchUsers()
  })
  return (
    <ul>
      {user.users.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul> 
  )
}

// inject中的参数名(user)会映射为props的属性
export default inject('user')(observer(App))
```

### useLocalObservable

> 将对象转换为 `observable` 对象，其中 `getter` 会被转换为 `computed` 属性，方法会与 `store` 进行绑定并自动执行mobx transactions

```jsx
import React, { useState } from 'react'
import { useLocalObservable, useObserver } from 'mobx-react'

const Child = props => {
  // useLocalObservable类似于useReducer
  // useLocalObservable只在初始化时执行1次,后续的更新它不会再次执行(hook的特性)
  const state = useLocalObservable(() => ({
    // 这里响应式数据count的初始值是props.num，由于useLocalObservable只会在组件初始化时执行一次，因此这里count不会随着父组件传递的num变化而变化
    count: props.num, 
    setCount(n) { // action,操作响应数据
      this.count += n
      console.log(this.count)
    },
    get showCount() { // 计算属性
      return this.count >= 103 ? '保密' : this.count
    }
  }))

  return useObserver(() => (
    <div>
      <h3>Child —— {state.count}</h3>
      <button onClick={() => state.setCount(1)}>加一</button>
    </div>
  ))
}

const App = () => {
  let [count, setCount] = useState(1)
  return (
    <div>
      <h3>App —— {count}</h3>
      <button onClick={() => setCount(v => v + 1)}>加一</button>
      <hr />
      {/* 如果你想让你的当前的数据修改后,让useLocalObservable再次执行1次,可以给自定义组件添加一个每次都不相同的key */}
      <Child key={'' + count} num={count} />
    </div>
  )
}

export default App
```

### useAsObservableSource

> 与 `useLocalObservable` 的区别是，它将纯（不包含 `getter` 或方法）对象转换为 `observable`，主要使用场景为：
>
> - 如果对象某个属性的值需经过复杂运算才能获得，可通过该方法进行包装，这样在组件的生命周期中该运算只需要运算一次
> - 一般情况下 `useLocalObservable` 仅用于组件内部，如果 `useLocalObservable` 中的对象需要依赖外部传递的属性，那么可通过`useAsObservableSource` 将这些属性进行转换，而后在 `useLocalObservable` 对象中进行引用，这样在外部属性改变时自动通知`useLocalObservable` 对象对变化进行响应

```jsx
import React, { useState } from 'react'
import { useObserver, useLocalObservable, useAsObservableSource } from 'mobx-react'

const Child = props => {
  let newProps = useAsObservableSource(props)
  const state = useLocalObservable(() => ({
    count: props.num,
    setCount() {
      // 如果使用props.num，它会保持初始化时的数据，而不会根据父组件传过来的props改变而改变。而通过useAsObservableSource把props转为响应对象后，就可以监听到数据的变化
      this.count += newProps.num
    },
    get showCount() { // 计算属性
      return newProps.num >= 103 ? '保密' : this.count
    }
  }))

  return useObserver(() => (
    <div>
      <h3>
        Child -- {state.count} -- {state.showCount}
      </h3>
      <button onClick={() => state.setCount()}>++++</button>
    </div>
  ))
}

const App = () => {
  let [count, setCount] = useState(1)
  return (
    <div>
      <h3>App -- {count}</h3>
      <button onClick={() => setCount(v => v + 1)}>加一</button>
      <hr />
      <Child num={count} />
    </div>
  )
}

export default App
```

