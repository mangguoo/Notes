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
import { Provider } from 'react-redux'
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

这样的写法其实是有缺陷的，而且会造成很多混乱：

- 只要是被persistReducer方法包裹后的reducer都会被进行持久化，如果是这样的话，那么下面的这个例子中，auth被做了一次持久化，而被合并为root的auth和other又被做了一次持久化，这样在rehydrate的时候，root被水合其实auth就已经被水合了，而auth还会自己进行一次水合

- auth做了持久化，但是other并没有做持久化，而在root进行持久化和水合的时候，其实也把other给连带的给持久化了

- 虽然在root的持久化配置中用黑名单把auth禁用掉了，root虽然不会持久化auth，但是auth自己配置了持久了，因此auth的持久化并不会失效

但是这样做也有好处，因为这样可以在根的位置进行一些集中配置，这在有些时候非常有用

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

### Transforms

> 使用Transforms可以让我们自定义获得持久化和rehydrated的状态对象
>
> 当state被持久化时，它首先用JSON.stringify()进行序列化。如果state的某些部分不能映射到JSON对象，那么序列化过程就会出现一些错误。例如JSON中不存在javascript Set类型。在使用JSON.stringify()序列化Set时，它会被转换为一个空对象
>
> 下面是一个Transforms的例子，它成功地持久化了一个Set属性，它只是将其转换为一个数组并返回。通过这种方式，Set被转换为Array，这是一个 JSON中可识别的数据结构。当从持久化存储中提取出来时，数组将在保存到redux之前转换回Set

```ts
import { createTransform } from 'reduxjs-toolkit-persist';

const SetTransform = createTransform(
  // transform state并将其序列化和持久化
  (inboundState, key) => {
    return { ...inboundState, mySet: [...inboundState.mySet] };
  },
  // transform state being rehydrated
  (outboundState, key) => {
    // convert mySet back to a Set.
    return { ...outboundState, mySet: new Set(outboundState.mySet) };
  },
  // define which reducers this transform gets called for.
  { whitelist: ['someReducer'] }
);

export default SetTransform;
```

CreateTransform函数接受三个参数：

- 在state持久化之前被调用的“inbound”函数(可选)

- 在state重新水合之前被调用的“outbound”函数(可选)

- 一个配置对象，指定转换state中的哪些键(默认情况下不转换任何键)

为了生效，需要将转换添加到PersisReducer的配置对象中：

```ts
import storage from 'reduxjs-toolkit-persist/lib/storage';
import { SetTransform } from './transforms';

const persistConfig = {
  key: 'root',
  storage: storage,
  transforms: [SetTransform]
};
```

## ts类型声明

- **/src/index.tsx**

> 在上面路由例子的基础上扩充了reudx（增加了一个user组件用于编写redux范例）

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import router from '@/routes'
import { RouterProvider } from 'react-router-dom'
import { Provider } from 'react-redux'
import { PersistGate } from 'reduxjs-toolkit-persist/integration/react'
import store, { persistor } from '@/store'
import './locale/i18next.config'

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement)

root.render(
  <React.StrictMode>
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <RouterProvider router={router} />
      </PersistGate>
    </Provider>
  </React.StrictMode>
)
```

- **/src/store/index.ts**

> store入口文件

```ts
import { configureStore, combineReducers } from '@reduxjs/toolkit'
import type { ThunkDispatch, Action, ThunkAction, AnyAction, Reducer } from '@reduxjs/toolkit'
import logger from 'redux-logger'
import {
  persistStore,
  persistReducer,
  FLUSH,
  REHYDRATE,
  PAUSE,
  PERSIST,
  PURGE,
  REGISTER
} from 'reduxjs-toolkit-persist'
import storage from 'reduxjs-toolkit-persist/lib/storage'
import autoMergeLevel1 from 'reduxjs-toolkit-persist/lib/stateReconciler/autoMergeLevel1'
import common from './slices/common'

// * common reducer
const commonPersistConfig = {
  key: 'root',
  storage,
  // blacklist: []
  // whitelist: []
  stateReconciler: autoMergeLevel1
}

// 由于这里使用了持久化，reducer被持久化函数包裹返回后的return type变成了any
// 所以这里对持久化后的reducer进行了显示注解，把它的return type赋回应该有的值(reducer返回一个完整的state)
const commonReducer: Reducer<ReturnType<typeof common>, AnyAction> = persistReducer<
  ReturnType<typeof common>,
  AnyAction
>(commonPersistConfig, common)

// * other reducer
// ... ...

// * 合并reducer
const reducer = combineReducers({
  common: commonReducer
})

// * root reducer
const rootPersistConfig = {
  key: 'root',
  storage,
  stateReconciler: autoMergeLevel1
}

const rootPersistedReducer: Reducer<ReturnType<typeof reducer>, AnyAction> = persistReducer<
  ReturnType<typeof reducer>,
  AnyAction
>(rootPersistConfig, reducer)

const store = configureStore({
  reducer: rootPersistedReducer,
  devTools: process.env.NODE_ENV !== 'production',
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        /* ignore persistance actions */
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER]
      }
    }).concat(logger)
})

// reduxjs/toolkit内置了thunk，因此它的dispatch方法可能是 Dispatch 和 ThunkDisptach 类型，因此我们必须把这两个类型统一下，然后使用统一的类型去给useDispatch定义dispatch的类型，否则在使用dispatch的时候可能会报错，因为它接收的是对象action类型，而不能接收函数action类型（redux-thunk赋予的能力）
// 获取redux-toolkit中的store.dispatch的类型，稍后赋给react-redux的hook useDispatch
export type AppDispatchType = typeof store.dispatch
// 得到redux中根的state类型（ReturnType是一个内置类型，用于获取函数返回值）
export type RootStateType = ReturnType<typeof store.getState>

// 定义thunkDispatch的类型，这个类型用于注解函数action中thunkDispatch的类型
export type AppThunkDispatchType = ThunkDispatch<RootStateType, unknown, Action<string>>
// 这个类型用于直接注解函数action，自然就会帮函数action中的thunkDispatch参数注解类型
// 这里使用了一个泛型参数ReturnType，用于表示函数action的返回值，默认为void
export type AppThunkActionType<ReturnType = void> = ThunkAction<ReturnType, RootStateType, unknown, Action<string>>

// 用于持久化
export const persistor = persistStore(store)

export default store
```

- **/src/store/hooks.ts**

> 重构 useDispatch 和 useSelector

```ts
import { useDispatch, useSelector } from 'react-redux'
import type { TypedUseSelectorHook } from 'react-redux'
import { AppDispatchType, RootStateType } from '.'

// 使用泛型参数把useDispatch方法返回的dispatch进行注解，注解为在入口文件中定义的合并类型
export const useAppDispatch = () => useDispatch<AppDispatchType>()
// 使用内置泛型对useSelector进行注解，让他的返回值（state）变成根state的类型，这样我们写代码的时候就有提示了
export const useAppSelector: TypedUseSelectorHook<RootStateType> = useSelector
```

- **/src/store/slice/common.ts**

> user的slice文件，定义user的action和reducer

```ts
import { createSlice } from '@reduxjs/toolkit'
import type { PayloadAction } from '@reduxjs/toolkit'
import type { AppThunkActionType, AppThunkDispatchType } from '@/store'
import * as Cookies from '@/utils/cookie'
import * as httpSettings from '@/config/httpSettings'
import NCommonSlice = global.NCommonSlice
// import Router from '@/routes'

const initialState: NCommonSlice.ICommonState = {
  userInfo: {}, // 用户信息
  roleAuth: {}, // 钱包权限
  allocationRoleAuth: {}, // 配资权限
  isLogin: false, // 登录状态
  lan: 'en_US' // 当前语言
}

// 这里指定类型是为了让RootStateType更加具体，在写useSelector的时候有代码提示
const commonSlice = createSlice({
  name: 'common',
  initialState,
  reducers: {
    logOut(state, action: PayloadAction<undefined>) {
      state.isLogin = false
      state.userInfo = {}
    }
  }
})

export const { logOut } = commonSlice.actions

// 函数action中的dispatch的类型，有三种写法：
// 写法一：着急时用时，先用any代替，让程序跑起来
export const logOutAction = () => async (dispatch: any) => {
   // logOut的参数会被传给reducer的action.payload: 
   // {type: 'common/logOut', payload: undefined}
   // 这里logOut的类型就是上面定义的PayloadAction<undefined>
   dispatch(logOut())
}

// 写法二：推荐写法，直接注解dispatch的类型，使用我们在入口文件中定义的thunkDispatch类型
export const logOutAction = () => async (dispatch: AppThunkDispatchType) => {
   // ... ...
}

// 写法三：使用toolkit提供的内置方法，直接注解函数action的类型
export const logOutAction = (): AppThunkActionType => async (dispatch) => {
  // ... ...
};

export default commonSlice.reducer
```
