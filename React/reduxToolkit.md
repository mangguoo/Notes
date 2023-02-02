## Redux/Toolkit

> 它是一个开箱即用的高效 Redux 开发工具集，是官方推荐今后在项目中使用的redux库，内置了immer、redux-thunk和redux-devtools等一系列实用的库。
>
> 它提供了一些工具来抽象redux设置过程中常见的用例，并包括一些有用的实用程序，简化用户的应用程序代码。

### 安装

```shell
$ npm i -S @reduxjs/toolkit
```

使用React和Redux Toolkit启动新项目的推荐方法是使用create-react-app的官方Redux + JS模板，该模板集成了React Redux与React组件

```shell
$ npx create-react-app my-app --template redux
```

```shell
$ npx create-react-app my-app --template redux-typescript
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

## Redux-toolkit-persist

### 基础使用

- **src/store/index.js**

```js
import { combineReducers, configureStore, getDefaultMiddleware } from '@reduxjs/toolkit';
import { persistReducer, persistStore, persistCombineReducers, FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER } from 'reduxjs-toolkit-persist';
import storage from 'reduxjs-toolkit-persist/lib/storage'
import autoMergeLevel1 from 'reduxjs-toolkit-persist/lib/stateReconciler/autoMergeLevel1';
import countReducer from './reducers/count';
import anotherReducer from './reducers/another';

const persistConfig = {
  key: 'root',
  storage: storage,
  stateReconciler: autoMergeLevel1,
};

// 拆开的写法
const reducers = combineReducers({
  count: countReducer,
  another: anotherReducer
});
const _persistedReducer = persistReducer(persistConfig, reducers);

// 也可以合起来写
const _persistedReducer = persistCombineReducers(
  persistConfig,
  {
    count: countReducer,
    another: anotherReducer
  }
);

export const store = configureStore({
  reducer: _persistedReducer,
  middleware: getDefaultMiddleware({
    serializableCheck: {
      /* ignore persistance actions */
      ignoredActions: [
        FLUSH,
        REHYDRATE,
        PAUSE,
        PERSIST,
        PURGE,
        REGISTER
      ],
    },
  }),
});

export const persistor = persistStore(store)
```

- **src/store/count.ts**

```js
import {createSlice} from '@reduxjs/toolkit';

export interface CountState {
  count: number
};

const defaultState : CountState = {
  count: 0,
};

const slice = createSlice({
  name: 'count',
  initialState: defaultState,
  reducers: {
    increment: (state: CountState, action) => {
      state.count++;
    },
    decrement: (state: CountState, action) => {
      state.count--;
    }
  }
});

export const { increment, decrement } = slice.actions;

export default slice.reducer;
```

> 如果要使用react，需要使用PersisGate包装根组件。它可以延迟应用程序UI的呈现，直到检索到持久化状态并将其重新存回redux。loading选项可以是任何react实例，例如 load = { < Load/> }

- **src/App.jsx**

```jsx
import React from 'react'
import { PersistGate } from 'reduxjs-toolkit-persist/integration/react'
import { store, persistor } from './store'

const App = () => {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <RootComponent />
      </PersistGate>
    </Provider>
  );
};
```

### state协调

> 状态协调器定义传入的state如何与初始state合并。为我们的state选择正确的状态协调器是至关重要的:

- **hardSet**
  - **incoming state**: `{ foo: incomingFoo }`
  - **initial state**: `{ foo: initialFoo, bar: initialBar }`
  - **reconciled state**: `{ foo: incomingFoo }` 
- **autoMergeLevel1**(它是默认值)
  - **incoming state**: `{ foo: incomingFoo }`
  - **initial state**: `{ foo: initialFoo, bar: initialBar }`
  - **reconciled state**: `{ foo: incomingFoo, bar: initialBar }` // incomingFoo overwrites initialFoo
- **autoMergeLevel2** 
  - **incoming state**: `{ foo: incomingFoo }`
  - **initial state**: `{ foo: initialFoo, bar: initialBar }`
  - **reconciled state**: `{ foo: mergedFoo, bar: initialBar } `// initialFoo and incomingFoo are shallow merged

```js
import hardSet from 'reduxjs-toolkit-persist/lib/stateReconciler/hardSet'
import autoMergeLevel2 from 'reduxjs-toolkit-persist/lib/stateReconciler/autoMergeLevel2'
// autoMergeLevel1是默认值，因此不需要导入

const persistConfig = {
  key: 'root',
  storage,
  stateReconciler: hardSet,
}
```

### 白名单和黑名单

```js
// DENYLIST
const persistConfig = {
  key: 'root',
  storage: storage,
  blacklist: ['navigation'] // navigation will not be persisted
};

// ALLOWLIST
const persistConfig = {
  key: 'root',
  storage: storage,
  whitelist: ['navigation'] // only navigation will be persisted
};
```

### 嵌套持久化

> 对于包含不同的storage adapters、code splitting或者deep filtering，嵌套持久化可能非常有用。例如，虽然黑名单和白名单只能深入一层，但是我们可以使用嵌套的持久化来深入黑名单:

```js
import { combineReducers } from '@reduxjs/toolkit'
import { persistReducer } from 'reduxjs-toolkit-persist'
import storage from 'reduxjs-toolkit-persist/lib/storage'

import { authReducer, otherReducer } from './reducers'

const rootPersistConfig = {
  key: 'root',
  storage: storage,
  blacklist: ['auth']
}

const authPersistConfig = {
  key: 'auth',
  storage: storage,
  blacklist: ['somethingTemporary']
}

const rootReducer = combineReducers({
  auth: persistReducer(authPersistConfig, authReducer),
  other: otherReducer,
})

export default persistReducer(rootPersistConfig, rootReducer)
```

### 存储引擎

- **localStorage** `import storage from 'reduxjs-toolkit-persist/lib/storage'`
- **sessionStorage** `import storageSession from 'reduxjs-toolkit-persist/lib/storage/session'`
