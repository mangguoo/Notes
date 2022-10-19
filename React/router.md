### 路由使用

#### 安装路由模块：

> react-router-dom路由库，它路由相关的配置与vue大不相同，它是使用组件来配置路由信息。

```js
// react-router-dom 它现在最新的版本6
npm i -S react-router-dom@5
```

#### 相关组件：

- **路由模式组件**：包裹整个应用，一个React应用只需要使用一次
  - HashRouter： 使用URL的哈希值实现 （localhost:3000/#/first）
  - BrowserRouter：使用H5的history API实现（localhost3000/first）

- **导航组件**：用于指
  
- 定导航链接, 最终**Link会编译成a标签**
  - Link:  不会有激活样式
    - exact属性：开启严格匹配模式
  - NavLink：如果地址栏中的地址和to属性相匹配，则会有激活样式 (.active)，可以通过activeClassName来修改匹配成功后样式名称
    - exact属性：开启严格匹配模式

- **路由规则定义组件**：指定路由规则和对应**匹配成功后要渲染的组件**

  - Route：

    - **path**属性：路由路径，在地址栏中访问的地址

    - **component**属性：和规则匹配成功后渲染的组件 /render/children

![a](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/a.png)

#### 定义路由模式：

> 为了日后让当前项目中所有的组件都受到路由控制，定义在isrc/index.js中。

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './utils/init'

// BrowserRouter history路由模式，上线时，需要对nginx进行配置
import { BrowserRouter as Router, HashRouter } from 'react-router-dom'

ReactDOM.render(
  <Router>
    <App />
  </Router>,
  document.getElementById('root')
)
```

#### 定义路由规则：

> 路由规则组件可以定义在src/index.js文件中，也可以定义在src/App.jsx组件中。定义在哪个组件中，哪个组件就是主页。

```jsx
import React, { Component } from 'react'
import { Route, Link, NavLink } from 'react-router-dom'
import Home from './views/Home'
import About from './views/About'

class App extends Component {
  render() {
    return (
      <div>
        <h3>App组件</h3>
        <div>
          <NavLink exact to="/home">Home</NavLink>
          <NavLink to="/about">About</NavLink>
        </div>
        <hr />
        <Route path="/home" component={Home} />
        <Route path="/about" component={About} />
      </div>
    )
  }
}

export default App
```

#### 声明式导航：

> react默认的匹配规则为模糊匹配，而且它还会一直匹配到没有规则组件为止，这就会导致它可能会匹配多个路由，从而显示多个组件。
>
> 可以使用exact严格模式来解决这个问题，也可以使用Switch来解决，Switch 会使得多个路由规则只匹配一个， 但是要注意书写顺序，范围大的路由规则应该放在后面，否则会导致无法匹配到正确的路由。
>
> Redirect：重定向时使用它。他要配合Switch组件使用，否则可能会导致死循环。

```jsx
/src/app.jsx
import React, { Component } from 'react'
import { Route, Link, NavLink, Switch, Redirect } from 'react-router-dom'
import Home from './views/Home'
import About from './views/About'
import Notfound from './views/Notfound'

class App extends Component {
  render() {
    return (
      <div>
        <h3>App组件</h3>
        <div>
          <NavLink exact to="/home" activeClassName='test'>Home</NavLink>
          <NavLink to="/about">About</NavLink>
        </div>
        <Switch>
          <Route path="/home" component={Home} />
          <Route path="/about" component={About} />
          <Redirect exact from="/" to="/home" />
          <Route component={Notfound} /> // 404路由
        </Switch>
      </div>
    )
  }
}

export default App
```

#### 编程式导航：

> 它是通过history对象中的push/replace/goBack等方法实现编程式导航功能。

```jsx
/src/views/Home/index.jsx
import React, { Component } from 'react'
import Btn from './Btn'

class Home extends Component {
  jumpUrl = () => {
    this.props.history.push({
      pathname: '/about'
    })
  }

  render() {
    return (
      <div>
        <h3>首页展示</h3>
        <button onClick={this.jumpUrl}>home组件中回关于</button>
        <Btn {...this.props} />
      </div>
    )
  }
}

export default Home
```

> 注：在react路由中this.props要想得到路由中的对象，则默认必须要通过路由规则匹配渲染的组件才能有此对象，必须是直接渲染的组件。
>
> 如果想要在子组件中进行编程式跳转，那就必须在父组件中把history参数传给子组件。

```jsx
/src/views/Home/Btn.jsx
import React, { Component } from 'react'

class Btn extends Component {
  jumpUrl = () => {
    this.props.history.push('/about')
  }
  render() {
    return <button onClick={this.jumpUrl}>在btn组件中回关于</button>
  }
}

export default Btn
```

#### 路由传参：

> 路由参数：在Route定义渲染组件时给定动态绑定的参数。React路由传参方式有三种：
>
> - 动态路由参数（param）
> - 查询字符串（search）
> - 隐式传参（state），通过地址栏是观察不到的

**动态路由参数：**

> 以“/detail/:id”形式传递的数据，在落地组件中通过**this.props.match.params**得到

- 定义路由规则

```jsx
// id 和 name 都是params名称，而?表示可有可无，没有也会匹配
<Route path="/detail/:id/:name?" component={Detail} />
```

- 定义link跳转

```js
{Array(10)
  .fill('')
  .map((_, index) => (
    <li key={index}>
	  <Link to={`/detail/${index}?age=${2 + index}`}>新闻 -- {index}</Link>
    </li>
))}
```

- 在落地路由组件中获取params

```js
let id = this.props.match.params.id || 0
```

**查询字符串：**

> 通过地址栏中的 ?key=value&key=value传递，在落地组件中通过**this.props.location.search**得到

- 在落地路由组件中获取查询字符串

```js
# 方法一：使用nodejs的一个库
import qs from 'querystring'
qs.parse(this.props.location.search.slice(1))
```

```js
# 方法二：使用ES6中的URLSearchParams
const search = new URLSearchParams(this.props.location.search)

let searchData = {}
search.forEach((value, key) => {
  console.log(value, key)
  searchData[key] = value
})
```

```js
# 方法三：自定义一个字符串原型方法
String.prototype.toSearch = function () {
  let searchData = {}
  if (this.toString() != '') {
    // this有可能是一个字符串的包装对象
    const search = new URLSearchParams(this.toString())
    search.forEach((value, key) => {
      searchData[key] = value
    })
  }
  return searchData
}

this.props.location.search.toSearch()
```

**隐式传参（state）**

>  通过地址栏是观察不到的，它是通过路由对象中的state属性进行数据传递，可以在落地组件中通过**this.props.location.state**得到

```jsx
import React, { Component } from 'react'
import { Link } from 'react-router-dom'

class Home extends Component {
  jumpDetail = () => {
    this.props.history.push({
      pathname: '/detail',
      search: 'name=zhangsan',
      state: { id: 200 }
    })
  }
  render() {
    return (
      <div>
        <Link to={{ pathname: '/detail', state: { id: 100 }, search: 'name=lisi' }}>去详情</Link>
        <button onClick={this.jumpDetail}>去详情页</button>
      </div>
    )
  }
}

export default Home
```

```jsx
import React, { Component } from 'react'

class Detail extends Component {
  render() {
    console.log(this.props.location.state)
    console.log(this.props.location.search.toSearch())
    return (
      <h3>详情页面</h3>
    )
  }
}

export default Detail
```

#### 嵌套路由：

> 在有一些功能中，往往请求地址的前缀是相同的，不同的只是后面一部份，此时就可以使用多级路由（路由嵌套）来实现此路由的定义实现。
>
> 在定义嵌套路由时，父路由一定一定一定不能定义为严格模式（exact），但是它的子路由可以定义为严格模式。

```jsx
/src/app.jsx
import React, { Component } from 'react'
import { Route, Link, NavLink, Switch, Redirect } from 'react-router-dom'
import Login from './views/Login'
import Admin from './views/Admin'

class App extends Component {
  render() {
    return (
      <Switch>
        <Route path="/login" component={Login} />
        <Route path="/admin" component={Admin} />
      </Switch>
    )
  }
}

export default App
```

> 在嵌套路由匹配的落地组件中，可以使用**this.props.match.path**来获取到父路由的路径

```jsx
/src/views/Admin/index.jsx
import React, { Component } from 'react'
import { NavLink, Redirect, Route, Switch } from 'react-router-dom'
import { AdminContainer } from './style'
import Index from '../Index'
import User from '../User'

class Admin extends Component {
  render() {
    // 在子路由定义的组件中，可以通过props中提供的路由对象来获取父路由定义的地址
    let parentRoutePath = this.props.match.path

    return (
      <AdminContainer>
        <h3>后台首页</h3>
        <div>
          <ul>
            <li>
              <NavLink to={`${parentRoutePath}/index`}>后台首页</NavLink>
            </li>
            <li>
              <NavLink to={`${parentRoutePath}/user`}>用户列表</NavLink>
            </li>
          </ul>
          <div>
            <Switch>
              <Route path={`${parentRoutePath}/index`} component={Index} />
              <Route path={`${parentRoutePath}/user`}component={User} />
              <Redirect exact from={parentRoutePath} to={`${parentRoutePath}/index`} />
            </Switch>
          </div>
        </div>
      </AdminContainer>
    )
  }
}

export default Admin
```

#### 路由渲染方式：

**component：**

> component方式分为两种写法，分别是函数组件方式和类组件方式。

```jsx
/src/app.js
import RenderCmp from './views/Render'
/*
  类：
  1.对于规则匹配成的组件没有办法去写逻辑，会直接渲染
  2.规则匹配成功后，会给组件中的props自动映射对应的路由对象
  3.当前载体中的state更新,它不会重新创建
*/
<Route path="/" component={RenderCmp} />;
/* 
  函数：
  1.可以在规则匹配成功后，进行业务逻辑的判断来决定最终是否渲染
  2.规则匹配成功后，它需要你把回调函数中路由对象，手动的通过props传给要渲染的组件
  3.当前的载体中的state如果发生改变,则它对应匹配要渲染的组件会销毁并重新创建
  建议: component属性的回调函数的方式,不要在工作中使用,可以用 render来代替
*/
<Route
  path="/"
  // 由于这里是一个函数组件，因此可以接收Route到传过来的路由对象，
  component={(route) => {
    if (true) {
      return <RenderCmp {...route} />;
    }
    return <div>没有权限</div>;
  }}
/>;
```

> 如果使用函数路由渲染方式，他会销毁并重新渲染组件，可以通过componentDidMount生命周期函数来验证。

```jsx
/src/views/Render/index.jsx
import React, { Component } from 'react'

class Render extends Component {
  componentDidMount() {
    console.log('child -- componentDidMount');
  }
  render() {
    return (
      <div>
        <h3>渲染页面</h3>
      </div>
    )
  }
}

export default Render
```

**Render：**

> 它有component类的优点也有component回调的优点,但没有component回调中的缺点（不会销毁组件并重新创建）

```jsx
<Route
  path="/"
  render={route => {
	if (true) {
	  return <RenderCmp {...route} />
	}
	return <div>没有权限</div>
  }}
/>
```

> 案例，登录守卫

```jsx
/src/app.jsx
import React, { Component } from 'react'
import { Redirect, Route, Switch } from 'react-router-dom'
import RenderCmp from './views/Render'
import Login from './views/Login'

class App extends Component {
  state = {
    num: 100
  }
  render() {
    return (
      <div>
        <Route path="/login" component={Login} />
        <Route
          path="/render"
          render={route => {
            if (sessionStorage.getItem('uid')) {
              return <RenderCmp {...route} />
            }
            return <Redirect to="/login" />
          }}
        />
      </div>
    )
  }
}

export default App
```

```jsx
/src/views/Login/index.jsx
import React, { Component } from 'react'

class Login extends Component {
  render() {
    return (
      <div>
        <h3>用户登录</h3>
        <button
          onClick={() => {
            sessionStorage.setItem('uid', 1)
            this.props.history.push('/render')
          }}
        >
          登录一下
        </button>
      </div>
    )
  }
}

export default Login
```

**Children：**

```jsx
{/* 
  children属性中传入jsx元素,则它会根据path路径来匹配规则,如果匹配成功则渲染,不会自动注入路由对象到props中
  这种方式一般不会用到，因为它的功能完全可以使用component或render来代替，并且component和render还可以注入路由对象
*/}
<Route path="/home" children={<RenderCmp />} />
{/* 
  如果children属性为一个函数,则它的渲染不会根据路由规则来匹配渲染,也就是说不管什么路径它都会渲染
  此函数会接收一个参数,如果当前访问的地址和path路径一致,则这个参数为Router对象,否则为null
  应用场景: 如果当前页面中有一些元素在任何地址中都要显示,且还要根据是否是激活路由而高亮出来,就可以用它
*/}
<Route
  path="/home"
  children={route => {
	return route.match ? <RenderCmp {...route} /> : null
  }}
/>
```

#### withRouter高阶组件：

> 作用：赋予不是通过路由渲染出来的落地组件react-router 的 history、location、match 三个对象，与落地组件相同，这三个对象会传入props属性中。
>
> 默认情况下必须是经过路由匹配渲染的组件才拥有路由参数，使用编程式导航的写法执行this.props.history.push('/uri')跳转到对应路由的页面，然而不是所有组件都直接与路由相连的，当这些组件需要路由参数时，使用withRouter就可以给此组件传入路由参数。

```jsx
import React, { Component } from 'react'
import { Redirect, Route, Switch, withRouter } from 'react-router-dom'
import Home from './views/Home'
import Detail from './views/Detail'

@withRouter
class App extends Component {
  state = {
    num: 100
  }
  // this.props.history.listen方法的作用是监听路由变化，他可以当做一个全局路由守卫来使用，但是他有一个致使缺点，那就是把它放在componentDidMount中后，页面挂载完成后才会开始监听，这就导致第一次页面跳转他是监听不到的，那这样用户就可以直接通过输入的url的方式跳转来避开这个守卫，解决办法就是在componentDidMount中手动调用一次回调函数，并且要把location对象传入进行，这样它才能判断地址
  componentDidMount() {
    this.listenerRouter(this.props.location)
    this.props.history.listen(this.listenerRouter)
  }
  listenerRouter = ({ pathname }) => {
    if(pathname!=='/login'){
      // 判断是否登录
      return;
    }
  }
  render() {
    return (
      <div>
        <Switch>
          <Route path="/home" component={Home} />
          <Route path="/detail" component={Detail} />
        </Switch>
      </div>
    )
  }
}

export default App
```

#### 自定义导航组件：

```jsx
// Fragment 它可以当作顶层元素来包裹子元素,但它不会编译成html元素
// Fragment 有简写,简写时,可以不需要导入Fragment组件 <></>
import React, { Component, Fragment } from 'react'
import { withRouter, Route } from 'react-router-dom'
import types from 'prop-types'

@withRouter // 引用router对象
class Mylink extends Component {
  jumpUrl = () => {
    this.props.history.push(this.props.to)
  }

  render() {
    let { tag: Tag, children, to, activeClassName } = this.props
    return (
      <>
        // 这里使用了Children渲染方法，它只有精确匹配的时候都会在回调函数中传入router对象
        <Route
          path={to}
          children={({ match }) => {
            let className = match ? activeClassName : ''
            return (
              <Tag className={className} onClick={this.jumpUrl}>
                {children}
              </Tag>
            )
          }}
        />
      </>
    )
  }
}

Mylink.propTypes = {
  to: types.string.isRequired,
  tag: types.string,
  activeClassName: types.string
}

Mylink.defaultProps = {
  tag: 'a',
  activeClassName: 'active'
}

export default Mylink
```

