# react-router@6

> **改变了什么：**
>
> - 推荐使用函数组件
>
> - 使用Routes替换原来的Switch组件
>
> - 使用Navigate组件替换原来的Redirect组件
>
> - Route组件必须包裹在Routes组件里
>
> - 路由规则是严格匹配，匹配成功就不向下继续匹配
>
> - 定义路由规则匹配渲染组件语法改变 component/render/children => element={<名称 />}
>
> - 新增了多个hook函数,useParams、useSearchParams、useNavigate、useRoutes等

## 安装路由：

```shell
$ yarn add react-router-dom@6
```

## 定义路由规则：

**在入口文件定义路由模式：**

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { BrowserRouter as Router } from "react-router-dom";

ReactDOM.createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <Router>
      <App />
    </Router>
  </Provider>
);
```

**在组件中定义路由规则：**

```jsx
import {  Routes,  Route } from "react-router-dom"

const App = () => {
  return (
    {/* 默认全部为严格模式的匹配 */}
    <Routes>
      {/* 嵌套路由，就是把定义规则的Route组件由单标签更换成双标签就可以 */}
      {/* 需要在父组件(Admin)中使用<Outlet />占位，这样子路由才能知道在哪里渲染 */}
      <Route path="/admin" element={<Admin />}>
        {/* 嵌套路由之默认页（加上index属性） */}
        <Route index element={<Navigate to="/admin/welcome" replace />} />
        {/* path中开头不要写 /  ,因为嵌套路由会自动把父组件中的path拼接过来 */}
        <Route path="welcome" element={<Welcome />} />
        <Route path="user" element={<User />} />
      </Route>
      <Route path="/home" element={<Home />} />
      <Route path="/about" element={<About />} />
      {/* 动态路由参数定义 */}
      <Route path="/detail/:id" element={<Detail />} />
      {/* 重定向 */}
      <Route path="/" element={<Navigate to="/home" replace />} />
      {/* 404页面处理 */}
      <Route path="*" element={<div>没有页面</div>} />
    </Routes>
  )
}

export default App
```

## 声明式导航：

> - to：指定跳转到的路由地址
>
> - replace：表示是否可以回退
>
> - end：是5版本中的exact严格匹配
>
> - 激活样式默认的名称为active,如果你想修改它，可以通过以下写法来改变
>
>   `className={isActive=>isActive?'aa':''}`
>
> - state：隐式传参的内容

```jsx
{/* 隐式传参 */}
<NavLink to="/" state={{ id: 1000 }}>
{/* 对于首页导航自定义激活样式名称 */}
<NavLink to="/" className={isActive => (isActive ? 'myactive' : '')}> 
{/* 
  默认情况下，当Home的子组件匹配成功，Home的导航也会高亮，
  当NavLink上添加了end属性后，若Home的子组件匹配成功，则Home的导航没有高亮效果。
*/}
<NavLink to="home" end >home</NavLink>
```

## useNavigate：

> 编程式导航，需要用到useNavigate这个hook函数来得到导航对象，完成编程导航。它有如下参数：
>
> - 参数1：string/number 
>   - 如果是string就是一个url地址
>   - 如果number就是回退的步数（正向前，负向后）
> - 参数2：object（可选）
>   - state: 隐式传递数据
>   - replace: 是否可回退

```jsx
import React from 'react'
import { NavLink, useNavigate } from 'react-router-dom'

const Home = () => {
  const navigate = useNavigate()

  const jumpUrl = () => {
    navigate('/about')
  }

  return (
    <div>
      <button onClick={jumpUrl}>跳转到关于页面</button>
    </div>
  )
}

export default Home
```

## useLocation,useParams,useSearchParams：

> - 动态路由参数：useParams获取动态路由参数
>
> - search字符串：useSearchParams获取search对象（返回一个URLSearchParams对象）
>
> - state隐式传递 (一般用于页面埋点)：useLocation获取location对象，可以得到state数据，得到pathname，得到search字符串等待

```jsx
import React from 'react'
import { useLocation, useParams, useSearchParams } from 'react-router-dom'

const Detail = () => {
  const location = useLocation()
  const params = useParams()
  // setSearch 可以动态的修改当前search字符串中的字段数据，一般用不到
  const [search, setSearch] = useSearchParams()

  return (
    <div>
      <h3>页面详情</h3>
      <div>动态路由参数id：{params.id}</div>
      <div>state中的数据：{location.state?.name}</div>
      <div>search字符串中的age：{search.get('age')}</div>
      <button
        onClick={() => {
          setSearch('?age=100')
        }}
      >
        修改search字符串
      </button>
    </div>
  )
}

export default Detail
```

## useRoutes：

```jsx
import React from 'react'
import { NavLink, useRoutes } from 'react-router-dom'
import routes from './router'

const App = () => {
  const routeElement = useRoutes(routes)

  return (
    <div>
      {routeElement}
    </div>
  )
}

export default App
```

```jsx
import { Navigate } from 'react-router-dom'

import Home from '@/views/Home'
import About from '@/views/About'
import Detail from '@/views/Detail'
import Admin from '@/views/Admin'
import Welcome from '@/views/Welcome'
import User from '@/views/User'

export default [
  { 
    path: '/', 
    element: <Navigate to="/home" replace /> 
  },
  {
    path: '/home',
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
    path: '/admin',
    element: <Admin />,
    children: [
      { 
        index: true, 
        element: <Navigate to="/admin/welcome" replace /> 
      },
      {
        path: 'welcome',
        element: <Welcome />
      },
      {
        path: 'user',
        element: <User />
      }
    ]
  }
]
```

## Routers

### createRoutesFromElements

> 该方法可以把<Route>元素转换为路由对象

```jsx
import {
  createBrowserRouter,
  createRoutesFromElements,
  Route,
  RouterProvider,
} from "react-router-dom";

const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<Root />}>
      <Route path="dashboard" element={<Dashboard />} />
      {/* ... etc. */}
    </Route>
  )
);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

### createBrowserRouter

> 这是所有React Router Web项目的推荐路由器。它使用[DOM History API](https://developer.mozilla.org/en-US/docs/Web/API/History)来更新和管理URL历史栈

```jsx
import * as React from "react";
import * as ReactDOM from "react-dom";
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "team",
        element: <Team />,
        loader: teamLoader,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

**Basename**

> 应用程序的基本名称，适用于无法将项目部署到域的根目录，而是部署到子目录的情况

```js
createBrowserRouter(routes, {
  basename: "/app",
});
```

链接到根时，尾部的斜杠将被遵守：

```jsx
createBrowserRouter(routes, {
  basename: "/app",
});
<Link to="/" />; // results in <a href="/app" />

createBrowserRouter(routes, {
  basename: "/app/",
});
<Link to="/" />; // results in <a href="/app/" />
```

### createHashRouter

> 如果无法配置Web服务器以将所有url访问重定向到React Router应用程序，则可以使用hash路由。因为它使用的是URL的散列 (#) 部分来管理“应用程序 URL”

```jsx
import * as React from "react";
import * as ReactDOM from "react-dom";
import {
  createHashRouter,
  RouterProvider,
} from "react-router-dom";

import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";

const router = createHashRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "team",
        element: <Team />,
        loader: teamLoader,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

### RouterProvider

> 所有路由器对象都可以传递给此组件以呈现应用程序并启用其余API

```jsx
<RouterProvider
  router={router}
  fallbackElement={<BigSpinner />}
/>
```

`fallbackElement`：如果app不是ssr，那么它将在打开时启动所有匹配的路由加载器。在此期间，我们可以提供一个`fallbackElement`给用户一些表明该应用程序正在运行的指示

## 路由懒加载

### 组件形式

```jsx
import React from 'react'
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom'
import Loading from './pages/loading'
import NoFound from './pages/no_found'

// React 组件懒加载
const Index = React.lazy(async () => await import('./pages/index'))
const About = React.lazy(async () => await import('./pages/about'))

export default function App() {
  return (
    <>
      <BrowserRouter>
        <div>
          <Link to="/index">
            <button>to: index</button>
          </Link>
          <Link to="/about">
            <button>to: about</button>
          </Link>
          <Link to="/aaa">
            <button>to: aaa</button>
          </Link>
        </div>
        <Routes>
          <Route
            path="/about"
            element={
              // React.Suspense.fallback 在懒加载文件没有加载完成之前显示的内容
              <React.Suspense fallback={<Loading />}>
                <About />
              </React.Suspense>
            }
          ></Route>
          <Route
            path="/index"
            element={
              <React.Suspense fallback={<Loading />}>
                <Index />
              </React.Suspense>
            }
          ></Route>
          <Route path="/:params" element={<NoFound />}></Route>
        </Routes>
      </BrowserRouter>
    </>
  )
}
```

### 路由表

```tsx
import React, { lazy, createElement } from 'react'
import { createBrowserRouter, type RouteObject, Navigate } from 'react-router-dom'

// 路由elemnt中使用了lazy方法懒加载，那么就一定要在使用router provider的外层包裹suspense组件
const routes: RouteObject[] = [
  {
    path: '/',
    element: <Navigate to="/custody" replace />
  },
  {
    path: '/custody',
    // lazy返回值的类型是React.LazyExoticComponent<React.FC<{}>>，但是element必须是React.ReactElement类型，因此这里需要使用createElement方法包裹一下，注意：import()方法中只能传入常量，不能传入变量
    element: createElement(lazy(async () => await import('@/App'))),
    children: [
      {
        index: true,
        element: <Navigate to="wallet" replace />
      },
      {
        path: 'wallet',
        element: createElement(lazy(async () => await import('@/pages/WalletList')))
      },
      {
        path: 'login',
        element: createElement(lazy(async () => await import('@/pages/Home')))
      }
    ]
  }
]

const router = createBrowserRouter(
  routes
  // ,{
  //   basename: '/custody'
  // }
)

export default router
```

