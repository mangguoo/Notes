## react+ts

### 初始化react为ts环境：

```js
// 本地先安装好typescript编译器  npm i -g typescript
npm init react-app myapp 
```

用上述命名创建出来的react脚手架，与react+js创建出来的脚手架几乎相同，只是增加了几个文件：

- /src/react-app-env.d.ts：这是一个类型声明文件，与常规ts项目中的/src/types.d.ts文件的功能相同，他就是在ts文件中导入js文件时，ts会在这个文件中查找js代码的类型声明
- /tsconfig.json：这个文件会做为tsc编译器的配置文件（`jsconfig.json`是tsconfig.json的后代，`tsconfig.json` 是 TypeScript 的配置文件。`jsconfig.json`相当于`tsconfig.json`把`"allowJs"`属性设置为`true`）

**注意：**

- 在react+ts工程中，如果是组件或方法有jsx语法，则文件扩展名一定要是.tsx扩展名，否则编译报错（在react+js的项目中，组件.jsx也可以写成.js）
- 在react+ts项目中的所有.ts文件都会被编译为.js文件，然后再进行打包，因此如果要在react+ts项目中备份文件，它的扩展名尽量不要写.ts .tsx，因为项目中虽然没有引用它，但是它仍然会被编译，这会导致编译速度变慢，甚至可以导致报错

### tsx写类组件：

> 组件的children属性就是在双标签组件中写的内容，可以给显示注解ReactElement或any类型。也可以使用react提供的一个泛型类型PropsWithChildren对props的类型进行包裹，这个泛型类型会对传入的类型添加children属性的类型。

```tsx
import React, { Component, ReactElement, PropsWithChildren } from 'react'

// 给props添加自定义属性的类型
interface IProps {
  num: number
  setNum: () => void
  // children: ReactNode
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

### tsx写函数组件：

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

### 增量配置webpack：

- **craco.config.ts**

```ts
import { resolve } from "path";
import mockFn from "./mock";
import { Application } from "express";

type IType = {
  app: Application;
};
export default {
  webpack: {
    alias: {
      '@': resolve('./src')
    }
  },
  devServer: {
    open: false,
    port: 3000,
    proxy: {
      // '/api': {
      //   target: 'http://localhost:3000',
      //   changeOrigin:true,
      //   pathRewrite: { '^/api': '' }
      // }
    }
    // 第三方库中的一些方法，它里面的类型过于复杂，这时就可以用any
    setupMiddlewares: (mids: any, { app }: IType) => {
      mockFn(app)
      return mids
    }
  }
}
```

- **/mock/index.ts**

> 由于这里是在ts中使用express，并且使用了express中的类型，因此要定义express中导出api的类型，或者下载别人写好的：
>
> `npm install -D express @types/express` 

```ts
import type { Request, Response, Application } from 'express'

export default (app: Application) => {
  app.get('/api/users', (req: Request, res: Response) => {
    res.send({
      code: 0,
      msg: 'ok',
      data: [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' },
        { id: 3, name: '王五' }
      ]
    })
  })
}
```

**注意：**由于这两个文件都是使用ts文件来写的，因此他们都需要经过编译才能进行使用，因此需要在tsconfig.json文件把这两个文件加上到includes选项中：

- **/src/tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src", "craco.config.ts", "mock"]
}
```

### ts编写路由：

- **/src/index.tsx**

> 定义路由模式：`BrowserRouter`

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

import { BrowserRouter as Router } from 'react-router-dom'

ReactDOM
.createRoot(document.getElementById('root') as HTMLElement)
.render(
    <Router>
      <App />
    </Router>
)
```

- **/src/router/index.tsx**

> 内置类型`RouteObject`，标注配置式路由的类型

```tsx
import { RouteObject } from 'react-router-dom'
import Home from '@/views/Home'
import About from '@/views/About'
import Detail from '@/views/Detail'
import User from '@/views/User'

export default [
  {
    path: '/',
    element: <Home />
  },
  {
    path: '/about',
    element: <About />
  },
  {
    path: '/detail/:id',
    element: <Detail />
  },
  {
    path: '/user',
    element: <User />
  }
] as RouteObject[]
```

- **/src/app.tsx**

> 使用`RouteObject`定义配置式路由

```tsx
import React, { Fragment } from 'react'
import { useRoutes } from 'react-router-dom'

import routesRule from '@/router'

const App = () => {
  let routes = useRoutes(routesRule)

  return <Fragment>
    {routes}
  </Fragment>
}

export default App

```

- **/src/views/About/index.tsx**

> 编程式导航与声明式导航：`useNavigate`和`Link`

```tsx
import React from 'react'
import { useNavigate, Link } from 'react-router-dom'

const data = Array(10)
  .fill('')
  .map((item, index) => index + 1)

const About = () => {
  const navigate = useNavigate()
  return (
    <div>
      <h3>关于我们</h3>
      <hr />
      <button onClick={() => navigate('/')}>去首页</button>
      <hr />
      <ul>
        {data.map(item => (
          <li key={item}>
            <Link to={`/detail/${item}?title=新闻`} state={{ phone: '13111111110' }}>{`新闻 -- ${item}`}</Link>
          </li>
        ))}
      </ul>
    </div>
  )
}

export default About
```

- **/src/views/Detail/index.tsx**

> 获取路由信息：`useParams`, `useSearchParams`, `useLocation`

```tsx
import React from 'react'
import { useParams, useSearchParams, useLocation } from 'react-router-dom'

const Detail = () => {
  // 使用定义params的类型，在下面调用时会有提示
  const params = useParams<{ id: string }>()
  const [search] = useSearchParams()
  const location = useLocation()

  return (
    <div>
      <h3>详情页面</h3>
      <div>动态路由参数：{params.id}</div>
      <div>搜索参数：{search.get('title')}</div>
      <div>state数据：{location.state.phone}</div>
    </div>
  )
}

export default Detail
```

- **/src/views/Home/index.tsx**

> 声明式路由：`Link`

```tsx
import React from 'react'
import { Link } from 'react-router-dom'

const Home = () => {
  return (
    <div>
      <h3>首页页面</h3>
      <hr />
      <Link to="/about">关于</Link>
    </div>
  )
}

export default Home
```

### ts编写redux：

- **/src/index.tsx**

> 在上面路由例子的基础上扩充了reudx（增加了一个user组件用于编写redux范例）

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

import { BrowserRouter as Router } from 'react-router-dom'

// redux
import { Provider } from 'react-redux'
import store from '@/store'

ReactDOM
.createRoot(document.getElementById('root') as HTMLElement)
.render(
  <Provider store={store}>
    <Router>
      <App />
    </Router>
  </Provider>
)
```

- **/src/store/index.ts**

> store入口文件

```ts
import { configureStore } from '@reduxjs/toolkit'
import type { ThunkDispatch, Action, ThunkAction } from '@reduxjs/toolkit'
// 把每个slice文件放在组件同级目录，方便操作
import user from '@/views/User/userSlice'

const store = configureStore({
  reducer: {
    user
  }
})

// reduxjs/toolkit内置了thunk，因此它的dispatch方法可能是 Dispatch 或 ThunkDisptach 类型，因此我们必须把这两个类型统一下，然后使用统一的类型去给useDispatch定义dispatch的类型，否则在使用dispatch的时候可能会报错，因为它接收的是对象action类型，而不能接收函数action类型（redux-thunk赋予的能力）
// 自动整合Dispatch或ThunkDisptach
export type AppDispatchType = typeof store.dispatch

// 得到redux中根的state类型（ReturnType是一个内置类型，用于获取函数返回值）
export type RootStateType = ReturnType<typeof store.getState>

// 定义thunkDispatch的类型，这个类型用于注解函数action中thunkDispatch的类型
export type AppThunkDispatchType = ThunkDispatch<RootStateType, unknown, Action<string>>

// 这个类型用于直接注解函数action，自然就会帮函数action中的thunkDispatch参数注解类型
// 这里使用了一个泛型参数ReturnType，用于表示函数action的返回值，默认为void
export type AppThunkActionType<ReturnType = void> = ThunkAction<ReturnType, RootStateType, unknown, Action<string>>

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

- **/src/views/User/index.tsx**

> 使用重构的useAppDispatch, useAppSelector

```tsx
import React, { useEffect } from 'react'
import { setCount, fetchUser } from './userSlice'
import { useAppDispatch, useAppSelector } from '@/store/hooks'

const User = () => {
  const dispatch = useAppDispatch()
  let count = useAppSelector(state => state.user.count)
  let users = useAppSelector(state => state.user.users)

  useEffect(() => {
    dispatch(fetchUser())
  })

  return (
    <div>
      <h3>用户列表 -- {count}</h3>
      <button
        onClick={() => {
          dispatch(setCount(1))
        }}
      >
        ++++
      </button>
      <hr />
      <ul>
        {users.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  )
}

export default User
```

- **/src/views/User/typings.ts**

> 定义网络请求返回数据的类型

```ts
// 用户类型
export interface UserType {
  id: number
  name: string
}

// 网络请求返回的数据类型
export interface DataType<T> {
  code: number
  msg: string
  data: T
}

// state类型
export interface IUserState {
  count: number
  users: UserType[]
}
```

- /src/views/User/userSlice.ts

> user的slice文件，定义user的action和reducer

```ts
import { createSlice } from "@reduxjs/toolkit";
import type { Slice, PayloadAction } from "@reduxjs/toolkit";
import type { IUserState, UserType, DataType } from "./typings";
import type { AppThunkActionType, AppThunkDispatchType } from "@/store";

// 限制state的数据类型，通过toolkit内置的泛型类型
// const userSlice: Slice<IUserState> = createSlice({
//   name: 'user',
//   initialState: {
//     count: 1
//   },
//   reducers: {}
// })

// 另外一种限制state中数据类型的方法，使用断言
const userSlice = createSlice({
  name: "user",
  initialState: {
    count: 1,
    // 此数据通过网络请求得到
    users: [],
  } as IUserState,
  reducers: {
    // 通过toolkit内置的泛型类型注解action的类型，参数表示pyload的类型
    setCount(state, action: PayloadAction<number>) {
      state.count += action.payload;
    },
    setUsers(state, action: PayloadAction<UserType[]>) {
      state.users = action.payload;
    },
  },
});

// 导出非异步的对象action
export const { setCount, setUsers } = userSlice.actions;

// 函数action中的dispatch的类型，有三种写法：
// 写法一：着急时用时，先用any代替，让程序跑起来
export const fetchUser = () => async (dispatch: any) => {
   let ret: DataType<UserType[]> = await (await fetch('/api/users')).json()
   dispatch(setUsers(ret.data))
}

// 写法二：推荐写法，直接注解dispatch的类型，使用我们在入口文件中定义的thunkDispatch类型
export const fetchUser = () => async (dispatch: AppThunkDispatchType) => {
   let ret: DataType<UserType[]> = await (await fetch('/api/users')).json()
   dispatch(setUsers(ret.data))
}

// 写法三：使用toolkit提供的内置方法，直接注解函数action的类型
export const fetchUser = (): AppThunkActionType => async (dispatch) => {
  let ret: DataType<UserType[]> = await (await fetch("/api/users")).json();
  dispatch(setUsers(ret.data));
};

export default userSlice.reducer;
```

