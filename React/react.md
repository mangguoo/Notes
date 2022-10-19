### React入门

#### 初体验：

> 1. 引入2个核心库
>    - react.js 创建虚拟dom和提供react开发操作的若干API方法 -- 跨平台
>    - react-dom.js 把虚拟dom转为真实的html的dom对象，渲染到视图节点中
> 2. 在页面中定义挂载的节点  id="root"
> 3. 定义要渲染的组件(虚拟dom)  React.createElement
> 4. 调用ReactDOM.render方法来把组件转为真实的dom，渲染到页面挂载点中

#### react组件分类：

> - 类组件：又叫状态组件 、 容器组件
> - 函组件：又叫无状态组件、ui组件

#### 组件定义:

- **函数组件**

  > 如果安装了jsx插件，则可以通过 【rfc】 完成创建；
  >
  > 注意事项：
  >
  > 1. 函数名称，首字母大写
  > 2. 必须要有jsx/null返回值，如果是jsx则一定要有顶层元素包裹

  ``` jsx
  import { Component } from 'react'
  
  const App = () => {
    return (
      <div>
        <h3>我是一个App函数组件</h3>
      </div>
    )
  }
  
  export default App
  ```

- **类组件**

  > 如果安装了jsx插件，则可以通过 【rcc】来创建类组件
  >
  > 1. 要以es6的定义类来定义 ，必须以class定义类
  > 2. 此类必须要继承父类[Component]
  > 3. 此类必须要有一个成员方法 render，此方法中必须要返回jsx/null,如果是jsx必须要有顶层元素

  ```jsx
  import { Component } from 'react'
  
  class App extends Component {
    render() {
      return (
        <div>
          <h3>App之类组件</h3>
        </div>
      )
    }
  }
  
  export default App
  ```

#### 组件传参：

- **函数组件**

  > 通过自定义属性，可以向子组件传递数据 ，此数据它是单向数据流，子组件不能直接修改
  >
  > 子组件通过入参 props来接收父组件传过来的数据，props它是一个对象

  ```jsx
  import { Component } from 'react'
  
  const Child = ({ title = '默认值', fn }) => {
    console.log(fn())
    return (
      <div>
        <h3>Child组件 -- {title}</h3>
      </div>
    )
  }
  
  const App = () => {
    const fn = () => 'fn'
    return (
      <div>
        <Child title="hello" num={100} obj={{ id: 1 }} fn={fn} />
      </div>
    )
  }
  
  export default App
  ```

- **类组件**

  > 类组件中，子组件它是通过成员属性this.props来接受父组件的传值

  ```jsx
  import { Component } from 'react'
  
  class Child extends Component {
    render() {
      let { title = '默认值', fn } = this.props
      console.log(fn())
      return (
        <div>
          <h3>Child组件 -- {title}</h3>
        </div>
      )
    }
  }
  
  class App extends Component {
    // 柯理化，在react中很常见
    fn = arg => () => 'fn@@@ -- ' + arg
    render() {
      return (
        <div>
          <h3>App</h3>
          <Child title="我是标题" fn={this.fn(200)} />
        </div>
      )
    }
  }
  
  export default App
  ```

#### 事件绑定：

- **函数组件**

  ```jsx
  import React, { Component } from 'react'
  
  // 函数组件中没有this指向问题
  const App = () => {
    let num = 1
    const clickFn1 = () => {
      console.log(num++)
    }
    const clickFn2 = () => () => {
      console.log(num++)
    }
    return (
      <div>
        {/* 普通的值，它不具备响应式，所以你修改数据后，视图是不会更新的 */}
        <h3>App组件 -- {num}</h3>
        {/* 一般情况下，绑定的实现方法是不会加小括号，如果要加小括号，则定义绑定方法时，这个方法应该返回一个函数 */}
        <button onClick={clickFn1}>++++</button>
        <button onClick={clickFn2()}>++++</button>
      </div>
    )
  }
  
  export default App
  ```

- **类组件**

  ```jsx
  import React, { Component } from 'react'
  
  class App extends Component {
    num = 100
    todos = []
  
    // 建议使用 箭头函数定义成员属性，因为这个方法被绑定给了button的click事件，如果使用function定义的函数，那么它的this一定会指向全局，而如果使用箭头函数，由于箭头没有this指向，因此它会向上去找this，从而指向正确的组件对象
    addNum = () => {
      // 同步修改成员属性中的数据，但是视图不更新
      this.num++
      // 类组件中提供一个方法，可以强制让视图更新
      this.forceUpdate()
    }
  
    // react会在没有写小括号的时候会主动的把event对象传给你
    onEnter = evt => {
      if (evt.keyCode === 13) {
        this.todos.push({
          id: Date.now(),
          title: evt.target.value.trim(),
          done: false
        })
        evt.target.value = ''
        this.forceUpdate()
      }
    }
  
    render() {
      return (
        <div>
          <h3>{this.num}</h3>
          <ul>
            {this.todos.map(item => (
              <li key={item.id}>{item.title}</li>
            ))}
          </ul>
          {/* 类组件中绑定方法，有this指向问题 */}
          <button onClick={this.addNum}>++++</button>
          <input type="text" onKeyUp={this.onEnter} />
        </div>
      )
    }
  }
  
  export default App
  ```

#### $$typeof：

```jsx
<marquee bgcolor="#ffa7c4">hi</marquee>
```

jsx语法其实是一种语法糖，其中的每一个自定义标签都会被编译为一个React对象，通过一个叫createEement的方法。

```jsx
React.createElement(
  /* type */ 'marquee',
  /* props */ { bgcolor: '#ffa7c4' },
  /* children */ 'hi'
)
```

通过这个方法创建出来的React对象大概长这样：

```jsx
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

可是看到这个方法创建出来的React对象中有一个`$$typeof`属性，这个属性是干什么的呢？其实它是用于防止`XSS`攻击的。

如果你的服务器有允许用户存储任意 `JSON` 对象的漏洞，而前端需要一个字符串，这可能会发生一个问题，那就是用户可以写一个React对象，并写一些危险代码存储到服务器中，这样这个React对象就有可能被渲染到每一个访问这个网站的用户界面上，这就是`XSS`攻击。

因此React在迭代版本后给React元素中加入一个`$$typeof`属性，里面存储了一个`Symbol`值，因为JSON不支持 `Symbol` 类型。所以即使服务器存在用`JSON`作为文本返回安全漏洞，`JSON` 里也不包含`Symbol.for('react.element')`。而React 在渲染时这个元素中是否存在 `element.$$typeof`，如果没有，则会拒绝处理该元素，从而防止了`XSS`攻击。

### 基础语法

#### React事件处理：

> React 事件的命名采用小驼峰式，而不是纯小写(onChange)

**合成事件**

在react中，所有的事件都是合成事件，而不是原生事件。React合成事件是React模拟原生DOM事件所有能力的一个事件对象。根据 W3C规范来定义合成事件，兼容所有浏览器，拥有与浏览器原声事件相同的接口。合成事件除了拥有和浏览器原生事件相同的接口，包括stopPropagetion()和preventDefault()。

在React中，所有事件都是合成的，不是原生DOM事件，可以通过 event.nativeEvent 属性获取原生DOM事件。合成事件不会映射到原生事件。

这样做有很多优点，比如它解决了跨平台问题，抹平了浏览器差异和平台之间的差异。并且也提供了性能优化。因为它使用事件代理统一接收原生事件的触发，从而可以使得真实 DOM 上不用绑定事件。React 挟持事件触发可以知道用户触发了什么事件，是通过什么原生事件调用的真实事件。这样可以通过对原生事件的优先级定义进而确定真实事件的优先级，再进而可以确定真实事件内触发的更新是什么优先级，最终可以决定对应的更新应该在什么时机更新。

```jsx
export default class extends React.Component {
   fn = (num, evt) => {
      console.log(num, evt);
   }

    render(){
        return (
            <div>
                {* 手动传递合成事件的事件对象 *} 
                <button onClick={evt => this.fn(200, evt)}>click</button>
            </div>
        )
    }
}

```

**this指向问题**

1. 使用bind解决

   ```jsx
   import React, { Component } from 'react';
   
   class App extends Component {
     // 类的构造函数，它只执行1次，在初始化
     constructor(props) {
       super(props);
       this.num = 100
       this.addNum = this.addNum.bind(this)
     }
   
     addNum(evt, n = 1) {
       this.num += n
       this.forceUpdate()
     }
   
     render() {
       return (
         <div>
           <button onClick={this.addNum}>累加</button>
         </div>
       );
     }
   }
   
   export default App;
   
   ```

2. 使用箭头函数解决

   ```jsx
   import React, { Component } from 'react';
   
   class App extends Component {
     num = 100
     addNum = (evt, n = 1) => {
       this.num += n
       this.forceUpdate()
     }
   
     render() {
       return (
         <div>
           <h3>{this.num}</h3>
           <button onClick={evt => this.addNum(evt, 2)}>累加</button>
         </div>
       );
     }
   }
   
   export default App;
   ```

#### state状态：

> state状态只在**class类组件才有**，函数组件没有此功能。成员属性state它是一个特殊的属性，它是当前类的私有数据，只有在当前的组件中才能操作里面的数据，针对于此属性，react提供一个专门的修改数据的方法  this.setState ，用来修改当前属性中的方法，该方法可以触发当前state中的数据变化，和视图更新。
>
> 1. 状态(state)即数据，是组件内部的**私有数据**，只能在组件内部使用,和vue中data差不多，不过它没有像vue中的data进行了数据劫持
> 2. state的值是**对象**，表示一个组件中可以有多个数据
> 3. 通过**this.state**来获取状态，react中没有做数据代理
> 4. state数据值同时让视图更新则需要调用专用的方法 **this.setState**
> 5. state它是类的一个成员属性

```js
import React, { Component } from 'react';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      num: 100
    }
  }

  addNum(evt, n = 1) {
    // 同步修改数据，它不会触发视图更新
    this.state.num += n
  }

  render() {
    return (
      <div>
        <h3>{this.state.num}</h3>
        <button onClick={evt => this.addNum(evt, 2)}>累加</button>
      </div>
    );
  }
}

export default App;
```

**state状态的同步与异步改变**

> setState本身并不是异步，但是react为了性能优化，使用state在某些情况下体现为异步。比如在react的生命周期函数或者作用域下为异步，在原生的环境下为同步。
>
> 总结：setState既可以是异步也可以是同步。

setState语法（可以传入一个对象，也可以传入一个函数）：

```js
this.setState({key:value}|state=>({key:value}),()=>{})
```

当使用对象方式更新state状态时，react会把这个操作放进一个任务队列中，当这个任务被真正执行时，会执行传入setState中的回调函数：

```js
this.setState({
   num: this.state.num + 1
 }, () => {
   console.log(this.state.num);
})
```

如果执行了多次setState，react会把相同的操作进行合并，下面addNum被执行后num会被+3：

```js
addNum(evt, n = 1) {
  this.setState({
    num: this.state.num + 1,
  });
  this.setState({
    num: this.state.num + 2,
  });
  this.setState({
    num: this.state.num + 3,
  });
}
```

如果为了保证数据完整性，不想让他进行合并，可以使用函数语法：

```js
addNum(evt, n = 1) {
  this.setState((state) => ({
    num: state.num + 1,
  }));
  this.setState((state) => ({
    num: state.num + 2,
  }));
  this.setState((state) => ({
    num: state.num + 3,
  }));
}
```

在原生环境中，react不会把setState放进任务队列中，因此在原生环境中，setState操作是同步的：

```js
addNum(evt, n = 1) {
  setTimeout(() => {
    this.setState({
      num: this.state.num + 3,
    });
    console.log(this.state.num); // 可以直接读取正确数据
  }, 0);
}

componentDidMount() {
  document.onclick = () => {
    this.setState({
      num: this.state.num + 3,
    });
    console.log(this.state.num); // 可以直接读取正确数据
  };
}
```

> 小技巧：
>
> ```js
> // 如果你setState写了一个空对象，则它会更新视图一次
> this.setState({})
> // 如果你写了为null,setState不会做任何事件，相当于没有调用
> this.setState(null)
> ```

**todoList案例**

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    todos: []
  }
  onEnter = evt => {
    if (evt.keyCode === 13) {
      let title = evt.target.value.trim()
      this.setState(state => ({
        todos: [...state.todos, { id: Date.now(), title, done: false }]
      }), () => evt.target.value = '')
    }
  }
  del = tid => () => {
    this.setState(state => ({
      todos: state.todos.filter(({ id }) => id != tid)
    }))
  }
  delIndex = index => () => {
    this.state.todos.splice(index, 1)
    this.setState({})
  }

  render() {
    return (
      <div>
        <div>
          <input type="text" onKeyUp={this.onEnter} />
        </div>
        <hr />
        <ul>
          {
            this.state.todos.map((item, index) => (
              <li key={item.id}>
                <span>{item.title}</span>
                <span onClick={this.del(item.id)}>删除</span>
                <span onClick={this.delIndex(index)}>删除</span>
              </li>
            ))
          }
        </ul>
      </div>
    );
  }
}

export default App;
```

**购物车案例**

```js
import React, { Component } from 'react';

class App extends Component {
  state = {
    carts: [
      { id: 1, name: '水果手机14', price: 1, num: 1 },
      { id: 2, name: '大米手机14', price: 1, num: 2 },
      { id: 3, name: '一般手机14', price: 1, num: 3 },
    ]
  }
  setNum = (n, index) => {
    this.state.carts[index]['num'] += n
    if (this.state.carts[index]['num'] <= 1) this.state.carts[index]['num'] = 1
    if (this.state.carts[index]['num'] >= 5) this.state.carts[index]['num'] = 5
    this.setState({})
  }
  totalPrice = () => {
    return this.state.carts.reduce((p, c) => {
      p += c.num * c.price
      return p
    }, 0)
  }

  render() {
    return (
      <div>
        <table width='600' border='1'>
          <thead>
            <tr>
              <th>ID</th>
              <th>名称</th>
              <th>价格</th>
              <th>数量</th>
            </tr>
          </thead>
          <tbody>
            {
              this.state.carts.map((item, index) => (
                <tr key={item.id}>
                  <td>{item.id}</td>
                  <td>{item.name}</td>
                  <td>{item.price}</td>
                  <td>
                    <button onClick={() => this.setNum(-1, index)}>---</button>
                    <span>{item.num}</span>
                    <button onClick={() => this.setNum(1, index)}>+++</button>
                  </td>
                </tr>
              ))
            }
          </tbody>
        </table>
        <hr />
        <h3>{this.totalPrice()}</h3>
      </div>
    );
  }
}
export default App;
```

#### props参数：

**props.children属性**

> children属性，表示组件标签的子节点，当组件标签有子节点时，props就会有该属性，与普通的props一样，其值可以为任意类型。
>
> 在自定义组件标签中间写的内容，它都会通过 this.props.children属性获取，如果自定义组件标签中间只有一个子元素，则此属性返回一个对象，如果是多个子元素则返回一个数组，使用它就可以实现类似于vue中的插槽功能

```jsx
import React, { Component } from 'react';

class Child extends Component {
  render() {
    return (
      <div>
        <h3>我是child组件</h3>
        {
          this.props.children
            ?
            this.props.children
            :
            <div>我是默认值</div>
        }
      </div>
    );
  }
}

class App extends Component {
  render() {
    return (
      <div>
        <Child>
          <div>我是child组件元素中间包裹的内容1</div>
          <div>我是child组件元素中间包裹的内容2</div>
          <div>我是child组件元素中间包裹的内容3</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

如果想要改变父元素传过来children元素，比如给它添加样式，或者绑定事件，直接改变props.children.props是不行的，因为它是不可以直接进行修改的，因此React提供了一个cloneElement方法，用于克隆一个React对象，这样就可以在克隆的时候修React对象的props属性了：

```js
import React, { Component } from 'react';

class Child extends Component {
  state = {
    id: 1000
  }
  render() {
    let cloneElement = React.cloneElement(this.props.children, {
      style: { color: 'red' },
      uid: this.state.id,
      onClick: () => console.log('我是事件', this.state.id)
    })
    return (
      <div>
        <h3>我是child组件</h3>
        <hr />
        {cloneElement}
      </div>
    );
  }
}

class App extends Component {
  state = {
    title: '我是child组件元素中间包裹的内容'
  }
  render() {
    return (
      <div>
        <Child>
          <div>{this.state.title}</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

模拟具名插槽：

```jsx
import React, { Component } from "react";

class Child extends Component {
  state = {
    id: 1000,
  };
  render() {
    let cloneElement = React.Children.map(this.props.children, (child, index) => {
      return React.cloneElement(child, {
        style: { color: "red" },
        uid: this.state.id,
        onClick: () => console.log("我是事件", this.state.id),
      });
    });
    return (
      <div>
        {this.props.header}
        <hr />
        {cloneElement}
        <hr />
        {this.props.footer}
      </div>
    );
  }
}

class App extends Component {
  render() {
    return (
      <div>
        <Child header={<div>我是头部</div>} footer={<div>底部</div>}>
          <div>我是child组件元素中间包裹的内容1</div>
          <div>我是child组件元素中间包裹的内容2</div>
          <div>我是child组件元素中间包裹的内容3</div>
        </Child>
      </div>
    );
  }
}

export default App;
```

**props类型限制**

> 对于组件来说，props是外部传入的，无法保证组件使用者传入什么格式的数据，简单来说就是组件调用者可能不知道组件封装着需要什么样的数据，如果传入的数据不对，可能会导致程序异常，所以必须要对于props传入的数据类型进行校验。

数据类型校验包在15.5之后就移出了核心库存，因此需要单独安装：

```js
npm i -S prop-types
```

在组件中导入：

```js
import PropTypes from 'prop-types'
```

使用方法：

```js
# 函数组件
function App(){}
App.propTypes = {
    prop-name:PropTypes.string
}

# 类组件
class App extends Component{
}
App.propTypes = {
    prop-name:PropTypes.string
}
```

类型：

- 类型： array、bool、func、number、object、string

- React元素类型：element

- 必填项：types.number.isRequired
- 指定元素类型的数组类型：types.arrayOf(types.number)
- 多个类型：types.oneOfType([types.number, types.string])

- 特定结构的对象： shape({})

  ```js
  types.shape({
        id: types.oneOfType([types.number, types.string]),
        name: types.string,
        fn: types.func
   })
  ```

- 自定义规则：

  ```js
  phone: (props, key) => {
      let value = props[key]
      if (!/^1[3-9]\d{9}$/.test(value)) {
          // 如果不正确，一定要返回一个Error对象，这样就可以在控制台中看到信息,不要throw
          return new Error('有误')
       }
  }
  ```

**props默认值：**

```jsx
import React, { Component } from 'react';
import types from 'prop-types'

class Child extends Component {
  static propTypes = { // 设置类型验证
    age: types.number,
  }
  static defaultProps = { // 设置默认值
    age: 2000
  }

  render() {
    return (
      <div>
        <h3>我是child组件</h3>
      </div>
    );
  }
}
```

#### 表单处理：

**受控组件**

> 将state与表单项中的value值绑定在一起，有state的值来控制表单元素的值，称为受控组件。
>
> 绑定步骤：
>
> - 在state中添加一个状态，作为表单元素的value值
>
> - 给表单元素绑定change事件，将表单元素的值设置为state的值

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    username: '',
    password: ''
  }
  setValue = name => evt => {
    this.setState({
      [name]: evt.target.value.trim()
    })
  }
  render() {
    let { username, password } = this.state
    return (
      <div>
        <div>
          <input type="text" value={username} onChange={this.setValue('username')} />
        </div>
        <div>
          <input type="text" value={password} onChange={this.setValue('password')} />
        </div>
      </div>
    );
  }
}

export default App;
```

> 另一种方法(简写)

```jsx
import React, { Component } from "react";

class App extends Component {
  state = {
    username: {
      value: "admin",
      onChange: (e) => this.setState((state) => ({ username: { ...state.username, value: e.target.value } })),
    },
    password: {
      value: "admin888",
      onChange: (e) => this.setState((state) => ({ password: { ...state.password, value: e.target.value } })),
    },
    num: 100,
  };
  render() {
    let { username, password } = this.state;
    return (
      <div>
        <div>
          <input type="text" {...username} />
        </div>
        <div>
          <input type="text" {...password} />
        </div>
      </div>
    );
  }
}

export default App;
```

> 单选框、多选框

```jsx
import React, { Component } from 'react';

class App extends Component {
  state = {
    sex: '女',
    lessons: ['html', 'js']
  }
  setLessonsFn = e => {
    if (e.target.checked) {
      this.setState(state => ({
        lessons: [...state.lessons, e.target.value]
      }))
    } else {
      this.setState(state => ({
        lessons: state.lessons.filter(item => item != e.target.value)
      }))
    }
  }
  render() {
    let { sex,lessons } = this.state
    return (
      <div>
        <div>
          {/* 单选 */}
          <input type="radio" name='sex' checked={sex === '男'} value='男' onChange={e => this.setState({ sex: e.target.value })} />男
          <input type="radio" name='sex' checked={sex === '女'} value='女' onChange={e => this.setState({ sex: e.target.value })} />女
        </div>
        <div>
          {/* 复选框 */}
          <div>
            <input type="checkbox" value='html' checked={lessons.includes('html')} onChange={this.setLessonsFn} />
            <input type="checkbox" value='css' checked={lessons.includes('css')} onChange={this.setLessonsFn} />
            <input type="checkbox" value='js' checked={lessons.includes('js')} onChange={this.setLessonsFn} />
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```

**非受控组件**(ref)

> 非受控组件：表单项中的value或checked值，它不受state中的属性控制，可以通过dom对象来获取当前表单项中的value或checked值。
>
> 没有和state数据源进行关联的表单项，而是借助ref，使用元素DOM方式获取表单元素值的步骤：
>
> - 调用 React.createRef() 方法创建ref对象
>
> - 将创建好的 ref 对象添加到文本框中
>
> - 通过ref对象获取到文本框的值
>
> ref 如果绑定在普通的html元素中则返回dom对象
> ref 如果绑定在"类组件"中，则返回当前自定义类组件实例对象，可以用它来完成组件通信

```jsx
import React, { Component, createRef } from 'react'

class App extends Component {
  usernameRef = createRef()
  getUsernameInput = () => {
    console.log(this.usernameRef.current.value)
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.usernameRef} />
        <button onClick={this.getUsernameInput}>获取表单项中的数据</button>
      </div>
    )
  }
}

export default App
```

> 绑定多选框

```jsx
import React, { Component, createRef } from 'react'

class App extends Component {
  lessonsRefs = Array(3)
    .fill('')
    .map(_ => createRef())

  getUsernameInput = evt => {
    console.log(
      this.lessonsRefs
        .map(item => {
          if (item.current.checked) {
            return item.current.value
          } else {
            return null
          }
        })
        .filter(item => item != null)
    )
  }

  render() {
    return (
      <div>
        <input type="checkbox" value="html" ref={this.lessonsRefs[0]} />
        <input type="checkbox" value="css" ref={this.lessonsRefs[1]} />
        <input type="checkbox" value="js" ref={this.lessonsRefs[2]} />
        <button onClick={this.getUsernameInput}>获取表单项中的数据</button>
      </div>
    )
  }
}

export default App
```

