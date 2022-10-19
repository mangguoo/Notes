## Redux Toolkit

> 它是一个开箱即用的高效 Redux 开发工具集，是官方推荐今后在项目中使用的redux库，内置了immer、redux-thunk和redux-devtools等一系列实用的库。
>
> 它提供了一些工具来抽象redux设置过程中常见的用例，并包括一些有用的实用程序，简化用户的应用程序代码。

### 安装

```js
npm i -S @reduxjs/toolkit react-redux
```

### 使用范例：

- **src/store/index.js**

```js
import { configureStore } from '@reduxjs/toolkit'
import logger from 'redux-logger'
import countReducer from "./modules/count";
import userReducer from "./modules/users";

const reducer = {
  count: countReducer,
  user: userReducer,
}

const preloadedState = {
  count: {
    num: 10
  },
  user: {
    users: []
  },
}

const store = configureStore({
  reducer, // 合并的reduers
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger), // 增加中间件
  devTools: process.env.NODE_ENV !== 'production', // 是否开启devTools
  preloadedState, // 要传递给 ReduxcreateStore函数的可选初始状态值。
})
```

- src/store/modules/count.js

```js
import { createSlice } from '@reduxjs/toolkit'

const countSlice = createSlice({
  // 设置命名空间名称，这个名称要和configureStore中的reducer配置对象的key名称保持一致
  name: 'count',
  // 初始化数据源
  initialState: {
    num: 10
  },
  // 修改数据源的方法集合（同步操作）
  reducers: {
    // 组件中派发dispatch(setNum(2))，2就会给payload
    // state它就是proxy对象[immer]
    setNum(state, { payload }) {
      state.num += payload
    }
  }
})

// 把action导出给在组件中调用（redux toolkit直接提供了action creator，不需要再自己写）
export const { setNum } = countSlice.actions

// 把当前模块的reducer导入，集合到大的reducer中
export default countSlice.reducer
```

- **src/store/modules/users.js**

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

const userSlice = createSlice({
  name: 'user',
  initialState: {
    users: []
  },
  reducers: {
    setUsers(state, { payload }) {
      state.users = payload
    }
  },
  extraReducers: builder => {
    // fetchUser有三个生命周期状态，这里只取成功状态 fulfilled
    // payload中的数据就是fetchUser方法它return出来的数据
    builder.addCase(fetchUser.fulfilled, (state, { payload }) => {
      state.users = payload
    })
  }
})

export const { setUsers } = userSlice.actions

// 异步方案一：由于redux toolkit内置redux-thunk，所以可以在此处完成异步操作，然后抛发一个同步的action（异步转同步）
export const fetchThunkUser = () => async dispatch => {
  let ret = await (await fetch('/api/users')).json()
  dispatch(setUsers(ret.data))
}

// createAsyncThunk可以创建一个异步action，它的第一个参数是action type，第二参数是一个应该返回Pomise的回调函数
// 它根据传入的action type做为前缀生成Pomise生命周期action type（生命周期有三个阶段：pending，fulfilled，rejected），并返回一个thunk action creator（函数action）
// 异步处理方案二：通过redux toolkit提供的createAsyncThunk直接抛发一个异步的action，这种方案的优势就在于语法明确，只需要抛发一个action
export const fetchUser = createAsyncThunk('users/fetchUser', async page => {
  let ret = await (await fetch('/api/users?page=' + page)).json()
  return ret.data
})

export default userSlice.reducer
```

- **src/app.jsx**

```jsx
import React, { useEffect } from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { setNum } from '@/store/modules/count'
import { fetchThunkUser, fetchUser } from '@/store/modules/users'

const App = () => {
  const dispatch = useDispatch()
  const count = useSelector(state => state.count.num)
  const users = useSelector(state => state.user.users)

  useEffect(() => {
    // 这两种抛发异步action的解决方案都可以
    dispatch(fetchThunkUser())
    dispatch(fetchUser(1)) // 传值
  }, [])

  return (
    <div>
      <h3>{count}</h3>
      <button
        onClick={() => {
          dispatch(setNum(10))
        }}
      >
        ++++
      </button>
      <hr />
      {users.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </div>
  )
}

export default App
```

