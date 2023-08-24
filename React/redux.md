## redux

### 安装redux

> redux不是内嵌在react框架中，使用时需要手动去安装

```shell
$ npm i -S redux
```

### 核心概念

**单一数据源(state)：**

> 整个redux中的数据都是集中管理，存储于同一个数据源中，数据源中的数据为单向数据流，不可直接修改

**纯函数(reducer)统一对state数据修改：**

> redux定义了一个reducer函数来完成state数据的修改，reducer会接收先前的state和action，并返回新的state

纯函数是什么？

1. 函数执行结果是可预期且多次调用结果相同 fn(n1,n2){return n1+n2}

2. 函数执行不会触发副作用 

3. 函数中的变量，没有使用外部的

**具体流程：**

![](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/%E5%A1%94%E9%A1%B6%E5%A1%94%E9%A1%B6%E5%9C%B0.png)

1. store通过reducer创建了初始状态

2. component通过store.getState()获取到了store中保存的state挂载在了自己的状态上

3. 用户产生了操作，调用了actions 的方法

4. actions的方法被调用，创建了带有标示性信息的action（描述对象）

5. actions将action通过调用store.dispatch方法发送到了reducer中

6. reducer接收到action并根据标识信息判断之后返回了新的state

7. store的state被reducer更改为新state的时候，store.subscribe方法里的回调函数会执行，此时就可以通知component去重新获取state

### 使用redux

**计数器案例：**

- **src/store/index.js**

```js
import { createStore } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'

const initState = {
  num: 100
}

// 定义一个纯函数reducer，专门用来操作state中的数据,要返回一个新的state
const reducer = (state = initState, action) => {
  if (action.type === 'add') return { ...state, num: state.num + action.payload }
  return state;
}

let store
// 开发与生产环境的判断，生产环境中要关闭开发者工具，提高安全性
process.env.NODE_ENV === 'development'
  ?
  store = createStore(
    reducer,
    composeWithDevTools()
  )
  :
  store = createStore(
    reducer
  )

export default store
```

- **src/app.jsx**

```jsx
import React, { useRef, useState } from 'react'
import store from './store'

const App = () => {
  const [_, update] = useState({})
  const handleUnSubscribe = useRef(null)
  
  useEffect(() => {
    // 由于没有将数据存入state中，所以无法使用setState触发更新
    // 因此这里在检测到数据更新后直接使用forceUpdate强制更新页面
    handleUnSubscribe.current = store.subscribe(() => this.forceUpdate())
    // store.subscribe(() => this.setState(state => store.getState()))
    return () => {
     	handleUnSubscribe.current?.() // 取消订阅
    }
  }, [])

  render() {
    return (
      <div>
        <h3>{store.getState().num}</h3>
        <hr />
        <button
          onClick={() => {
            store.dispatch({ type: 'add', payload: 2 })
          }}
        >
          ++++
        </button>
      </div>
    )
  }
}

export default App
```

## react-redux

> React-Redux是Redux的官方针对React开发的**扩展**库，默认没有在React项目中安装，需要手动来安装。react-redux是依赖于redux，所以必须安装redux
>
> 可以理解为react-redux就是redux给我们提供一些高阶组件，能解决的问题是：使用它以后我们不需要在每个组件中再去手动订阅数据的更新了

### 安装：

```shell
$ npm i -S react-redux  redux
```

### 使用：

**定义Provider：**

> 在程序主文件index.js文件中,定义Provider，并提供数据源，也就是store对象，可以猜测react-redux是通过生产消费者模式为给子组件提供数据的
>
> 类似于之前跨组件通信处的Provider一样，旨在让全局的组件共享store中的数据

```jsx
import React from 'react';
import App from './App';
import { Provider } from 'react-redux'
import store from './store'
import { BrowserRouter as Router, HashRouter } from 'react-router-dom'

ReactDOM.render(
  <Provider store={store}>
    <Router>
      <App />
    </Router>
  </Provider>,
  document.getElementById('root')
)
```

### react-redux-hooks

> react-redux支持了hook的解决方案，提供两个方法:
>
> - **useDispatch：**用于获取redux中的state数据
> - **useSelector：**用于发送指令（与store.dispatch相同）

```jsx
import React from 'react'
import { useDispatch, useSelector } from 'react-redux'

const MyRedux = () => {
  const { username, password } = useSelector(state => state.userinfo)
  const dispatch = useDispatch()

  return (
    <div>
      <h3>{username}</h3>
      <hr />
      <button onClick={
        e => {
          dispatch({
            type: 'setname',
            name: 'aaaaa'
          })
        }
      }>修改一下姓名</button>
    </div>
  )
}
```

## redux模块化

> 使用redux库中提供的**combineReducers**方法，可以将多个拆分reducer函数合并成统一的reducer函数，提供给createStore来使用。
>
> combineReducers会把state进行模块化拆分，但它并不会把reducer函数中的type名称进行模块化命名，因此需要注意 type的类型名称不能重名，如果重名则都会执行。
>
> 拆分后的state：{ count: { num: 100 }, film: { nowplayings: [] } }
>
> 如果想要避免type的命名冲突，则需要规范type的命名，比如以拆分后的文件名称为前缀（xxx_type类型名）。

- **src/store/index.js**

```js
import { createStore, combineReducers } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'
import count from './reducers/count'
import film from './reducers/film'

const reducer = combineReducers({
  count,
  film
})

const store = createStore(
  reducer,
  composeWithDevTools()
)

export default store
```

- **src/store/reducers/count.js**

```js
const initState = {
  num: 100
}

const reducer = (state = initState, action) => {
  if (action.type === 'count_add_num') return { ...state, num: state.num + action.payload }
  return state;
}

export default reducer
```

- **src/store/reducers/film.js**

```js
const initState = {
  nowplayings: []
}

const reducer = (state = initState, action) => {
  if (action.type === 'film_set_nowplayings') return { ...state, nowplayings: action.payload }
  return state;
}

export default reducer
```

## redux中间件

> Redux中间件执行时机是**在dispatching action和到达reducer之间**。也就是说，原本组件中执行store.dispatch会直接传递到reducer中，现在它会再经过所有的中间件后再去到reducer中
>
> 中间件的原理就是封闭了redux自己的dispatch方法：
>
> - 没有中间件：store.dispatch()就是Redux库自己提供的dispatch方法，用来发起状态更新
> - 使用中间件：store.dispatch()就是中间件封装处理后的dispatch，但是最终一定会调用Redux自己的dispatch方法发起状态更新

- 没有中间件：**`dispatch(action) => reducer`**

![d57303e625c00c0509865edea7d9affc](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/d57303e625c00c0509865edea7d9affc.png)

- 使用中间件：**`dispatch(action) => 执行中间件代码 => reducer`**

![fdc7f48d1b87fe7ebbe753578a75dbba](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fdc7f48d1b87fe7ebbe753578a75dbba.png)

### 自定义中间件

```js
// Redux 中间件的写法：
const myMiddleware = store => next => action => { /* 此处写 中间件 的代码 */ }
```

```js
// store 表示：redux 的 store
// next 表示：下一个中间件，如果只使用一个中间，那么 next 就是 store.dispatch
// action 表示：要分发的动作
const logger = store => next => action => {
  console.log('prev state:', store.getState()) // 更新前的状态
  console.log('dispatching', action)
  let result = next(action)
  // 上面 next 代码执行后，redux 状态就已经更新了，所以，再 getState() 拿到的就是更新后的最新状态值
  console.log('next state', store.getState()) // 更新后的状态
  return result
}
```

### redux-logger：

> 与上面自己封闭的logger中间件大致相同，只是他做了更多的边界判断

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'
import count from './reducers/count'
import film from './reducers/film'
import logger from 'redux-logger'

const reducer = combineReducers({
  count,
  film
})

const store = createStore(
  reducer,
  composeWithDevTools(applyMiddleware(logger))
)

export default store
```

### redux-thunk：

> redux-thunk它是由redux官方开发出来的redux中间件，它的作用：解决redux中使用异步处理方案。

mapDispatchToProps的对象写法中，action只能返回一个对象，connect方法会主动调用dispatch，把action返回的对象传入其中。而dispatch方法显然不能接收一个Promise，因此会报错。

```js
const mapDispatchToProps = {
  add: (n = 1) => ({ type: 'add', payload: n })
}
```

而redux-thunk让我们可以在对象写法的action中返回一个函数，这个函数会接收三个参数，第一个参数就是store.dispatch，可以使用这个方法手动dispatch一个action，从而实现异步。

```js
const mapDispatchToProps = {
   add: (page = 1) => async dispatch => {
      let ret = await getNowPlayingFilmListApi(page)
      dispatch({ type: film_set_nowplayings, payload: ret.data.films })
   }
}
```

**redux-thunk源码解析：**

> 如果action是一个函数，则把dispatch等参数注入给它，这样就可以在这个函数中执行异步操作，异步操作完成之后调用dispatch，抛发一个动作，由于dispatch是被middleWare封装过的，因此他会重新再经历所有的中间件，其实就是一个隐式的递归，这次再经过redux-thunk的时候，由于action已经是一个对象了(或者还是一个函数，那就再递归一遍)，因此就会next到下一个中间件。

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    // 否则直接next到下一个中间件
    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

**函数action特性:**

```ts
type Store = ReturnType<typeof makeStore>

export type RootStateType = ReturnType<Store['getState']>

export type AppThunkActionType<ReturnType = void> = ThunkAction<ReturnType, RootStateType, unknown, Action<string>>
```

AppThunkActionType这个类型是用来标识redux-thunk的函数action类型的，其中的泛型参数为这个函数action的返回值，可以看到下面的例子中，他的返回值是一个promise，解决后为一个string

```ts
export const fetchTest = (): AppThunkActionType<Promise<string>> => async (dispatch) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('aaaa')
    }, 1000)
  })
}
```

通过dispatch这个函数action，dispatch会直接返回这个函数action，因此可以通过await进行操作

```tsx
const Com = () => {
  const dispatch = useAppDispatch()

  useEffect(() => {
    async function test() {
      const res = await dispatch(fetchTest())
      console.log(res) // 'aaaa'
    }
    test()
  }, [dispatch])

  return ...
}
```

下面是ThunkDispatch的类型定义，可以看到他有两个重载，第一个重载接口一个ThunkAction(函数action)返回ReturnType(ThunkAction的泛型参数，也就是函数action的返回值)，而第二个重载接收一个普通Action，同时直接返回这个Action

react-thunk的dispatch方法会直接返回传入的函数action，这样就使用我们可以在这个函数中做一些事情，并且可以通过await去接口这个函数中的返回值

```ts
export interface ThunkDispatch<State, ExtraThunkArg, BasicAction extends Action> {
    /** Accepts a thunk function, runs it, and returns whatever the thunk itself returns */
    <ReturnType>(thunkAction: ThunkAction<ReturnType, State, ExtraThunkArg, BasicAction>): ReturnType;
    /** Accepts a standard action object, and returns that action object */
    <Action extends BasicAction>(action: Action): Action;
    /** A union of the other two overloads for TS inference purposes */
    <ReturnType, Action extends BasicAction>(action: Action | ThunkAction<ReturnType, State, ExtraThunkArg, BasicAction>): Action | ReturnType;
}
```

下面是ThunkAction的类型定义，可以看这函数action中会接收一个参数ThunkDispatch(它就是react-redux提供的dispatch方法，可以dispatch函数action和普通action)，redux-thunk会判断dispatch提交的action的类型，如果是函数，则会执行这个函数，并返回它的结果，而如果是一个普通的action，就会直接提交到reducer，从而形成一个递归，直到调用完毕

```ts
export declare type ThunkAction<ReturnType, State, ExtraThunkArg, BasicAction extends Action> = (dispatch: ThunkDispatch<State, ExtraThunkArg, BasicAction>, getState: () => State, extraArgument: ExtraThunkArg) => ReturnType;
```

### redux-saga：

> redux-saga是redux一个中间件，它是基于ES6的**Generator**功能实现，用于解决异步问题（让redux中可以直接进行异步操作）

![1](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/1.png)

**安装：**

```shell
$ npm i -S redux-saga
```

**常见effect的用法：**

1. call 异步阻塞调用
2. all 监听多个监听saga，它的功能和Promise.all方法一样
3. fork 异步非阻塞调用，无阻塞的执行fn，执行fn时，不会暂停Generator
4. put 相当于dispatch，分发一个action
5. select 相当于getState，用于获取store中相应部分的state
6. take 监听action，暂停Generator，匹配的action被发起时，恢复执行。take结合fork，可以实现takeEvery和takeLatest的效果
7. takeEvery 监听action，每监听到一个action，就执行一次操作
8. takeLatest 监听action，监听到多个action，只执行最近的一次
9. cancel 指示 middleware 取消之前的 fork 任务，cancel 是一个无阻塞 Effect。也就是说，Generator 将在取消异常被抛出后立即恢复
10. race 竞速执行多个任务
11. throttle 节流

**项目中使用redux-saga：**

- **src/store/index.js**

```js
import { createStore, applyMiddleware } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'
// 合并后的reducer
import reducer from './reducer'
// saga中间件
import createSagaMiddleware from 'redux-saga'
import mainSaga from './sagas'

// 创建saga中间件
const sagaMiddleware = createSagaMiddleware()

const store = createStore(reducer, composeWithDevTools(sagaMiddleware)))

// 运行saga
sagaMiddleware.run(mainSaga)

export default store
```

- **src/store/saga.js**

```js
import { takeEvery, put, all } from 'redux-saga/effects'
import { loginApi } from '@/api/userApi'

// saga中间件主saga，用于区别是否需要saga来处理异步操作，如果没有异步，则放行
function* mainSaga() {
  // 这样的调用，它只能监听一个saga，不能进行模块化
  yield watchSaga()
}

// 监听saga，监听type类型为异步操作名称的，此saga会通过主saga分配过来
function* watchSaga() {
  yield takeEvery('asyncUserLogin', workSaga)
}

// 工作saga，监听saga得到任务后，把任务分配给工作saga
function* workSaga({ payload }) {
  let ret = yield call(loginApi, payload)
  // 得到的数据同步到redux中
  if (ret.code === 0) {
    // userLogin会把payload中的数据同步到redux
    yield put({ type: 'userLogin', payload: ret.data })
  }
}

export default mainSaga
```

在有了saga中间件后，再抛发action时，如果saga发现它是被监听的action（有异步请求的action），那它会被saga拦截，在异步请求完成之后，再在workSaga中抛发一个同步的action

