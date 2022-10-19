### Transition

>  react-transition-group是react的第三方模块，借住这个模块可以实现动画切换效果

```js
# npm 安装
npm install react-transition-group --save
```

#### Transiton：

> Transition 组件可以让一个组件随时间变化从一个状态转换到另一个状态。最常见的是它用于动画组件的安装和卸载。
>
> ------
>
> **注意**：`Transition`是一个基础组件（一般不会使用）。如果想要在 CSS 中使用过渡，可以使用CSSTransition。它继承了`Transition`所有特性，并且包含了与 CSS 过渡配合得很好所必需的附加特性。

#### CSSTransiton：

**timeout:** 

>  组件动画时长（必选） ，真实的动画时长还得看css的动画时间，它只能控制在指定的时间后加上相应的className

**in:** 

> 动画开关，进场或出场，它是一个boolean值（true进入 false退场）

**unmountOnExit:** 

> 退场后，删除动画dom元素

**classNames:**

> 指定动画的样式名称或样式定义的前缀名，可以是一个对象，也可以是一个字符串。 例如：`classNames="fade"`

- `fade-appear`, `fade-appear-active`, `fade-appear-done`
- `fade-enter`, `fade-enter-active`, `fade-enter-done`
- `fade-exit`, `fade-exit-active`, `fade-exit-done`

每个阶段的 classNames 也可以独立指定，例如：

```js
classNames={{
 appear: 'my-appear',
 appearActive: 'my-active-appear',
 appearDone: 'my-done-appear',
 enter: 'my-enter',
 enterActive: 'my-active-enter',
 enterDone: 'my-done-enter',
 exit: 'my-exit',
 exitActive: 'my-active-exit',
 exitDone: 'my-done-exit',
}}
```

如果在 CSS 模块中设置这些类：

```js
import styles from './styles.module.css';
```

可以在 CSS 文件中使用 camelCase，这样就可以直接展开，而不是一一列出：

```js
classNames={{ ...styles }}
```

**示例：**

```jsx
import React, { Component } from 'react'
import { CSSTransition } from 'react-transition-group'
import 'animate.css'
import './style/transistion.css' // 放在animate.css后面

class App extends Component {
  state = { show: true }

  render() {
    const { show } = this.state
    return (
      <div>
        <button onClick={() => this.setState(state => ({ show: !state.show }))}>显示/隐藏</button>
        <CSSTransition
          in={show}
          timeout={300}
          classNames={{
            enter: 'animate__animated',
            exit: 'animate__animated',
            enterActive: 'animate__fadeIn',
            exitActive: 'animate__fadeOut'
          }}
          unmountOnExit
        >
          // 这里要把div设为行内元素，因为块级元素会占一整行，动画效果会受到影响
          <div style={{ display: 'inline-block' }}>
            <h3>我是内容</h3>
          </div>
        </CSSTransition>
      </div>
    )
  }
}

export default App
```

> 重置animate.css动画的时长，与CSSTransition的timeout保持一致

```css
:root {
  --animate-duration: 300ms;
  --animate-delay: 300ms;
}
```

#### TransitionGroup：

> `<TransitionGroup>`组件管理列表中的一组`<Transition>`或`<CSSTransition>`组件。与Transition组件一样，TransitionGroup是一个状态管理器，用于管理组件的安装和卸载。只要TrantionsGroup组件中增加或者减少了一个CSSTransiton组件，它就会执行该组件相应的过渡动画，不需要我们再手动改变CSSTransition组件的in属性。
>
> 使用了TransitionGroup组件，则里面的CSSTransition属性in无效，用要唯一不重复的key来代替换。

```jsx
import React, { Component } from 'react'
import { CSSTransition, TransitionGroup } from 'react-transition-group'
import './style/transistion.css'

class App extends Component {
  state = { todos: [] }

  render() {
    const { todos } = this.state
    return (
      <div>
        <TransitionGroup>
          {todos.map((item, index) => (
            <CSSTransition key={index + ''} timeout={300} classNames="fade" unmountOnExit>
              <div style={{ width: '300px' }}>
                <span>{item}</span>
                <span
                  onClick={() => {
                    this.setState(state => ({
                      todos: state.todos.filter(val => val != item)
                    }))
                  }}
                >-- 删除</span>
              </div>
            </CSSTransition>
          ))}
        </TransitionGroup>
        <button
          onClick={() => {
            this.setState(state => ({
              todos: [...state.todos, Date.now() + '']
            }))
          }}
        >添加</button>
      </div>
    )
  }
}

export default App
```

#### SwitchTransition：

> 受vue 过渡模式启发的过渡组件。它可以实现组件切换的效果。如果想在SwitchTransition中有过渡动画，必须要key属性。SwitchTransition有两个模式：
>
> - `out-in`：`SwitchTransition`等待直到oldChild离开，然后插入一个newChild
> - `in-out`：`SwitchTransition`先插入一个newChild，等待newChild进入，然后移除oldChild
>
> **注意**：如果希望动画同时发生（即同时删除旧的孩子并插入新的孩子**）**，您应该使用 `TransitionGroup`

```jsx
import React, { Component } from 'react'
import { CSSTransition, SwitchTransition } from 'react-transition-group'
import './style/transistion.css'

class App extends Component {
  state = { on: true }

  render() {
    const { on } = this.state
    return (
      <div>
        // key属性发生变化时，switchTransition会认为是一个组件被添加，一个组件被摧毁，然后分别执行两个组件的进入和消失的过渡动画
        <SwitchTransition mode='out-in'>
          <CSSTransition key={on ? 'a' : 'b'} timeout={300} classNames="fade">
            <h3 style={{ display: 'inline-block' }}>{on ? '你好' : '再见'}</h3>
          </CSSTransition>
        </SwitchTransition>
        <hr />
        <button
          onClick={() => {
            this.setState(state => ({
              on: !state.on
            }))
          }}>添加任务</button>
      </div>
    )
  }
}

export default App
```

#### TransitionGroup实现路由过渡：

> this.props.location.key可以当做是路由对象的唯一标识，把它做为CSSTransition组件的key，就可以实现路由组件间的过渡

```jsx
import React, { Component } from 'react'
import { Switch, Route, Link, withRouter } from 'react-router-dom'
import { CSSTransition, TransitionGroup } from 'react-transition-group'
import RenderCmp from './views/Render'
import Login from './views/Login'

@withRouter
class App extends Component {
  render() {
    return (
      <div>
        <Link to="/login">login</Link> --
        <Link to="/render">render</Link>
        <hr />
        <TransitionGroup>
          <CSSTransition timeout={300} key={this.props.location.key} classNames="router">
            <Switch>
              <Route path="/login" component={Login} />
              <Route path="/render" component={RenderCmp} />
            </Switch>
          </CSSTransition>
        </TransitionGroup>
      </div>
    )
  }
}

export default App
```

#### 高阶组件实现路由过渡：

```js
/src/config/router/settings.js
import Home from "@/views/Home";
import Category from "@/views/Category";
import CookMap from "@/views/CookMap";
import Center from "@/views/Center";
import Detail from "@/views/Detail";
import Login from "@/views/Login";

const routerConfig = [
  {
    path: "/home",
    component: Home,
  },
  {
    path: "/category",
    component: Category,
  },
  {
    path: "/map",
    component: CookMap,
  },
  {
    path: "/center",
    component: Center,
  },
  {
    path: "/detail/:id",
    component: Detail,
  },
  {
    path: "/login",
    component: Login,
  },
];

export default routerConfig;
```

使用children渲染方式来渲染路由组件，因为children方式会无视路由规则，根据条件自定义条件来渲染路由，并且我们可以根据router.match来判断当前落地组件是否是精准匹配，从而来让这些组件执行进场和离场动画。

```jsx
/src/app.jsx
import React, { Component } from "react";
import { Redirect, Route, withRouter } from "react-router-dom";

import NotFound from "@/views/NotFound";
import routerConfig from "@/config/router/settings";

@withRouter
class App extends Component {
  routes = routerConfig.map((item) => item.path.split("/")[1]);
  render() {
    return (
      <Fragment>
        <Redirect exact from="/" to="/home" />
        {routerConfig.map((item) => (
          <Route key={item.path} path={item.path} children={(router) => <item.component {...router} />} />
        ))}
        <Route
          path="*"
          children={(router) => {
            if (!this.routes.includes(router.match?.url.split("/")[1])) {
              return <NotFound {...router} />;
            }
            return null;
          }}
        />
      </Fragment>
    );
  }
}

export default App;
```

```jsx
/src/views/Login/index.jsx
import React, { Component } from 'react'
import withTransition from '../../hoc/withTransition'

@withTransition
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
        >登录一下</button>
      </div>
    )
  }
}

export default Login
```

```js
/src/hoc/withTransition.js
import React, { Component } from "react";
import { CSSTransition } from "react-transition-group";
import { RouterTransition } from "@/assets/style/routerTransition.js";

const withTransition = (Cmp) => {
  return class extends Component {
    render() {
      return (
        <RouterTransition>
          <CSSTransition in={this.props.match ? true : false} timeout={300} classNames="fade" unmountOnExit>
            {this.props.match ? <Cmp {...this.props} /> : <div></div>}
          </CSSTransition>
        </RouterTransition>
      );
    }
  };
};

export default withTransition;
```



