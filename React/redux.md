### 状态管理（redux）

#### 安装redux

> redux不是内嵌在react框架中，使用时需要手动去安装

```jsx
npm i -S redux
```

#### 核心概念

**单一数据源(state)：**

> 整个redux中的数据都是集中管理，存储于同一个数据源中，数据源中的数据为单向数据流，不可直接修改

**纯函数(reducer)统一对state数据修改：**

> redux定义了一个reducer函数来完成state数据的修改，reducer会接收先前的 state 和 action，并返回**新的** **state**。

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

#### 使用redux

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
import React, { Component } from 'react'
import store from './store'

class App extends Component {
  componentDidMount() {
    // 由于没有将数据存入state中，所以无法使用setState触发更新
    // 因此这里在检测到数据更新后直接使用forceUpdate强制更新页面
    this.unsubscribe = store.subscribe(() => this.forceUpdate())
    // store.subscribe(() => this.setState(state => store.getState()))
  }
  componentWillUnmount() {
    this.unsubscribe() // 取消订阅
  }

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

### react-redux

> React-Redux是Redux的官方针对React开发的**扩展**库，默认没有在React项目中安装，需要手动来安装。react-redux是依赖于redux，所以必须安装redux。
>
> 可以理解为react-redux就是redux给我们提供一些高阶组件，能解决的问题是：使用它以后我们不需要在每个组件中再去手动订阅数据的更新了。

#### 安装：

```js
npm i -S react-redux  redux
```

#### 使用：

**定义Provider：**

> 在程序主文件index.js文件中,定义Provider，并提供数据源，也就是store对象，可以猜测react-redux是通过生产消费者模式为给子组件提供数据的。
>
> 类似于之前跨组件通信处的Provider一样，旨在让全局的组件共享store中的数据。

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

**在子组件中使用react-redux：**

> connect用来把redux中的state或action映射到当前组件的props中：
>
> - 参数1：函数，把redux中的state数据映射到当前的props属性中
> - 参数2：函数|对象 ， 把你操作的dispatch方法映射到当前的props属性中
> - connect(mapStateToProps, mapDispatchToProps)
>
> mapStateToProps是用来规定要映射到props中的数据：
>
> - 返回值必须是一个json对象
> - 参数就是store.getState()
>
> mapDispatchToProps用于规定要映射到props中的action方法，它有两种写法：
>
> - 函数写法需要返回一个对象，并接收一个参数，就是store.dispatch
> - 对象写法直接列出要映射的action方法即可

```jsx
import React, { Component } from 'react'
import { connect } from 'react-redux'

const mapStateToProps = state => state

// 如果为对象方式则只能使用同步，因为如果是对象时，connect方法会主动调用dispatch
// dispatch方法显然不能接收一个Promise参数，因此会报错
const mapDispatchToProps = {
  add: (n = 1) => ({ type: 'add', payload: n })
}

// 函数的方式可以同步也可以异步，dispatch是你手动在需要的地方来调用
const mapDispatchToProps = dispatch => ({
  add(n = 1) {
    setTimeout(() => {
      dispatch({ type: 'add', payload: n })
    }, 1000)
  }
})

// @connect(state => state, mapDispatchToProps)
@connect(mapStateToProps, mapDispatchToProps)
class App extends Component {
  render() {
    console.log('props', this.props)
    return (
      <div>
        <h3>{this.props.num}</h3>
        <hr />
        <button onClick={() => this.props.add()}>++++</button>
      </div>
    )
  }
}

export default App
// export default connect(mapStateToProps, mapDispatchToProps)(App)
```

### redux模块化

> 使用redux库中提供的**combineReducers**方法，可以将多个拆分reducer函数合并成统一的reducer函数，提供给createStore来使用。
>
> combineReducers会把state进行模块化拆分，但它并不会把reducer函数中的type名称进行模块化命名，因此需要注意 type的类型名称不能重名，如果重名则都会执行。
>
> 拆分后的state：{ count: { num: 100 }, film: { nowplayings: [] } }
>
> 如果想要避免type的命名冲突，则需要规范type的命名，比如以拆分后的文件名称为前缀（xxx_type类型名）。

#### store数据源

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

#### 组件使用数据：

- **src/views/Count/connect.js**

```js
import { connect } from 'react-redux'
import countAction from '@/store/actionCreators/countAction'

const mapDispatchToProps = dispatch => ({
  add(n = 1) {
    dispatch({ type: 'count_add_num', payload: n })
  }
})

// 注意这里的mapStateToProps写的是state => state.count，因为使用了combineReducers后state进行了拆分
export default connect(state => state.count, mapDispatchToProps)
```

- **src/views/Count/index.jsx**

```jsx
import React, { Component } from 'react'
import connect from './connect'

@connect
class Count extends Component {
  render() {
    return (
      <div>
        <h3>{this.props.num}</h3>
        <button onClick={() => this.props.add()}>累加NUM</button>
      </div>
    )
  }
}

export default Count
```

### redux异步请求

- **src/views/films/connect.js**

```js
import { connect } from 'react-redux'
import { getNowPlayingFilmListApi } from '@/api/filmApi'

const mapDispatchToProps = dispatch => ({
  async add(page = 1) {
    // getNowPlayingFilmListApi是请求电影列表的api,会返回一个Promise
    let ret = await getNowPlayingFilmListApi(page)
    dispatch({ type: 'film_set_nowplayings', payload: ret.data.films })
  }
})

export default connect(state => state.film, mapDispatchToProps)
```

- **src/views/films/index.jsx**

```jsx
import React, { Component } from 'react'
import connect from './connect'

@connect
class Nowplaying extends Component {
  componentDidMount() {
    this.props.add()
  }
  render() {
    return (
      <div>
        {this.props.nowplayings.length === 0 ? (
          <div>加载中...</div>
        ) : (
          this.props.nowplayings.map(item => <div key={item.filmId}>{item.name}</div>)
        )}
      </div>
    )
  }
}

export default Nowplaying
```

### redux中间件

> Redux 中间件执行时机是**在 dispatching action 和 到达 reducer 之间**。也就是说，原本组件中执行store.dispatch会直接传递到reducer中，现在它会再经过所有的中间件后再去到reducer中。
>
> 中间件的原理就是封闭了redux自己的dispatch方法：
>
> - 没有中间件：store.dispatch() 就是 Redux 库自己提供的 dispatch 方法，用来发起状态更新
> - 使用中间件：store.dispatch() 就是中间件封装处理后的 dispatch，但是，最终一定会调用 Redux 自己的 dispatch 方法发起状态更新

- 没有中间件：**`dispatch(action) => reducer`**

![d57303e625c00c0509865edea7d9affc](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/d57303e625c00c0509865edea7d9affc.png)

- 使用中间件：**`dispatch(action) => 执行中间件代码 => reducer`**

![fdc7f48d1b87fe7ebbe753578a75dbba](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fdc7f48d1b87fe7ebbe753578a75dbba.png)

**实现一个logger中间件：**

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

#### redux-logger：

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

#### redux-thunk：

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

#### redux-saga：

> redux-saga 是 redux 一个中间件，它是基于ES6 的 **Generator** 功能实现，用于解决异步问题（让redux中可以直接进行异步操作）。

![1](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/1.png)

**安装：**

```js 
npm i -S redux-saga
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

// saga中间件 主saga，用于区别是否需要saga来处理异步操作，如果没有异步，则放行
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

在有了saga中间件后，再抛发action时，如果saga发现它是被监听的action（有异步请求的action），那它会被saga拦截，在异步请求完成之后，再在workSaga中抛发一个同步的action。

但是现在有一个问题，在asyncUserLogin中异步请求完毕后，抛发了一个同步action（userLogin），把请求到的数据同步到了redux中，同步完成后要进行页面跳转，那就要在login组件中通过路由对象来进行，那么如果才能在组件中得知redux中的数据改变了呢？

- **src/views/Login/index.jsx**

```js
import React, { useEffect } from 'react'
import { useDispatch, useSelector } from 'react-redux'

const Login = ({ history }) => {
  const dispatch = useDispatch()
  let uid = useSelector(state => state.user.uid)

  useEffect(() => {
    // uid初始值为0，只要登录成功，则一定会大于0，表示登录成功，跳转到首页
    if (uid > 0) history.push('/')
  }, [uid])

  const doLogin = () => {
    // 进行登录，它是一个异步任务，会交给saga，saga会完成异步操作
    // 然后saga通知reducer完成同步修改redux中的state数据改变
    // reducer把state中的数据修改后，由于当前的组件通过useEffect
    // 来监听它的变化，所以它只要变化了，就可以实现跳转，从而可以确认
    // redux中的数据一定是存在后才跳转的
    dispatch({ type: 'asyncLogin', payload: { username: 'admin', password: 'admin888' } })
  }

  return (
    <div>
      <h3>{uid}</h3>
      <button onClick={doLogin}>进入系统</button>
    </div>
  )
}

export default Login
```

### connected-react-router

> 这个库存可以让我们在redux中完成路由跳转相关的功能。这样就不需要在组件中监听redux中数据的变化再跳转路由了，可以在redux中直接跳转。
>
> 这个组件的原理就是把react-router信息同步到了redux中，当我们使用dispatch抛发connect-react-router模块提供的action，并进入`routerMiddleware`中间件后。
>
> 该中间件的作用就是识别路由操作相关action，执行history暴露的方法，比如push，在hash路由中就会执行`window.location.hash = to`，进而改变路由。React-Router内部会监听hash变化，重新渲染React组件。

**安装模块：**

```js
yarn add connected-react-router
```

**使用流程：**

- **src/history.js**

```js
// history模块它是react-router-dom安装成功后就存在的，无需手动再安装
import { createBrowserHistory, createHashHistory } from 'history'
const history = createBrowserHistory()
// 告知当前路由的模式为 history模式
export default history
```

- **src/index.js**

> 在入口文件中把原来的react-router-dom中定义路由模式组件更换

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
// 使用connected-react-router库，就需要把原来的路由模式组件进行更换
import { ConnectedRouter as Router } from 'connected-react-router'
import { Provider } from 'react-redux'

import history from './history'
import store from './store'
import App from './App'

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <App />
    </Router>
  </Provider>,
  document.getElementById('root')
)
```

- **src/store/reducer/index.js**

> 在reducer模块化中，定义一个router的模块

```js
import { combineReducers } from 'redux'
import { connectRouter } from 'connected-react-router'
import history from '@/history'
import user from './user'

export default combineReducers({
  router: connectRouter(history),
  user
})
```

- **src/store/index.js**

> 在redux入口文件中，以中间件的方式把connected-react-router包含到redux中

```js
import { createStore, applyMiddleware } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'
// 合并后的reducer
import reducer from './reducer'
// redux中的路由
import { routerMiddleware } from 'connected-react-router'
import history from '@/history'

const store = createStore(reducer, composeWithDevTools(applyMiddleware(routerMiddleware(history))))

export default store
```

- **src/store/saga.js**

> 在redux中通过抛发action的方式完成路由跳转

```js
import { takeEvery, put, all } from 'redux-saga/effects'
import { push, replace, goBack } from 'connected-react-router'
import { loginApi } from '@/api/userApi'

function* mainSaga() {
  yield watchSaga()
}

function* watchSaga() {
  yield takeEvery('asyncUserLogin', workSaga)
}

function* workSaga({ payload }) {
  let ret = yield call(loginApi, payload)
  if (ret.code === 0) {
    yield put({ type: 'userLogin', payload: ret.data })
    // 通过抛发action的方式完成路由跳转
    yield put(push('/'))
  }
}

export default mainSaga
```

### redux-thunk最佳实践

#### 目录结构：

> 这里省略了一些非必要的目录结构

```js
├─store
│  ├─actionCreators    
│  │  └─countAction.js    // 抛发action的方法
│  ├─mutations         
│  │  └─filmMutation.js   // mutation方法
│  ├─reducers          
│  │  ├─film.js           // 拆分开的reducer
│  │  └─index.js          // reducers的入口文件(自动导入)
│  └─index.js             // redux的入口文件 
├─typings
│  └─filmType.js          // 消除魔术字符串
└─views
    └─NowPlaying          // film组件
      ├─connect.js
      └─index.js
```

#### 源码解析：

- **src/store/actionCreators/countAction.js**

```js
import { getNowPlayingFilmListApi } from "@/api/filmApi";
import { FILM_SET_NOWPLAYINGS } from "@/typings/filmType";

// 这个函数它返回一个函数，这个函数就是要抛发的action（因为使用了redux-thunk，因此可以使用函数作为action）
// 函数中的参数是中间件注入过来dispatch方法(store.dispatch)
export const add = (page = 1) => async (dispatch) => {
  let ret = await getNowPlayingFilmListApi(page);
  dispatch({ type: FILM_SET_NOWPLAYINGS, payload: ret.data.films });
};
```

- **src/store/mutations/filmMutation.js**

```js
import { FILM_SET_NOWPLAYINGS } from '@/typings/filmType'
import { fromJS } from 'immutable';

// 使用策略模式，避免多次使用if判断
const mutations = {
  [FILM_SET_NOWPLAYINGS](state, data) {
    return state.set('nowplayings', fromJS(data))
  }
}

export default mutations
```

- **src/store/reducers/film.js**

```js
import mutations from '../mutations/filmMutation'
import { fromJS } from 'immutable';

// 使用immutable数据初始化state数据
const initState = fromJS({
  nowplayings: []
})

// export default (state = initState, action) => mutations[action.type] ? mutations[action.type](state, action.payload) : state

const reducer = (state = initState, action) => {
  // 判断一下mutations中是否有指定的action.type，避免报错
  if (mutations[action.type]) {
 	return mutations[action.type](state, action.payload)
  }
  return state;
}

export default reducer
```

- **src/store/reduces/index.js**

```js
import { combineReducers } from 'redux-immutable'
// 这里可以直接使用module.context进行自动导入
import film from './film'

// 注意这里使用的是redux-immutable中的combineReducers进行合并
const reducer = combineReducers({
  film
})
export default reducer
```

- **src/store/index.js**

```js
import { createStore, applyMiddleware } from "redux";
import { composeWithDevTools } from "@redux-devtools/extension";
import thunk from "redux-thunk";
import logger from "redux-logger";
import reducer from "./reducers";

let store;
// 开发与生产环境的判断，生产环境中要关闭开发者工具，提高安全性
process.env.NODE_ENV !== "development"
  ? (store = createStore(reducer, composeWithDevTools(applyMiddleware(thunk, logger))))
  : (store = createStore(reducer, applyMiddleware(thunk)));

export default store;
```

- **src/typings/filmType.js**

```js
export const FILM_SET_NOWPLAYINGS = 'film_set_nowplayings'
```

- **src/views/NowPlaying/connect.js**

> actions是一个对象 {a:funtion(){}}，因为它是ESModule导出的，并且通过别名导入（* as actions）
>
> 这也是为什么我们不直接用mapDispatchToProps的函数写法来写异步，而是要用对象写法配合redux-thunk来写异步
>
> 这其实是为了更好的拆分每个actionCreator（在文件中的体现就是导出一个个的函数，详见**src/store/actionCreators/countAction.js**）

```js
import { connect } from 'react-redux'
import * as actions from '@/store/actionCreators/filmAction'

// 由于使用了redux-immutable，所以这里state是一个immutable对象
export default connect(state => state.get('film').toJS(), actions)
```

- **src/views/NowPlaying/index.jsx**

```jsx
import React, { Component } from 'react'
import connect from './connect'

@connect
class Nowplaying extends Component {
  componentDidMount() {
    this.props.add()
  }
  render() {
    return (
      <div>
        {this.props.nowplayings.length === 0 ? (
          <div>加载中...</div>
        ) : (
          this.props.nowplayings.map(item => <div key={item.filmId}>{item.name}</div>)
        )}
      </div>
    )
  }
}

export default Nowplaying
```

### redux-saga最佳实践：

#### 目录结构：

> 这里省略了一些非必要的目录结构

```js
├─store
│  ├─watchSagas         
│  │  └─userSaga.js   // mutation方法
│  ├─reducers          
│  │  ├─user.js           // 用户页面的saga
│  │  └─index.js          // reducers的入口文件(自动导入)
│  ├─sagas.js             // saga中间件入口          
│  └─index.js             // redux的入口文件 
└─views
│   └─Login               // film组件
│      └─index.js
├─history.js              // 保存路由模式
└─index.js                // 入口文件 
```

#### 源码解析：

- **src/store/watchSagas/userSaga.js**

```js
import { put, takeEvery, call } from 'redux-saga/effects'
// 通过connected-react-router，可以完成在redux中实现跳转跳转功能
import { push, replace, goBack } from 'connected-react-router'
import { loginApi } from '@/api/userApi'

export default function* watchSaga() {
  yield takeEvery('asyncLogin', login)
}

function* login({ payload }) {
  let ret = yield call(loginApi, payload)
  // 得到的数据同步到redux中
  if (ret.code === 0) {
    // 通过reducer完成redux中的数据更新
    yield put({ type: 'userLogin', payload: ret.data })
    // 通过抛发action的方式来进行路由跳转
    yield put(push('/'))
  }
}
```

- **src/store/reducer/user.js**

```js
const initState = {
  uid: 0,
  token: '',
  nickname: ''
}

const reducer = (state = initState, { type, payload }) => {
  if ('userLogin' === type) return { ...state, ...payload }
  return state
}

export default reducer
```

- **src/store/reducer/index.js**

```js
import { combineReducers } from 'redux'
import { connectRouter } from 'connected-react-router'
import history from '@/history'
import user from './user'

// 这里不能用react-immutable的combineReducers，因为它会把state变成immutable对象，而我们需要的是一个普通的对象
// 否则connected-react-router会报错，原因是它不认识immutable对象，只认识普通对象
export default combineReducers({
  router: connectRouter(history),
  user
})
```

- **src/store/saga.js**

```js
import { takeEvery, put, all } from 'redux-saga/effects'
import userWatchSaga from './watchsagas/userSaga'

// 主saga，用于区别是否需要saga来处理异步操作，如果不是异步则放行
function* mainSaga() {
  // all方法，可以监听多个监听saga，它的功能和Promise.all方法一样
  yield all([
    userWatchSaga()
  ])
}

export default mainSaga
```

- **src/store/index.js**

```js
import { createStore, applyMiddleware } from 'redux'
import { composeWithDevTools } from '@redux-devtools/extension'
// 合并后的reducer
import reducer from './reducer'
// saga中间件
import createSagaMiddleware from 'redux-saga'
import mainSaga from './sagas'
// redux中路由
import { routerMiddleware } from 'connected-react-router'
import history from '@/history'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(reducer, composeWithDevTools(applyMiddleware(routerMiddleware(history), sagaMiddleware)))
// 运行saga
sagaMiddleware.run(mainSaga)

export default store
```

- **src/history.js**

```js
// history模块它是react-router-dom安装成功后就存在的，无需手动再安装
import { createBrowserHistory, createHashHistory } from 'history'
const history = createBrowserHistory()
// 告知当前路由的模式为 history模式
export default history
```

- **src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
// 使用connected-react-router库，就需要把原来的路由模式组件进行更换
import { ConnectedRouter as Router } from 'connected-react-router'
import { Provider } from 'react-redux'

import history from './history'
import store from './store'
import App from './App'

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <App />
    </Router>
  </Provider>,
  document.getElementById('root')
)
```



