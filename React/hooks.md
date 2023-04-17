> 在 React 的世界中，有容器组件和 UI 组件之分，在 React Hooks 出现之前，UI 组件我们可以使用函数组件，无状态组件来展示 UI，而对于容器组件，函数组件就显得无能为力，我们依赖于类组件来获取数据，处理数据，并向下传递参数给 UI 组件进行渲染。React在**v16.8** 的版本中推出了 React Hooks 新特性，Hook是**一套工具函数的集合**,它增强了函数组件的功能，**hook不等于函数组件，所有的hook函数都是以use开头。**
>
> 使用 React Hooks 相比于从前的类组件有以下几点好处：
>
> - 代码可读性更强，原本同一块功能的代码逻辑被拆分在了不同的生命周期函数中，通过 React Hooks 可以将功能代码聚合，方便维护
>
> - 组件树层级变浅，在原本的代码中，我们经常使用 HOC/render/Props 等方式来复用组件的状态，增强功能等，无疑增加了组件树层数及渲染，而在 React Hooks 中，这些功能都可以通过强大的自定义的 Hooks 来实现

**使用hook限制：**

- hook只能用在函数组件中，class组件不行

- 普通函数不能使用hook，默认只能是函数组件才能用

  例外：普通函数名称以**use**开头也可以,(自定义的函数以use开头，称为自定义hook)

- 函数组件内部的函数也不能使用hook

- hook函数一定要放在函数组件的第一层，别放在if/for中(块级作用域)

- 要求函数组件名称必须首字母大写

### useState

> 保存组件状态，功能类似于类组件中的this.state状态管理

```jsx
import React, { useState } from 'react';

const App = () => {
  // 相当于在App函数组件是定义一个state数据，变量名为 count,修改此count的值的方法为setCount
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count + 1)
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

注意：在使用useState的set函数时，尽量不要使用++ 运算符，因为它有可能导致一些怪异行为：

> 如果这里使用++，会发现点击再次按钮数值才会增加一次，这里因为++是后++，因此第一次setCount的其实还是100，但是count确实是++变成101了。而由于count值没有改变，因此函数组件也不会重新渲染，count通过闭包的形式保存了下来。
>
> 在下一次点击时，setCount的值就是101了，而count也变成了102，由于count值发生了变化，因此组件重新渲染，count值也变成了101，这样下次再点击的时候就会重新开始上面的流程。

```jsx
import React, { useState } from 'react';

const App = () => {
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count++)
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

**useState并发处理：**

> 如果使用下面这种写法，虽然在addCount方法中进行了三次+1操作，但是实际上count只会被+1，这是因为在setCount后react不会立即更新组件状态，而是会先把它放进一个队列中，而后在合适的时机执行它们，但由于count在这里一直是100（组件状态没有被更新），这就导致其实是重复执行了3次setCount(101)。

```jsx
import React, { useState } from 'react';

const App = () => {
  let [count, setCount] = useState(() => 100)
  const addCount = () => {
    setCount(count + 1)  // 100
    setCount(count + 1)  // 100
    setCount(count + 1)  // 100
  }

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

> 想要解决这个问题，就需要使用回调函数，这样它在每次setCount的时候，就能把当前的count通过回调函数的参数告诉你，从而实现并发操作。

```jsx
const addCount = () => {
    setCount(v => v + 1)
    setCount(v => v + 1)
    setCount(v => v + 1)
}
```

**使用useState完成自定义hook：**

- /src/app.js

```jsx
import React from 'react';
import useInput from './hook/useInput';

const App = () => {
  let usernameInput = useInput('')
  let passwordInput = useInput('')

  return (
    <div>
      <input type="text" {...usernameInput} />
      <input type="text" {...passwordInput} />
    </div>
  );
}

export default App;
```

- /src/hooks/useInput.js

```jsx
import { useState } from 'react';

// 在react中，定义的函数是以use开头，则认为它就是一个自定义hook函数
// 在自定义hook函数中，可以调用内置hook
const useInput = (initialValue = '') => {
  let [value, setValue] = useState(initialValue)
  return { value, onChange: e => setValue(e.target.value.trim()) }
}

export default useInput
```

**使用useState获取dom:**

> 通常情况下，我们获取dom的方式是通过useRef，用它获取dom的优势在于useRef不会每次重新渲染都重新获取实例。但是他也有缺点，那就是useRef的值在更新后不会触发更新，因此想要将useRef获取到的元素实例用于传递给其它hook做为参数就变得不可能了。这里就可以使用useState来获取：

```tsx
import React, { type ReactNode, useState } from 'react'
import { usePopper } from 'react-popper'
import styles from './Popper.module.scss'
import ReactDOM from 'react-dom'
import { BrowserView, MobileView } from 'react-device-detect'
import { Alert } from '@/components/common/Alert'
import { type Placement } from '@popperjs/core/lib/enums'

interface PopperProps {
  children: ReactNode | string
  content: string | ReactNode
  placement?: Placement
}
const Popper = (props: PopperProps) => {
  const [referenceElement, setReferenceElement] = useState<HTMLDivElement | null>(null)
  const [popperElement, setPopperElement] = useState<HTMLDivElement | null>(null)
  const [arrowElement, setArrowElement] = useState<HTMLDivElement | null>(null)
  const { styles: popperStyles, attributes } = usePopper(referenceElement, popperElement, {
    placement: props.placement,
    modifiers: [
      { name: 'arrow', options: { element: arrowElement } },
      {
        name: 'offset',
        options: {
          offset: [0, 10]
        }
      }
    ]
  })

  const [visible, setVisible] = useState(true)

  const handleOpenTip = () => {
    Alert({
      content: props.content,
      hasCancel: false
    })
  }

  return (
    <>
      <BrowserView>
        <div
          ref={setReferenceElement}
          onMouseLeave={() => {
            setVisible(true)
          }}
          onMouseEnter={() => {
            setVisible(false)
          }}
          className={styles.popperButton}
        >
          {props.children}
        </div>
        {ReactDOM.createPortal(
          <div
            ref={setPopperElement}
            style={popperStyles.popper}
            {...attributes.popper}
            className={styles.popperContent}
            data-popper-reference-hidden={visible}
          >
            {props.content}
            <div ref={setArrowElement} style={popperStyles.arrow} className={styles.popperArrow} />
          </div>,
          // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
          document.querySelector('#root')!
        )}
      </BrowserView>
      <MobileView>
        <div className={styles.popperButton} onClick={handleOpenTip}>
          {props.children}
        </div>
      </MobileView>
    </>
  )
}

export default Popper
```

### useEffect：

> useEffect是副作用钩子：用于处理组件中的副作用，用来取代生命周期函数

```js
useEffect(()=>{//副作用函数
    return ()=>{ // 返回函数

    }
},[依赖参数])
```

**useEffect的作用：**

- 挂载阶段：
   从上向下执行函数，如果碰到 useEffect 就执行并将 useEffect 传入的副作用函数推入一个队列(链表)，在组件挂载完成之后，将队列（链表）中的副作用函数执行，并将副作用函数的返还函数，推入一个新的队列（链表）

- 更新阶段不同于其他阶段，这个阶段中对应的函数是否要执行，取决于依赖参数：
   从上向下执行函数，如果碰到 useEffect 就执行并将 useEffect 传入的副作用函数推入一个队列(链表)，在组件更新完成之后，找出之前的返回函数队列，依次准备执行，在执行前会判断该 useEffect 的依赖参数，如果依赖参数改变就执行，否则跳过当前项去看下一项，然后再执行副作用队列，执行时同样判断依赖是否变化，来决定其是否执行，如果执行，就重新获取其对应的返回函数

- 卸载阶段：
  组件即将卸载时，将副作用函数返还函数队列中的函数依次执行

**useEffect的常用情况：**

- 没有依赖参数的情况：

> 这个情况下副作用函数会在组件挂载和组件更新时执行（componentDidMount componentDidUpdate）

```js
useEffect(() => {
  console.log('App -- useEffect');
})
```

- 依赖项为空数据：

> 这个情况下他只会执行一次（componentDidMount）

```js
useEffect(() => {
  console.log('App -- useEffect');
}, [])
```

- 传入依赖项：

> 如果对依赖项进行传值，只要依赖项中的值发生改变（componentDidMount componentDidUpdate），就会执行相应的副作用函数（可以传入多个依赖项）

```js
let [count, setCount] = useState({ num: 100 })
const addCount = () => setCount(v => ({ num: v.num + 1 }))

useEffect(() => {
  console.log('App -- useEffect');
}, [count])
```

- 返回一个函数：

> 如果副作用函数中返回一个函数，那个这个函数会在组件卸载时被调用（componentWillUnmout）。
>
> 下面这种写法中，由于它的依赖项是一个空值，所以他只会在挂载时被调用，但是他的返回函数会被收集，在组件卸载时被调用。

```js
useEffect(() => {
  return () => {
    console.log('componentWillUnmout');
  }
}, [])
```

**使用useEffect进行网络请求：**

> 在之前的react版本中，useEffect的副作用回调函数只能返回一个函数，用于模拟生命周期componentWillUnmout
> 因此useEffect的参数不能使用异步函数。因为async函数都会默认返回一个隐式的promise。但是，useEffect不应该返回任何内容。这就是为什么会在控制台日志中看到以下警告：
> Warning: useEffect function must return a cleanup function or nothing. Promises and useEffect(async () => …) are not supported, but you can call an async function inside an effect

- 如果useEffect顶层不支持 async/await可以使用如下的解决方案：

```js
useEffect(() => {
   ;(async function() {
      let ret = await get('/api/swiper')
      setFilms(ret.data)
   })()
}, [])
```

- 发送网络请求：

```jsx
import React, { useState, useEffect } from 'react';
import { get } from '@/utils/http'

const App = () => {
  let [films, setFilms] = useState([])
  useEffect(async () => {
    let ret = await get('/api/swiper')
    setFilms(ret.data)
  }, [])

  return (
     <ul>
        {
          films.map(item => (
            <li key={item.id} > {item.title}</li>
          ))
        }
     </ul>
  );
}

export default App;
```

**useEffect封装自定义hook：**

- /src/hooks/useGoodFood.js

```js
import { useEffect, useState } from 'react';
import { get } from '@/utils/http'

const useGoodFood = (initialValue = []) => {
  let [data, setData] = useState(initialValue)
  let [page, setPage] = useState(1)
  // 暴露请求数据的接口，实现翻页，或数据懒加载
  const loadData = async () => {
    let ret = await get('/api/goodfood?page=' + page)
    if (ret.data.length > 0) {
      // setData(v => [...v, ...ret.data]) // 懒加载
      setData(ret.data) // 翻页
      setPage(v => v + 1)
    }
  }
  // 组件挂载时请求数据，并把请求到的数据(data)暴露出去
  useEffect(() => {
    loadData()
  }, [])
  return [data, loadData]
}

export default useGoodFood
```

- /src/app.js

```jsx
import useGoodFood from "./hooks/useGoodFood";

const App = () => {
  let [data, loadData] = useGoodFood()
  return (
    <div>
      <ul>
        {
          data.map(item => (
            <li key={item.id} > {item.name}</li>
          ))
        }
      </ul>
      <hr />
      <button onClick={() => {
        loadData()
      }}>下一页</button>
    </div >
  );
}

export default App;
```

### useReducer

> useReducer 它就是一个小型的redux，它几乎和redux是一样的，也可以管理数据，唯一缺点的就是无法使用 redux 提供的中间件。
>
> 使用hook函数后，一般情况下面，一个组件组中的数据我们可以用useReducer来管理，而不是用redux来完成，redux一般存储公用数据，不需要把所有的数据都存储在里面，redux它是一个单一数据源。
>
> useReducer返回一个数组，这个数组有两项，分别是：
>
> - state：它就是初始化后的状态对象
> - dispatch：方法，用来发送指令让reducer来修改state中的数据
>
> useReducer有三个参数：
>
> - reducer，它必须就是一个纯函数，用来修改initState初始化后数据状态函数
>
> - initState，表示state状态的初始化数据
>
> - 第3参数是一个回调函数，要求返回一个对象数据，或直接直接返回一个值，也是表示初始化state的初始化数据
>
>   如果useReducer它有第3个参数，则第2个参数就没有意义，会以第3个参数优先，虽然他们都是初始化数据，但是第3个参数是惰性初始化，可以提升性能（一般用不到）
>   

```jsx
import React, { useReducer } from 'react'

const initState = {
  count: 100
}

const reducer = (state, { type, payload }) => {
  if (type === 'add') {
    return { ...state, count: state.count + payload }
  }
  return state
}

const App = () => {
  const [state, dispatch] = useReducer(reducer, null, () => ({ count: 2000 }))

  const addCount = () => {
    dispatch({ type: 'add', payload: 2 })
  }

  return (
    <div>
      <h3>App组件 -- {state.count}</h3>
      <button onClick={addCount}>++++</button>
    </div>
  );
}

export default App;
```

**案例：todoList**

- /reducer.js

```js
export const initState = {
  todos: []
}

export const reducer = (state, { type, payload }) => {
  if ('add' === type) return { ...state, todos: [...state.todos, payload] }
  if ('del' === type) return { ...state, todos: state.todos.filter(item => item.id != payload) }
  return state
}
```

- /index.js

```jsx
import React, { useReducer } from 'react'
import Form from './components/Form'
import List from './components/List'
import { initState, reducer } from './reducer'

const Todo = () => {
  const [{ todos }, dispatch] = useReducer(reducer, initState)
  return (
    <div>
      <Form dispatch={dispatch} />
      <List dispatch={dispatch} todos={todos} />
    </div>
  )
}

export default Todo
```

- /components/List.js

```js
import React from 'react'

const List = ({ todos, dispatch }) => {
  const del = id => {
    dispatch({ type: 'del', payload: id })
  }

  return (
    <div>
      <h3>任务项</h3>
      <hr />
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            <span>{item.title}</span>
            <span onClick={() => del(item.id)}>删除</span>
          </li>
        ))}
      </ul>
    </div>
  )
}

export default List
```

- /components/Form.js

```js
import React from 'react'

const Form = ({ dispatch }) => {
  // 回车事件
  const onEnter = evt => {
    if (evt.keyCode === 13) {
      const title = evt.target.value.trim()
      // todo每项中的数据
      const todo = {
        id: Date.now(),
        title,
        done: false
      }
      dispatch({ type: 'add', payload: todo })
      evt.target.value = ''
    }
  }

  return (
    <div>
      <input onKeyUp={onEnter} />
    </div>
  )
}

export default Form
```

### useContext

> 在函数组件中，可以使用useContext来获取context中的数据。但是使用它不是必须的，它只是可以方便调用。

- /context/appContext.js

```js
import {createContext} from 'react'
const context = createContext()

export const {Provider,Consumer} = context

export default context
```

- /app.jsx

```jsx
import React, { useState, useContext } from "react";
import ctx, { Provider, Consumer } from "./context/appContext";

const Child1 = () => {
  return (
    <div>
      <h3>Child1</h3>
      {/* 不一定非要使用useContext，也可以直接使用组件来调用context中的数据 */}
      <Consumer>{(contextState) => <div>{contextState.name}</div>}</Consumer>
      <hr />
    </div>
  );
};

const Child2 = () => {
  const contextState = useContext(ctx);
  return (
    <div>
      <h3>Child2 --- {contextState.name}</h3>
      <button
        onClick={() => {
          contextState.changeName(Date.now() + "");
        }}
      >
        修改姓名
      </button>
    </div>
  );
};

const App = () => {
  const [name, setName] = useState("张三");
  const changeName = (name) => setName(name);
  return (
    <div>
      <Provider value={{ name, changeName }}>
        <Child1 />
        <Child2 />
      </Provider>
    </div>
  );
};

export default App;
```

### useMemo

> 记忆组件，可以理解为计算属性(它是类组件中的memoizeOne组件是一样的，只不过，useMemo是内置功能，无需安装) 
>
> useMemo它可以对于【数值】进行缓存，如果你的依赖项没有发生改变，则下一次调用时，读取上一次的缓存，从而提升性能。
>
> 如果useMemo参数2为一个空数据，则它只会执行一次，后续它所计算的变量的值发生的改变，它也不会重新计算，因为它没有依赖。如果参数2中的数组为多个元素，则其中任意一个元素发生了改变，则它都会重新计算一次。

```jsx
import React, { useMemo, useState } from 'react'

const App = () => {
  const [num1, setNum1] = useState(1)
  const [num2, setNum2] = useState(2)

  const sum = useMemo(() => {
    return num1 + num2
  }, [num1, num2])

  return (
    <div>
      <div>{sum}</div>
      <button onClick={() => setNum1(v => v + 1)}>num1+1</button>
      <button onClick={() => setNum2(v => v + 1)}>num2+1</button>
    </div>
  )
}

export default App
```

### useCallback

> 这个hook用来缓存函数。它的第一个参数是一个函数，表示要被缓存的函数。它的第二个参数表示依赖项，如果依赖项是一个空数组，则表示没有依赖项，则可以完全的缓存一个函数，如果有依赖项，则依赖项中的数值发生改变，则它会重新赋值一次函数。

```jsx
import React, { useCallback, useState, useEffect } from 'react'

const Child = ({ addCount }) => {
  // 查看addCount是否被更新了（使用了useCallback后只会更新一次）
  useEffect(() => {
    console.log('更新了 addCount')
  }, [addCount])

  return (
    <div>
      <h3>Child组件</h3>
      <button
        onClick={() => {
          addCount(1)
        }}
      >
        修改count值 -- Child组件
      </button>
    </div>
  )
}

const App = () => {
  const [count, setCount] = useState(100)

  // 在没有缓存此函数时，子组件中的按钮被点击，count增加，会导致组件重新渲染
  // 这样App组件重新执行后，它会再次赋值一次
  const addCount = n => {
    console.log('addcount')
    setCount(v => v + n)
  }

  // 但是使用了useCallback后，该函数会被缓存，只要依赖项没有发生变化
  // 那么useCallback不会重新赋值该函数，而是会把上次的函数返回
  const addCount = useCallback(n => {
    setCount(v => v + n)
  }, [])

  return (
    <div>
      <h3>App -- {count}</h3>
      <Child addCount={addCount} />
    </div>
  )
}

export default App
```

### React.memo

> 此方法是一个React顶层Api方法，给函数组件来减少重复渲染，类似于PureComponent父和shouldComponentUpdate方法的集合体。
>
> shouldComponentUpdate必须要有一个返回值，true则表示继续渲染，false停止渲染，而React.memo与它相同，参数2返回值如果为true则表示停止渲染，false继续渲染。
>
> 并且参数是一个回调函数，一般情况下都在props传过来的数据为引用类型才需要手动来判断，如果是基本类型则不需要写参数2来手动判断，memo会帮助我们做这个工作，这就类似于PureComponent。

```jsx
import React, { useState, memo } from 'react'
import _ from 'lodash'

const Child = memo(
  ({ count }) => {
    console.log('child')
    return (
      <div>
        <h3>child组件 -- {count.n}</h3>
      </div>
    )
  },
  (prevProps, nextProps) => _.isEqual(prevProps, nextProps)
)

const App = () => {
  let [count, setCount] = useState({ n: 100 })
  let [name, setName] = useState('张三')
  return (
    <div>
      <h3>App -- {count.n}</h3>
      <input type="text" value={name} onChange={e => setName(e.target.value)} />
      <button
        onClick={() => {
          setCount({ n: Date.now() })
        }}
      >
        ++++
      </button>
      <Child count={count} />
    </div>
  )
}

export default App
```

### useRef与createRef

> useRef返回一个可变的 ref 对象，该对象只有个 current 属性，初始值为传入的参数( initialValue，如果没有传入初始值，则该ref对象的current属性为undefined )
>
> useRef返回的 ref 对象在组件的整个生命周期内保持不变，当更新 current 值时并不会 reRender ，这就是它与 useState 不同的地方
>
> 更新 useRef 是 side effect (副作用)，所以一般写在 useEffect 或 event handler 里

**useRef与createRef的区别：**

1. createRef 可以用在类组件和函数组件中，声明时不能赋初始值

2. useRef 只能使用在函数组件中，可以在声明时给初始值

3. createRef每次重新渲染时都会创建一个ref对象，但是在类组件它可以定义在生命周期函数中，所以它一般不会被重新创建

4. useRef第1次渲染时创建之后，并重新渲染时，如果发现这个对象已经存在则不会再创建，它的性能更好一些，在函数组件中推荐使用useRef

**验证useRef不会重复创建：**

> 当button被点击触发点击事件时，count会被改变，从而触发reRender，多次点击时，就会发现，"createRef"会被多次打印，而"useRef"只会被打印一次。

```jsx
import React, { createRef, useRef, useEffect, useState } from "react";

const App = () => {
  const usernameCreateRef = createRef();
  const usernameUseRef = useRef();
  const [count, setCount] = useState(100);

  if (!usernameCreateRef.current) {
    console.log("createRef");
    usernameCreateRef.current = count;
  }

  // 如果你第一次创建成功后，第二次使用之前的对象，条件就不成立
  // 以此就可以验证useRef是不是只会创建一次
  if (!usernameUseRef.current) {
    console.log("useRef");
    usernameUseRef.current = count;
  }

  return (
    <div>
      <h3>useState中的count值：{count}</h3>
      <h3>createRef：{usernameCreateRef.current}</h3>
      <h3>useRef：{usernameUseRef.current}</h3>
      <button onClick={() => setCount(v => v + 1)}>+++++++</button>
    </div>
  );
};

export default App;
```

**使用useRef获取dom：**

```jsx
import React, { createRef, useRef, useEffect, useState } from "react";

const App = () => {
  const usernameRef = createRef();
  return (
    <div>
      <input type="text" ref={usernameRef} />
      <button
        onClick={() => {
          console.log(usernameRef.current.value);
        }}
      >
        获取表单项中的数据
      </button>
    </div>
  );
};

export default App;
```

### forwardRef

> 如果ref对象绑定在自定义类组件中，则得到当前自定义类组件的实例，从而可以实现组件通信。但是如果ref直接绑定在自定义函数组件中，则不行，因为函数组件没有实例，所以会报错，因此函数组件如果需要绑定ref，则需要通过react提供的一个顶层api方法来增强函数组件从而实现函数组件也可以绑定ref对象。
>
> Rect.forwardRef(render)的返回值是react组件，接收的参数是一个render函数，函数签名为render(props, ref)，第二个参数将其接收的ref属性转发到render返回的组件中。

```jsx
import React, { useRef, forwardRef } from 'react'

// 使用forwardRef方法完成对于函数组件中绑定ref对象
const Child = forwardRef((props, _ref) => {
  return (
    <h3 ref={_ref}>child组件</h3>
  )
})

const App = () => {
  let childRef = useRef()
  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => console.log(childRef.current)}>
        ++++
      </button>
    </div>
  )
}

export default App
```

- **ts类型**

```tsx
import React, { forwardRef, useImperativeHandle, type ForwardedRef } from 'react'
import { Input, Form } from 'antd'

interface IProps {
  range: [number, number]
  decimal: number
}

// 导出这个类型，可以在引入组件中当做useRef的泛型参数useRef<TRef>(null)
export interface TRef {
  rules: string[]
}

const HyInput = (props: IProps, _ref: ForwardedRef<TRef>) => {
  const { range, decimal } = props
  const messageApi = useContext(context)
  const rules = []

  useImperativeHandle(_ref, () => {
    return { rules }WalletEditListRef
  })

  return <></>
}

export default forwardRef<TRef, IProps>(HyInput)
```

- 将ref绑定在指定元素上

```tsx
import React, { forwardRef } from 'react'

const CurrentItem = (
  props: IProps,
  ref: React.LegacyRef<HTMLDivElement> | undefined
) => {
  return (
    <div {...props} ref={ref}>
      ... ...
    </div>
  )
}

const CurrentItemRef = forwardRef(CurrentItem)
```

### useImperativeHandle

> 使用它可以透传 Ref，因为函数组件没有实例，所以在默认自定义函数组件中不能使用ref属性，使用为了解决此问题，react提供了一个hook和一个高阶组件来帮助函数组件能够使用ref属性。
>
> useImperativeHandle有两个参数：
>
> - 参数1：父组件传过来的ref对象
> - 参数2：回调函数，返回一个对象，穿透给父组件的数据

```js
import React, { useRef, forwardRef, useState, useImperativeHandle } from 'react'

const Child = forwardRef((props, _ref) => {
  let [name, setName] = useState('张三')
  const setNameFn = name => setName(name)
  const userRef = useRef()

  useImperativeHandle(_ref, () => {
    return { name, setNameFn, userRef }
  })

  return (
    <div>
      <h3>child组件 -- {name}</h3>
      <input type="text" ref={userRef} />
    </div>
  )
})

const App = () => {
  let childRef = useRef()
  return (
    <div>
      <Child ref={childRef} />
      <button
        onClick={() => {
          console.log(childRef.current.name)
          childRef.current.setNameFn(Date.now() + '')
          childRef.current.userRef.current.value = 'abc'
        }}
      >
        ++++
      </button>
    </div>
  )
}

export default App
```

### useLayoutEffect

> useLayoutEffect与useEffect非常类似，他们都接受一个函数和一个数组，只有在数组里面的值改变时才会再次执行 effect。
>
> useLayoutEffect与useEffect的差异：
>
> - `useEffect` 是异步执行的，而`useLayoutEffect`是同步执行的。
> - `useEffect` 的执行时机是浏览器完成渲染之后，而 `useLayoutEffect` 的执行时机是浏览器把内容真正渲染到界面之前，和 `componentDidMount` 等价。

**例子：**

> 下面两个例子中，上面使用useEffect来处理副作用，而下面的例子使用useLayoutEffect来处理副作用，从运行的结果中可以直观的看出，使用useEffect的例子中，dom元素有明显的闪烁，而使用的useLayoutEffect的例子中则没有。
>
> 这里因为 useEffect 是渲染完之后异步执行的，所以会导致 hello world 先被渲染到了屏幕上，再变成 world hello，就会出现闪烁现象。而 useLayoutEffect 是渲染之前同步执行的，所以会等它执行完再渲染上去，就避免了闪烁现象。也就是说我们最好把操作 dom 的相关操作放到 useLayouteEffect 中去，避免导致闪烁。

```jsx
import React, { useEffect, useLayoutEffect, useState } from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  const [state, setState] = useState("hello world")
  useEffect(() => {
    let i = 0;
    while(i <= 100000000) {
      i++;
    };
    setState("world hello");
  }, []);

  return (
    <>
      <div>{state}</div>
    </>
  );
}

export default App;
```

```jsx
import React, { useEffect, useLayoutEffect, useState } from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  const [state, setState] = useState("hello world")
  useLayoutEffect(() => {
    let i = 0;
    while (i <= 100000000) {
      i++;
    }
    setState("world hello");
  }, []);

  return (
    <>
      <div>{state}</div>
    </>
  );
}

export default App;
```

### react-router-dom-hook

> react-router-dom@5也提供了hook的解决方案，它包括了三个hook，分别是：
>
> - useHistory
> - useLocation
> - useParams

在函数组件中也可以直接使用withRouter来包裹组件，来实现传递路由对象的目的：

```jsx
import React, { useState } from 'react';
import { withRouter } from 'react-router-dom'

const App = ({ history: { push } }) => {
  return <h1>app</h1>
};

export default withRouter(App)
```

#### useHistory

> 可以使用useHistory 在函数组件中直接得到react-router-dom路由组件中的history对象

```jsx
import React, { useState } from 'react';
import { useHistory } from 'react-router-dom'

const App = () => {
  const history = useHistory()
  return <h1>app</h1>
};

export default App
```

**listen方法监听路由变化**

> 在使用react的函数组件时，有时候我们会想监听路由变化，并在路由变化时进行某些操作，就需要使用useHistory().listen来实现了

```js
const history = useHistory();
useEffect(() => {
    const unlisten = history.listen((historyLocation) => {
        cnosole.log(historyLocation)
    });
    return () => {
        unlisten();
    };
}, [history])
```

#### useLocation

> 用于获取location对象

#### useParams

> 用于获取动态路由参数

### react-redux-hook

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
