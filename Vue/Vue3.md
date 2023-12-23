# Vue3

## Vue3的改进

### 源码升级

- vue3的源码使用typescript来编写,vue3更好的支持typescript

- 使用Proxy代替Object.defineProperty 实现双向数据绑定
- 重写虚拟DOM的实现和Tree-Shaking(按需打包)[**静态标记**]

```js
// new Vue({}) ==> 类，里面有很多的方法或模块，但是有时候工作中用不到，但它也打包进来了
// vue3创建vue实例的方式变化了，不是直接导入vue，而是使用一个createApp方法来创建实例，这样就可以在打包的时候进行tree-shaking
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

### 新的特性

**composition API（组合式api）[很像react中的hooks]**

- setup配置项，它是组合式api的启点

- ref和reactive

- watch 和watchEffect

- provide和inject

- toRefs

- computed

- 组合api中的生命周期函数 onMounted()

- ... ...

**提供了新的内置组件**

- Fragment vue3中template可以不需要顶层元素包裹

- Teleport 指定组件挂载到页面的节点位置

- Suspense 异步组件：使用<script setup>语法时，支持顶层async，可以用它完成异步组件

## 创建Vue3项目

### 基于webpack的工程创建(vue-cli)

> - 确保你的vue-cli脚手架版本在4.5.0以上，这样才能创建
> - 检查脚手架版本：`vue --version`
> - 如果版本低话，使用`npm update -g @vue/cli`来更新vue-cli  

```bash
vue create myapp
cd myapp
npm run serve
```

**vue-cli配置：(.vuerc文件，保存在用户的home目录下)**

```json
 {
  "useTaobaoRegistry": true, // 是否使用淘宝镜像安装依赖
  "packageManager": "npm", // 指定使用什么包管理器下载依赖
  "presets": { // 用于保存项目预设
    "ilmango": {
      "useConfigFiles": true,
      "plugins": {
        "@vue/cli-plugin-babel": {}
      },
      "vueVersion": "3"
    }
  }
}
```

### 基于vite的工程创建(create-vite)

```bash
yarn create vite myapp
npm init vite@latest myapp
```

### 基于vite的工程创建(create-vue)*

```bash
npm init vue@3
npm init vue@2
```

## Vue3全局方法的改变

| **2.x Global API**         | **3.x Instance API(app)**  |
| -------------------------- | -------------------------- |
| Vue.config                 | app.config                 |
| Vue.config.productionTip   | 已移除                     |
| Vue.config.ignoredElements | app.config.isCustomElement |
| Vue.component              | app.component              |
| Vue.directive              | app.directive              |
| Vue.mixin                  | app.mixin                  |
| Vue.use                    | app.use                    |

### Vue全局混入

```js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mixin({
  data() {
    return {
      title: '我是全局混入'
    }
  },
  mounted() {
    console.log('全局混入 -- mounted');
  }
})
app.mount('#app')
```

### Vue自定义指令

> 在Vue的模板语法中有各种各样的指令：v-show、v-for、v-model等等，除了使用这些指令之外，Vue也允许我们来自定义自己的指令。自定义指令通常用在需要对DOM元素进行底层操作的情况。自定义指令分为两种：
>
> - 自定义局部指令：组件中通过 directives 选项，只能在当前组件中使用
>
>   ```js
>   export default {
>     directives: {
>       focus: {
>         mounted(el, bindings, vnode, preVnode) {
>           console.log("focus mounted");
>           el.focus();
>         },
>       },
>     },
>   };
>   ```
>
> - 自定义全局指令：app的 directive 方法，可以在任意组件中被使用
>
>   ```js
>   import { createApp } from 'vue'
>   const app = createApp(App);
>   
>   app.directive("focus", {
>     mounted(el, bindings, vnode, preVnode) {
>       console.log("focus mounted");
>       el.focus();
>     },
>   });
>   
>   app.mount("#app");
>   ```

#### 指令的生命周期

> - created：在绑定元素的 attribute 或事件监听器被应用之前调用
> - beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用
> - mounted：在绑定元素的父组件被挂载后调用
> - beforeUpdate：在更新包含组件的 VNode 之前调用
> - updated：在包含组件的 VNode 及其子组件的 VNode 更新后调用
> - beforeUnmount：在卸载绑定元素的父组件之前调用
> - unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次

```vue
<template>
  <button 
    v-if="counter < 2" 
    v-why 
    @click="increment"
  >
    {{ counter }}
  </button>
</template>
<script>
import { ref } from "vue";

export default {
  directives: {
    why: {
      created(el, bindings, vnode, preVnode) {
        console.log("why beforeMount");
      },
      beforeMount(el, bindings, vnode, preVnode) {
        console.log("why beforeMount");
      },
      mounted(el, bindings, vnode, preVnode) {
        console.log("why mounted");
      },
      beforeUpdate(el, bindings, vnode, preVnode) {
        console.log("why beforeUpdate");
        console.log(vnode, preVnode);
      },
      updated(el, bindings, vnode, preVnode) {
        console.log("why updated");
        console.log(vnode, preVnode);
      },
      beforeUnmount(el, bindings, vnode, preVnode) {
        console.log("why beforeUnmount");
      },
      unmounted(el, bindings, vnode, preVnode) {
        console.log("why unmounted");
      },
    },
  },
  setup() {
    const counter = ref(0);
    const increment = () => counter.value++;
    return { counter, increment };
  }
};
</script>
```

#### 指令的参数和修饰符

> - el： 表示使用自定义指令的组件或元素
> - bindigs： 自定义指令传回来的参数及修饰符
> - vnode： 表示当前元素的VNode对象
> - preVnode： 表示当前元素更新之前的VNode对象

```vue
<template>
  <button v-why:info.aaaa.bbbb="{ name: 'anlw', age: 18 }"></button>
</template>

<script>
export default {
  directives: {
    why: {
      created(el, bindings, vnode, preVnode) {
        console.log(bindings.arg); // info
        console.log(bindings.value); // {name: 'anlw', age: 18}
        console.log(bindings.modifiers); // {aaaa: true, bbbb: true}
      }
    },
  }
};
</script>
```

#### 自定义指令案例(时间戳)

```js
import { createApp } from 'vue'
import dayjs from "dayjs"; // dayjs是一个专门用于处理时间的包

const app = createApp(App);
app.directive("format-time", {
  created(el, bindings) {
    // created中可以进行初始化操作
    bindings.formatString = "YYYY-MM-DD HH:mm:ss";
    if (bindings.value) {
      bindings.formatString = bindings.value;
    }
  },
  // mounted中才能操作dom，因为这里个dom才真实存在于页面中
  mounted(el, bindings) {
    const textContent = el.textContent;
    let timestamp = parseInt(textContent);
    if (textContent.length === 10) {
      // length为10的时间戳表示它的单位为秒钟
      timestamp = timestamp * 1000;
    }
    el.textContent = dayjs(timestamp).format(bindings.formatString);
  },
});

app.mount("#app");
```

### Vue插件

> 想要向Vue全局添加一些功能时，可以采用插件的模式，它有两种编写方式：
>
> - 一个对象，但是必须包含一个 install 的函数，该函数会在安装插件时执行
> - 一个function，这个函数会在安装插件时自动执行
>
> 插件可以完成的功能没有限制，比如下面的几种都是可以的：
>
> - 添加全局方法或者 property，通过把它们添加到 config.globalProperties 上实现
> - 添加全局资源：指令/过滤器/过渡等
> - 通过全局 mixin 来添加一些组件选项
> - 一个库，提供自己的 API，同时提供上面提到的一个或多个功能

- **/src/main.js**

> 在使用 createApp() 初始化 Vue 应用程序后，可以通过调用 use() 方法将插件添加到你的应用程序中：
>
> - 如果插件是一个**对象**，那么use方法会执行该对象中的install方法，该方法会接收两个参数：由 Vue 的 createApp 生成的 app 对象和用户传入的选项
>
> - 如果插件是一个**函数**，那么use方法会直接执行这个函数，同样也会传入两个参数

```js
import { createApp, h } from 'vue'
import App from './App.vue'
import createGlobalComponent from './components'
import globalProperties from './config/globalProperties'

// 实例化一个Vue顶层组件
// vue3中把之前vue2中的全局方法进行了改变，全都迁移到app实例上
const app = createApp(App)
app.use(createGlobalComponent) // 创建全局组件
app.use(globalProperties, {name: "anlw"}) // 创建全局属性
app.mount('#app')
```

- **/src/config/globalProperties.js**

> 插件的函数写法

```js
export default (app, options) => {
  // 给vue添加全局的成员属性(在组件中可以通过this.$http访问)
  // 为了区别全局方法属性和组件实例中的私有属性，社区中约定全局方法或属性命名是前面要加上$
  app.config.globalProperties.$http = '我是网络请求对象'
  // 打印外部传入的options属性
  console.log(options)
}
```

- **/src/components/index.js**

> 插件的对象写法

```js
import loading from './loading'

const createGlobalComponent = {
  name: "plugin",
  install(app, options) {
    // 注入全局组件
    loading(app)
    // 访问插件对象的属性
    console.log(this.name)
  }
}

export default createGlobalComponent
```

- **/src/components/loading/index.js**

```js
import loading from './loading.vue'

export default app => app.component('loading', loading)
```

- **/src/components/loading/loading.vue**

```vue
<template>
  <h3>加载中</h3>
</template>

<script>
export default {
  setup() {
    return {}
  }
}
</script>

<style lang="scss" scoped></style>
```

### setup中访问Vue全局属性

> 使用插件添加全局方法和属性后，可以在所有子组件实例中通过this获取到这些方法和属性。但是在setup函数中，由于无法获取this，因此不能通过这种方式获取，要想获取需要先通过getCurrentInstance方法获取到当前组件实例，然后通过当前组件的实例的appContext属性获取到app的上下文，然后就可以获取到全局的属性和方法了
>

```js
import { getCurrentInstance } from "vue";

export default {
  setup() {
    const instance = getCurrentInstance();
    console.log("setup：", instance.appContext.config.globalProperties.$http);
  },
  mounted() {
    console.log("mounted：", this.$http);
    this.$hello();
  },
};
```

 ## Vue3中的生命周期

> 因为 setup 是围绕 beforeCreate 和 created 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 setup 函数中编写

![2f1sad32f1sad](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/2f1sad32f1sad.png)

> compositionApi和optionsApi中的生命周期是可以混写的，这些所有生命周期的执行顺序如下：

![lifecycle.16e4c08e](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/lifecycle.16e4c08e.png)

- **/src/app.vue**

```vue
<template></template>

<script>
import { onMounted} from 'vue'
import usePrintHook from './hooks/usePrintHook'
export default {
  // setup它就是组合Api的入口，setup中不能使用this，它里面没有 beforeCreate和created生命周期
  setup() {
    let ret = usePrintHook()
    return {}
  }
}
</script>
```

- **/src/hooks/usePrintHook.js**

```js
import { onMounted } from "vue";

// 和optionsApi中的生命周期不同的是，compositionApi中的同一个生命周期可以定义多个，并且都会执行，但是optionsApi中的同一个生命周期只能定义一个，因为它是对象中的属性，重名会重复
const usePrintHook = () => {
  onMounted(() => {
    console.log("我是一个mounted生命周期");
  });
  onMounted(() => {
    console.log("我是一个mounted生命周期");
  });
  onMounted(() => {
    console.log("我是一个mounted生命周期");
  });
  onMounted(() => {
    console.log("我是一个mounted生命周期");
  });
  return "abc";
};
export default usePrintHook;
```

## data属性的变化

data属性是传入一个函数，并且该函数需要返回一个对象：

- 在Vue2.x的时候，也可以传入一个对象（虽然官方推荐是一个函数）
- 在Vue3.x的时候，必须传入一个函数，否则就会直接在浏览器中报错

## v-for v-if优先级的改动

#### 能否同时使用：

> 当`v-if`与`v-for`一起使用时，`v-for`具有比`v-if`更高的[优先级](https://so.csdn.net/so/search?q=优先级&spm=1001.2101.3001.7020)，这意味着`v-if`将重复运行于每个`v-for`循环出来的元素中。
>
> 想要循环生成一系列组件块，但是不希望生成序号2之后的内容，同时用了`v-if`和`v-for`，那么，还是会根据整个数组生成所有组件块，之后才判断`v-if`让多余的消失，非常耗资源。

```jsx
<div
  v-for="(fileMsg,index) in fileMsgList"
  :key="fileMsg.id"
  v-if="index > 2"
>
  <sys-file-layout :fileMsg="fileMsg"></sys-file-layout>
</div>
```

#### **解决方案：**

**使用computed解决：**

```js
computed: {
  fileMsgListCom() {
    return this.fileMsgList.filter((item, index) => {
      return item > 2;
    });
  }
}
```

```jsx
<div
  class="file_name"                                     
  v-for="(fileMsg,index) in fileMsgListCom"             
  :key="fileMsg.id"                                          
>                                                       
  <sys-file-layout :fileMsg="fileMsg"></sys-file-layout>
</div> 
```

**v-if替换成v-show：** 

> 因为v-show只是使用css操作元素的显示和隐藏，因此它根本不会销毁和重新创建元素，因此可以优化一些性能

```jsx
<div
  class="file_name"
  v-for="(fileMsg,index) in file.documents"
  :key="fileMsg.id"
  v-show="index < 2"
>
  <sys-file-layout :fileMsg="fileMsg"></sys-file-layout>
</div>
```

==注意==：在vue3中，v-if的优先级高于v-for，因此v-if执行时要调用的变量可能还不存在，会导致报错。

## 动态ref属性

> vue3中的ref如果写成动态属性，则可以传一个函数，该函数会接收绑定到的dom元素

```vue
<template>
  <ul>
    <li :ref="addNamesRef" v-for="item of names" :key="item">
      {{ item }}
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      names: ['张三', '李四', '乐乐'],
      namesRefs: []
    }
  },
  mounted() {
    console.log(this.namesRefs) // 挂载完成后查看ref绑定的dom列表
  },
  methods: {
    addNamesRef(el) {
      el && this.namesRefs.push(el)
    }
  }
}
</script>
```

## 多事件处理器

> 事件处理程序中可以有多个方法，这些方法由逗号运算符分隔

```vue
<template>
  <button @click="handleClick1($event, 'click1'), handleClick2()">
    click
  </button>
</template>

<script>
export default {
  methods: {
    handleClick1(evt, msg) {
      console.log(msg)
    },
    handleClick2() {
      console.log('click2')
    }
  }
}
</script>
```

## 按键修饰符

> 从 KeyboardEvent.keyCode 已被废弃开始，Vue 3 继续支持这一点就不再有意义了。因此，现在建议对任何要用作修饰符的键使用 Keyboard-cased (短横线) 名称

```vue
<template>
  <input type="text" @keyup.alt.a="click" />
  <input type="text" @keyup.arrow-up="click" />
</template>

<script>
export default {
  methods: {
    click() {
      console.log(111)
    }
  }
}
</script>
```

## $attrs可授权class和style

> 当传递给一个组件某个属性，但是该属性并没有定义对应的props或者emits时，就称之为非Prop的Attribute，常见的包括class、style、id属性等

- 当子组件有单个根节点时，非Prop的Attribute将自动添加到根节点的Attribute中
- 如果不希望子组件的根元素继承attribute，可以在组件中设置 inheritAttrs: false：
  - 禁用attribute继承的常见情况是需要将attribute应用于根元素之外的其他元素；
  - 可以通过 $attrs来访问所有的非props的attribute；

- 如果子节点有多个根节点，并且这些节点都没有对attribute进行显示的绑定，那么Vue就会在控制台中报警告：必须手动绑定非Props属性

## \$root和\$children

> \$children 实例 property 已从 Vue 3.0 中移除，不再支持。但\$root和$parent属性还是可以使用

- $root永远指向createApp中的app组件
- $parent指向它的父组件

## Composition Api

### setup

> - 它是vue3新增的一个配置项，值为一个函数，所有的组合式API方法都写在在setup函数中
> - 组件中用到的数据，方法，计算属性，事件方法，监听函数，都配置在setup中
> - setup默认情况下不能使用async/await,在读取数据时它只能是json对象，才能读取到里面的属性值，如果你后续使用了异步组件+Suspense组合，可以使用async/await
> - setup中不能使用this，因为它不会找到组件实例。setup的调用发生在data property、computed property或methods被解析之前，所以它们无法在setup中被获取

#### **setup的返回值：**

- setup的返回值为一个对象，对象中的属性和方法均可以在模板中直接使用

- setup的返回值也可以是一个函数，这个函数就相当于render方法

> 这里只能写h函数的语法，不能写jsx，因为@vitejs/plugin-vue-jsx虽然可以用于解析vue文件下面的jsx语法，但是如果让这个插件来解析vue sfc(而不是使用@vitejs/plugin-vue)，会导致一个问题。那就是jsx插件根本不认识vue sfc文件中的语法(不认识template模板和style样式)，因此它就会报错
>
> 所以如果想要使用jsx语法，应该直接使用jsx文件，用jsx语法来写组件。或者可以在script标签中标注 `lang='jsx'`，这样 @vitejs/plugin-vue-jsx 就会帮我们解析这个script标签中的jsx语法

```html
<script>
import { ref, h } from 'vue'
export default {
  setup() {
    return () => {
      return h('div', null, [
        h('h3', null, '我是一个标题'),
        h(
          'button',
          {
            onClick: () => console.log('我是点击事件')
          },
          '按钮'
        )
      ])
    }
  }
</script>
```

```vue
<script lang='jsx'>
import { ref, h } from 'vue'
    
export default {
  setup() {
    return () => (
      <div>
        <h3>我是一个标题 jsx</h3>
        <button>按钮jsx</button>
      </div>
    )
  }
}
</script>
```

#### **setup的参数：**

  - **参数1：**对应就是props属性对象，如要要定义props的类型，还是和之前一样在props选项中定义，并且在template中依然是可以正常去使用props中的属性

    setup 函数中的 props 是响应式的，当传入新的 prop 时它将被更新。但是因为props 是响应式的，你不能使用 ES6 解构，它会消除 prop 的响应性。如果需要解构 prop，可以在 setup 函数中使用 toRefs 函数来完成此操作：

    ```js
    setup(props) {
        const { title } = toRefs(props)
        console.log(title.value)
    }
    ```
    
  - **参数2：**context对象，此对象中有包含attrs, slots, emit，expose
  
    - attrs：所有的非prop的attribute（非响应式对象，等同于 \$attrs）
    - slots：父组件传递过来的插槽（非响应式对象，等同于 $slots）
    - emit：当我们组件内部需要发出事件时会用到emit（方法，等同于 \$emit）
    - expose：暴露公共property（函数）
    

context 是一个普通的 JavaScript 对象，也就是说，它不是响应式的，这意味着你可以安全地对 context 使用 ES6 解构。attrs 和 slots 是有状态的对象，它们总是会随组件本身的更新而更新。这意味着你应该避免对它们进行解构，并始终以 attrs.x 或 slots.x 的方式引用 property

setup函数中可以返回一个渲染函数，该函数可以直接使用在同一作用域中声明的响应式状态，但是返回一个渲染函数将阻止我们返回任何其它的东西。从组件内部来说这没有问题，但当我们想要将这个组件的方法通过模板 ref 暴露给父组件时就不一样了，我们可以通过调用 expose 来解决这个问题，给它传递一个对象，其中定义的 property 将可以被外部组件实例访问：    

```js
import { h, ref } from 'vue'
export default {
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value
    // 暴露变量
    expose({
      increment
    })
    return () => h('div', count.value)
  }
}
```

### ref

> reactive API对传入的类型是有限制的，它要求必须传入的是一个对象或者数组类型，如果传入的是一个基本数据类型（String、Number、Boolean），则Vue会报一个警告
>
> ref会返回一个可变的响应式对象，该对象作为一个响应式的引用维护着它内部的值，这就是ref名称的来源。该对象内部有一个value值，就是可以进行响应式的属性
>
> 如果ref赋值的为基础类型，是它是用Object.defineProperty来进行响应式处理，如果ref赋值的为引用类型，它是用Proxy来进行响应式处理
>
> - 在模板中引入ref的值时，Vue会自动帮助我们进行解包操作，所以我们并不需要在模板中通过 ref.value 的方式来使用
> - 但是在 setup 函数内部，它依然是一个ref引用， 所以对其进行操作时，我们依然需要使用 ref.value的方式

```vue
<template>
  <div>
    <!-- 如果num它是一个ref定义的响应式变量，在模板中调用时无须写 num.value，模板在解析的过程中会给你补上 -->
    <h1 ref="titleRef">我是一个标题 - {{ num.n }}</h1>
    <button @click="addNum">+++</button>
    <input type="number" v-model.number="user.age.n" />
  </div>
</template>

<script>
import { ref, shallowRef } from 'vue'
export default {
  // setup中不能使用this
  // 它必须返回一个json对象，对象中的属性，可以直接在模板中使用
  // 如果它返回一个函数，它会被当作是一个组件结构，那么当前组件不需要 template模板
  // async setup() 当作异步组件 结合 Suspense来使用
  setup() {
    // 绑定到dom元素或自定义组件上
    let titleRef = ref(null)
    // 引用类型
    let user = ref({ id: 1, age: { n: 100 } })
    // 浅层响应，只能是对于value值进行修改时它才响应(地址改变)
    let num = shallowRef({ n: 200 })
    const addNum = () => {
      num.value = { n: num.value.n + 1 }
    }
    // num: num.value会采用值传递，template模板中的num只是一个普通数值，并不是对num.value的引用，这会导致num ref在收集依赖时就不会收集到它，因此在num ref发生变化时页面不会重新渲染
    // return { num: num.value }
    return { num, addNum, titleRef, user }
  }
}
</script>
```

**ref相关API：**

- unref：如果我们想要获取一个ref引用中的value，可以通过unref方法，如果参数是一个 ref，则返回内部值，否则返回参数本身，这个方法其实就是如下语句的语法糖

  `val = isRef(val) ? val.value : val`

- isRef：判断值是否是一个ref对象
- shallowRef：创建一个浅层的ref对象(如果传入一个对象，则只有该对象地址发生改变时，才会触发改变)
- triggerRef：手动触发 shallowRef 相关联的副作用

```vue
<div>
    <h2>{{ info }}</h2>
    <button @click="changeInfo">修改Info</button>
</div>

<script>
import { shallowRef, triggerRef } from 'vue';

export default {
    setup() {
        const info = shallowRef({ name: "why" })
        const changeInfo = () => {
            info.value.name = "james";
            triggerRef(info); // 手动触发副作用
        }
        return {
            info,
            changeInfo
        }
    }
}
</script>
```

### toRefs和toRef

> 如果我们使用ES6的解构语法，对reactive返回的对象进行解构获取值，那么页面将无法进行响应式操作，原因很简单，解构语句 “let { name, age } = info” 使用的是值传递，因此修改name或age都是响应式对象info无关
>
> 因此Vue提供了一个toRefs的函数，可以将reactive返回对象中的属性都转成ref，因此再次进行解构出来的 name 和 age 都将是ref。这种做法相当于已经在state.name和ref.value之间建立了链接，任何一个修改都会引起另外一个变化
> 如果我们只希望转换一个reactive对象中的属性为ref, 那么可以使用toRef的方法

```js
<div>
    <h2>{{ name }}-{{ age }}</h2>
    <button @click="changeAge">修改age</button>
</div>

setup() {
    const info = reactive({ name: "why", age: 18 });
    // 1.toRefs: 将reactive对象中的所有属性都转成ref, 并建立链接
    let { name, age } = toRefs(info);

    // 2.toRef: 对其中一个属性进行转换ref, 并建立链接
    let { name } = info;
    let age = toRef(info, "age");

    const changeAge = () => {
        age.value++;
        /* 效果相同 */
        info.age++;
    }
    return {
        name,
        age,
        changeAge
    }
}
```

### reactive

> 如果想为在setup中定义的数据提供响应式的特性，那么可以使用reactive的函数，为什么这样就可以变成响应式的呢？
>
> - 这是因为使用reactive函数处理数据之后，数据再次被使用时就会进行依赖收集
> - 当数据发生改变时，所有收集到的依赖都是进行对应的响应式操作（比如更新界面）
> - 事实上，options API中的data选项，也是在内部交给了reactive函数将其编程为响应式对象的

```vue
<template>
  <div>
    <h3>姓名：{{ name }} --- {{ age.n }}</h3>
    <button @click="setName">修改姓名</button>
  </div>
</template>

<script>
// ref 把数据拆分开，reactive 把数据整合在一起，它只能传入一个对象或数组，进行深层监听
import { reactive, shallowReactive, toRef, toRefs } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: '张三',
      age: { n: 22 }
    })
    // 浅层监听，只监听对象的第1层
    const state1 = shallowReactive({
      name: '张三',
      age: { n: 22 }
    })
    const setName = () => {
      state.name = Date.now() + ''
    }
    // state是无法进行响应式的。这是由于name: state.name是值传递，因此模板中的name实际只是一个普通数值，并不是对state.name的引用，这就会导致响应式对象state在收集依赖时不会对其进行收集，导致其无法进行响应式操作
    // return { name: state.name }
    // reactive引用对象不能解构，这会破坏它的响应，如果想要拆分，可以使用toRefs
    return { ...toRefs(state), setName }
  }
}
</script>
```

**reactive相关API：**

- isProxy：检查对象是否是由 reactive 或 readonly创建的 proxy

- toRaw：返回 reactive 或 readonly 代理的原始对象（不建议保留对原始对象的持久引用。请谨慎使用）

- isReactive：检查对象是否是由 reactive创建的响应式代理，如果该代理是readonly建的，但包裹了由 reactive 创建的另一个代理，它也会返回 true

  ```js
  const info = reactive({name: "why", age: 18})
  const readonlyInfo = readonly(info)
  isReactive(readonlyInfo)
  ```

- shallowReactive：创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换 (深层还是原生对象)

  ```js
  const info = shallowReactive({name: "why", friends: {name: "kobe"}})
  info.friends.name = "jams"
  ```

### readonly

> 通过reactive或者ref可以获取到一个响应式的对象，但是某些情况下，我们希望把它传入给其他地方（组件）使用，但是不能被其他地方修改
>
> Vue3为我们提供了readonly的方法，该方法会返回原生对象的只读代理（即返回该对象的Proxy，这个proxy的set方法被劫持，保证原对象不能被修改）
>
> readonly可以传入三种类型的参数：
>
> - **类型一：**普通对象；
>
> - **类型二：**reactive返回的对象。没有经过readonly处理过的响应式对象，它的值可以被子组件修改，并影响到父组件。因此，为了避免这样的问题，经过readonly处理过的响应式对象Proxy，他会劫持对象的setter，从而阻止对对象的修改
>
> - **类型三：**ref的对象。由于在模板中引入ref的值时，vue会自动给其解包，这就导致ref值传入到子元素的prop其实是非响应的变量，所以在子元素中修改这个值肯定不会影响到父元素
>   如果把ref对象放进另一个对象中，他在模板中就不会进行解包，因此在子元素中对其引用进行修改时，会影响到父元素，而做过readonly处理后，在子元素中对其修改时就不会影响父元素
>
>   ```js
>   const info = ref("why");
>   const readonlyInfo = readonly(info);
>                 
>   const infoContainer = { info }
>   const readonlyInfoContainer = { readonlyInfo }
>   ```

```vue
<template>
  <div>
    <h3>{{ state.name }}</h3>
    <button @click="setName">++++</button>
  </div>
</template>

<script>
import { readonly,onBeforeMount } from 'vue'
export default {
  setup() {
    let state = readonly({ name: '张三' })
    const setName = () => {
      state.name = Date.now() + ''
    }
    return { state, setName }
  }
}
</script>
```

**readonly相关的API:**

- **isProxy：**检查对象是否是由 reactive 或 readonly创建的 proxy
- **toRaw：**返回 reactive 或 readonly 代理的原始对象（不建议保留对原始对象的持久引用。请谨慎使用）
- **isReadonly：**检查对象是否是由 readonly 创建的只读代理
- **shallowReadonly：**创建一个 proxy，使其自身的 property 为只读，但不执行嵌套对象的深度只读转换（深层还是可读、可写的）

### customRef

> customRef会创建一个自定义的ref，并对其依赖项跟踪和更新触发进行显式控制
> 它的参数是一个工厂函数，该函数接受 track 和 trigger 函数作为参数，并且该函数中应该要返回一个带有 get 和 set 的对象。track函数用于收集依赖，trigger函数用于触发副作用

- **/src/hooks/useDebouncedRef.js**

```js
import { customRef } from 'vue'

const useDebouncedRef = (value, delay = 300) => {
  let timeout
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newValue
          trigger()
        }, delay)
      }
    }
  })
}

export default useDebouncedRef
```

- **/src/app.vue**

```vue
<template>
  <div>
    <h3>{{ num }}</h3>
    <input type="number" v-model="num" />
  </div>
</template>

<script>
import useDebouncedRef from './hooks/useDebouncedRef'

export default {
  setup() {
    let num = useDebouncedRef(100)
    return { num }
  }
}
</script>
```

### computed

> 某些属性要依赖其他属性的状态时，就可以使用计算属性来处理。在Options API中是使用computed选项来完成的，而在Composition API中可以在 setup 函数中使用 computed 方法来编写一个计算属性。computed函数的返回值是一个ref对象，因此当它也可以进行响应式操作
>
> **注意：**在计算属性中不能使用普通变量，而应该使用响应式变量。如果使用普通变量，普通变量并不会进行依赖收集，那么我们在修改这个变量时，computed属性就不会触发重新计算

```vue
<template>
  <div>
    <ul>
      <li v-for="(item, index) of carts" :key="item.id">
        <span>{{ item.name }}</span>
        <span>
          <button @click="setNum(index, 1)">+</button>
          <button @click="totalPrice = { index, n: 1 }">++++</button>
          <span>{{ item.num }}</span>
          <button @click="setNum(index, -1)">-</button>
        </span>
      </li>
    </ul>
    <h3>合计：{{ totalPrice }}</h3>
  </div>
</template>

<script>
import { computed, reactive, toRefs } from 'vue'
export default {
  setup() {
    let state = reactive({
      carts: [
        { id: 1, name: '小米', price: 1, num: 1 },
        { id: 2, name: '大米', price: 1, num: 1 }
      ]
    })
    // 可以传入一个对象，包含getter和setter，也可以直接传入一个getter函数
    let totalPrice = computed({
      get() {
        return state.carts.reduce((p, { price, num }) => {
          p += price * num
          return p
        }, 0)
      },
      set(v) {
        if (v.n) {
          state.carts[v.index].num += v.n
        }
      }
    })
    const setNum = (index, n) => {
      state.carts[index].num += n
      if (state.carts[index].num <= 1) state.carts[index].num = 1
      if (state.carts[index].num >= 10) state.carts[index].num = 10
    }
    // 如果你给计算属性赋值，则一定要给计算属性写set方法
    totalPrice.value = 'abc'
    return { ...toRefs(state), totalPrice, setNum }
  }
}
</script>
```

### watchEffect

> watchEffect传入的函数会被立即执行一次，由于函数中使用了响应式对象属性，因此该函数会被进行依赖收集，之后只要这些响应式对象属性发生变化，watchEffect传入的函数就会被做为依赖再次执行
>
> 给watchEffect传入的函数被回调时可以获取到一个参数：onCleanup。它是一个函数，并且需要给它传入一个函数，传入的函数会在侦听器的副作用函数执行或者侦听器被停止(stop函数被执行)时被执行
>
> 可以在onCleanup函数中执行一些清除工作。比如在开发中我们需要在侦听函数中执行网络请求，但是在网络请求还没有达到的时候，我们停止了侦听器，或者侦听器侦听函数被再次执行了。那么上一次的网络请求应该被取消掉，这个时候我们就可以清除上一次的副作用
>
> watchEffect的返回值是一个函数，该函数可以用于结束监听。watchEffect的第二个参数是一个对象，里面要写一些监听属性：
>
> `flush?: 'pre' | 'post' | 'sync';`（默认为 pres）
>
> - pre 模板渲染之前触发
> - post 模板渲染之后触发
> - sync 和模板渲染同步来触发

```vue
<template>
  <div>
    <h3>{{ num }}</h3>
    <input type="text" v-model="name" />
    <button @click="num++">+++</button>
    <button @click="stop">停止侦听</button>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue'
export default {
  setup() {
    const num = ref(100)
    const name = ref('')
    let timer
    const stopHandle = watchEffect(
      onCleanup => {
        console.log(num.value)
        onCleanup(() => {
          console.log('清理上一次的处理')
          timer && clearTimeout(timer)
        })
        timer = setTimeout(() => {
          console.log('网络请求成功')
        }, 1000)
      },
      {
        flush: 'pre'
      }
    )
    const stop = () => stopHandle()
    return { num, name, stop }
  }
}
</script>
```

### watch

> watch的API完全等同于组件watch选项的Property：
>
> - watch需要侦听特定的数据源，并在回调函数中执行副作用
> - 默认情况下它是惰性的，只有当被侦听的源发生变化时才会执行回调
>
> 与watchEffect的比较，watch允许我们：
>
> - 懒执行副作用（第一次不会直接执行）
> - 更具体的说明当哪些状态发生变化时，触发侦听器的执行
> - 可以访问侦听状态变化前后的值
>
> watch侦听函数的数据源有两种类型：
>
> - 一个getter函数：但是该getter函数必须引用可响应式的对象（比如reactive或者ref）
> - 直接写入一个可响应式的对象，reactive或者ref（比较常用的是ref）

**watch的选项：**

watch的第三个参数是一个对象，可以接收一个监听属性：

- deep：是否深度侦听(默认为true)
- immediate：是否立即执行一次(默认为false)
- flush：调整回调函数的刷新时机

```js
const info = reactive({
    name: "why",
    age: 18,
    friend: {
        name: "kobe"
    }
});

watch(() => ({ ...info }), (newInfo, oldInfo) => {
    console.log(newInfo, oldInfo);
}, {
    deep: true,
    immediate: true，
    flush: 'pre'
})

const changeData = () => {
    info.friend.name = "james";
}
```

**监听单个数据源：**

watch函数会侦听getter函数中引用的可响应式对象，当它们发生变化时，副作用函数就会执行。副作用函数中有两个参数，newValue和oldValue，这两个参数就是getter函数返回值修改前的值和修改后的值

这里有一个技巧，如果直接用watch侦听reactive对象，那么newValue和oldValue都是同一个reactive对象，因为他们都是引用同一个对象，修改后全都改变。如果想让newValue和oldValue显示正确的值，那就可以用getter函数，把reactive对象进行解构，这样newValue和oldValue就都是一个普通对象，并且由于他们都是由reactive对象解构出来的，因此不是对同一个对象的引用，而是对reactive对象修改前和修改后的一个拷贝，这样newValue和oldValue就显示正常了

```js
setup() {
    const info = reactive({ name: "why", age: 18 });

    watch(() => info.name, (newValue, oldValue) => {
        console.log("newValue:", newValue);
        console.log("oldValue:", oldValue);
    })

    watch(() => {
        return { ...info }
    }, (newValue, oldValue) => {
        console.log("newValue:", newValue);
        console.log("oldValue:", oldValue);
    })

    const changeData = () => {
        info.name = "kobe";
    }
}
```

侦听reactive对象获取到的newValue和oldValue本身都是reactive对象，而侦听ref对象获取到的newValue和oldValue是ref.value值

```js
setup() {
    const info = reactive({ name: "why", age: 18 });
    const name = ref("why");

    watch(info, (newValue, oldValue) => {
        console.log("newValue:", newValue);
        console.log("oldValue:", oldValue);
    })

    watch(name, (newValue, oldValue) => {
        console.log("newValue:", newValue);
        console.log("oldValue:", oldValue);
    })

    const changeData = () => {
        info.name = "kobe";
        name.value = "kobe";
    }
}
```

**监听多个数据源：**

watch侦听器可以使用数组同时侦听多个源，侦听多个源时newValue和oldValue也是数组，可以使用解构语法把监听的多个源解构出来

```js
setup() {
    const info = reactive({ name: "why", age: 18 });
    const name = ref("why");

    watch([() => ({ ...info }), name], ([newInfo, newName], [oldInfo, oldName]) => {
        console.log("newName", newName);
        console.log("oldName", oldName);
        console.log("newInfo", newInfo);
        console.log("oldInfo", oldInfo);
    })

    const changeData = () => {
        info.name = "kobe";
        name.value = "kobe";
    }
}
```

### provider和inject

> Options API中可以通过Provide与Inject实现非父子组件的通信，Composition API可以替代Options API中的Provide和Inject的选项：
>
> - 可以在父组件中通过 provide来提供数据，它可以传入两个参数：
>
>   - name：提供的属性名称
>
>   - value：提供的属性值
>
> - 在后代组件中可以通过 inject 来注入需要的属性和对应的值，它可以传入两个参数：
>   - 要注入的属性的name
>   - 给要注入的属性设置默认值

```vue
<template>
  <div>
    <h2>App Counter: {{ counter }}</h2>
    <button @click="increment">App +1</button>
    <home />
  </div>
</template>

<script>
import { provide, ref, readonly } from "vue";
import Home from "./About.vue";

export default {
  components: {
    Home,
  },
  setup() {
    let counter = ref(100);
    const increment = () => counter.value++;
    provide("counter", readonly(counter));

    return {
      increment,
      counter,
    };
  },
};
</script>
```

```vue
<template>
  <div>
    <h2>{{ counter }}</h2>
    <button @click="homeIncrement">home +1</button>
  </div>
</template>
<script>
import { inject } from "vue";

export default {
  setup() {
    const counter = inject("counter");
    const homeIncrement = () => counter.value++;

    return {
      counter,
      homeIncrement,
    };
  },
};
</script>
```

provide可以提供一个可响应式对象，也能提供一个普通变量，但是提供一个普通变量显然是无法触发响应式操作的

上例中provide提供了一个ref对象，子元素中inject的也是ref对象，因此，如果在父元素中操作该ref对象的value值，子元素就会受到响应式操作

但是如果子元素中操作该ref对象的value值，父元素也会受到影响。这就违反了单向数据流的约定，如果在父组件中定义的变量，在其任意子组件中都可以修改，那么排错就会变量非常困难，因此子组件想要修改父组件中的变量，应该通过$emit事件的方式，而不是这样违反单向数据流。解决方法也非常简单，只需要在父组件中把要provide的变量变成只读即可

### $ref的替代方案

首先创建一个响应式的ref对象title，其value值为null，并返回出去。然后当页面挂载时，ref="title"就会生效，他会自动把h2元素赋值给title.value

```vue
<template>
  <div>
    <h2 ref="title">哈哈哈</h2>
  </div>
</template>

<script>
import { ref, watchEffect } from "vue";

export default {
  setup() {
    const title = ref(null);
    return {
      title,
    };
  },
};
</script>
```

但是，页面挂载是在setup函数执行之后才进行的，那么我们就只能在生命周期函数中获取到h2元素。或者就是通过侦听器侦听title，只要页面挂载结束，title.value就会发生变化，从而侦听器的侦听函数也会重新执行，这样就可以获取到h2元素了

```vue
<template>
  <div>
    <h2 ref="title">哈哈哈</h2>
  </div>
</template>

<script>
import { ref, watchEffect } from "vue";

export default {
  setup() {
    const title = ref(null);
    watchEffect(
      () => {
        console.log(title.value);
      },
      {
        flush: "post | pre",
      }
    );
    return {
      title,
    };
  },
};
</script>
```

如果将flush设为pre，会发现页面挂载完毕后，控制台输出了两个值。这是因为setup函数在执行时就会立即执行一次侦听器传入的副作用函数，这个时候DOM并没有挂载，所以会打印一次null。而当DOM挂载时，会给title的ref对象赋新的值，侦听器传入的副作用函数会再次执行，打印出来对应的元素

而当把flush设为post时，页面挂载完毕后，控制台输出就正常了，这是由于这次副作用函数是在页面挂载后才执行的，所以就可以直接获取到h2元素

## custom hook

> 当我们需要在两个组件之间共享行为的场景,我们通常使用mixin。不过随着hooks的出现，现在又有另一种可选方案，我们可以使用custom hook 复用业务逻辑。cutom hook可以和Composition API配合写出非常精简漂亮的代码

- **/src/hooks/index.js**

```js
import useCounter from './useCounter';
import useTitle from './useTitle';
import useScrollPosition from './useScrollPosition';
import useMousePosition from './useMousePosition';
import useLocalStorage from './useLocalStorage';

export {
    useCounter,
    useTitle,
    useScrollPosition,
    useMousePosition,
    useLocalStorage
}
```

- **/src/hooks/useCounter.js**

```js
import {
    ref,
    computed
} from 'vue';

export default function () {
    const counter = ref(0);
    const doubleCounter = computed(() => counter.value * 2);

    const increment = () => counter.value++;
    const decrement = () => counter.value--;

    return {
        counter,
        doubleCounter,
        increment,
        decrement
    }
}
```

- **/src/hooks/useTitle.js**

```js
import {
    ref,
    watch
} from 'vue';

export default function (title = "默认的title") {
    const titleRef = ref(title);

    watch(titleRef, (newValue) => {
        document.title = newValue
    }, {
        // watch侦听是隋性的，其副作用函数只有在titleRef发生变化时才会执行，这会导致在第一次创建titleRef时，document.title不会改变，解决方法除了在watch监听函数外写document.title=title，就是让该副作用函数立即执行一次
        immediate: true
    })
    
    // document.title=title

    return titleRef
}
```

- **/src/hooks/useScrollPosition.js**

```js
import {
    ref
} from 'vue';

export default function () {
    const scrollX = ref(0);
    const scrollY = ref(0);

    document.addEventListener("scroll", () => {
        scrollX.value = window.scrollX;
        scrollY.value = window.scrollY;
    });

    return {
        scrollX,
        scrollY
    }
}
```

- **/src/hooks/useMousePosition.js**

```js
import {
    ref
} from 'vue';

export default function () {
    const mouseX = ref(0);
    const mouseY = ref(0);

    window.addEventListener("mousemove", (event) => {
        mouseX.value = event.pageX;
        mouseY.value = event.pageY;
    });

    return {
        mouseX,
        mouseY
    }
}
```

- **/src/hooks/useLocalStorage.js**

```js
import {
    ref,
    watch
} from 'vue';

export default function (key, value) {
    const data = ref(value);

    if (value) {
        window.localStorage.setItem(key, JSON.stringify(value));
    } else {
        data.value = JSON.parse(window.localStorage.getItem(key));
    }

    watch(data, (newValue) => {
        window.localStorage.setItem(key, JSON.stringify(newValue));
    })

    return data;
}

// 一个参数: 取值
// const data = useLocalStorage("name");

// 二个参数: 保存值
// const data = useLocalStorage("name", "coderwhy");

// 修改值
// data.value = "kobe";  
// 闭包，访问到的key就是上次调用useLocalStorage函数时传入的key
```

## 异步组件

### webpack的代码分包

> 默认情况下，在构建整个组件树的过程中，因为组件和组件之间是通过模块化直接依赖的，那么webpack在打包时就会将组件模块打包到一起（比如一个app.js文件中），这个时候随着项目的不断庞大，app.js文件的内容过大，会造成首屏的渲染速度变慢
>
> 所以，对于一些不需要立即使用的组件，可以单独对它们进行拆分，拆分成一些小的代码块，这些小的代码块可以在需要的时候从服务器上加载下来并且运行，显示对应的内容。
> 要通过webpack拆包也很简单，在需要某块代码的时候使用import函数对其进行异步导入，这样webpack在打包项目时就会对该块代码进行拆分打包

```js
import("./package.js").then(res => {
    console.log(res.sum(10, 20))
})
```

### 异步组件(defineAsyncComponent)

> 在Vue开发中，如果项目过大了，对于某些组件希望通过异步的方式来进行加载（目的是可以对其进行分包处理），而Vue给我们提供了一个函数defineAsyncComponent用来异步加载Vue组件。该函数接受两种类型的参数：
>
> - 工厂函数，该工厂函数需要返回一个Promise对象，该Promise对象的res是要请求的vue组件(import()返回一个Promise对象)
> - 接受一个对象类型，对异步函数进行配置

**工厂函数写法：**

```html
<script>
  import { defineAsyncComponent } from 'vue';
  const AsyncCategory = defineAsyncComponent(() => import("./AsyncCategory.vue"))

  export default {
    components: {
      AsyncCategory,
    }
  }
</script>
```

**对象写法：**

```html
<script>
import { defineAsyncComponent } from "vue";
import Home from "./Error.vue";
import Loading from "./Loading.vue";

const AsyncCategory = defineAsyncComponent({
  loader: () => import("./AsyncCategory.vue"),
  // 加载过程中显示的组件
  loadingComponent: Loading,
  // 加载失败时显示的组件
  errorComponent: Error,
  // 在显示loadingComponent组件之前的延迟 | 默认值：200(单位ms)
  delay: 2000,
  // 如果提供了timeout，并且加载组件的时间超过了设定值
  // 将显示errorComponent | 默认值： Infinity(即永不超时)
  timeout: 0,
  // 该函数会在组件加载失败时执行
  // err: 错误信息对象
  // retry: 函数, 用于指示当 promise 加载器 reject 时，加载器是否应该重试
  // fail: 函数，表示加载程序退出，不再retry
  // attempts: 记录尝试的次数
  onError: function (err, retry, fail, attempts) {
    // 请求发生错误时重试，最多可尝试 3 次
    // 注意，retry/fail 就像 promise 的 resolve/reject 一样，必须调用其中一个才能继续错误处理。
    if (error.message.match(/fetch/) && attempts <= 3) retry()
    else fail()
  },
  // 定义组件是否可以挂起 | 默认值为true，当配置项中的suspensible为true时，被Suspense包裹的异步组件将会被控制
  suspensible: true,
});

export default {
  components: {
    AsyncCategory,
    Loading,
    Error,
  },
};
</script>
```

### Suspense组件

> `defineAsyncComponent`方法提供了处理异步组件加载失败时显示备用组件的逻辑，这个逻辑是在`defineAsyncComponent`方法的`errorComponent`属性中指定
>
> 而`<suspense>`组件提供了另一个方案，允许将等待组件加载的过程提升到组件树中处理。Suspense是一个内置的全局组件，该组件有两个插槽，`#default` 和 `#fallback`。两个插槽都只允许**一个**直接子节点，在可能的时候都将显示默认槽中的节点，否则将显示后备槽中的节点

```vue
<template>
  <suspense>
    <template #default>
      <todo-list />
    </template>
    <template #fallback>
      <div> Loading... </div>
    </template>
  </suspense>
</template>

<script>
export default {
  components: {
    TodoList: defineAsyncComponent(() => import('./TodoList.vue'))
  }
}
</script>
```

**`<Suspense>` 可以等待的异步依赖有两种：**

1. 带有异步 `setup()` 钩子的组件，这也包含了使用 `<script setup>` 时有顶层 `await` 表达式的组件

```js
export default {
  async setup() {
    const res = await fetch(...)
    const posts = await res.json()
    return {
      posts
    }
  }
}
```

如果使用 `<script setup>`，那么顶层 `await` 表达式会自动让该组件成为一个异步依赖：

```html
<script setup>
const res = await fetch(...)
const posts = await res.json()
</script>

<template>
  {{ posts }}
</template>
```

2. 异步组件 `defineAsyncComponent`

异步组件默认就是**“suspensible”**的。这意味着如果组件关系链上有一个 `<Suspense>`，那么这个异步组件就会被当作这个 `<Suspense>` 的一个异步依赖。在这种情况下，加载状态是由 `<Suspense>` 控制，而该组件自己的`加载时显示的组件loadingComponent`、`报错时显示的组件errorComponent`、`延时delay`和`超时timeout`等选项都将被忽略

异步组件也可以通过在选项中指定 `suspensible: false` 表明不用 `Suspense` 控制，并让组件始终自己控制其加载状态

**事件：**

`<Suspense>` 组件会触发三个事件：`pending`、`resolve` 和 `fallback`。`pending` 事件是在进入挂起状态时触发。`resolve` 事件是在 `default` 插槽完成获取新内容时触发。`fallback` 事件则是在 `fallback` 插槽的内容显示时触发

例如，可以使用这些事件在加载新组件时在之前的 DOM 最上层显示一个加载指示器

**错误处理：**

`<Suspense>` 组件自身目前还不提供错误处理，不过你可以使用 [`errorCaptured`](https://cn.vuejs.org/api/options-lifecycle.html#errorcaptured) 生命周期选项或者 [`onErrorCaptured()`](https://cn.vuejs.org/api/composition-api-lifecycle.html#onerrorcaptured) 钩子，在使用到 `<Suspense>` 的父组件中捕获和处理异步错误

### setup顶层async

> ==setup配置选项顶层async + Suspense组件实现异步数据获取时，组件的等待切换的效果==
>
> 给组件的setup方法前加async的话，你会收到一个警告`async setup() is used without a suspense boundary!`如果你想屏蔽掉这个警告就需要在使用组件的地方（父组件内）加上suspense标签。如下面的例子中，表示的意思就是子组件setup函数尚未执行完成时，将显示`<loading />`组件

- **/src/app.vue**

```vue
<template>
  <div>
    <Suspense>
      <template #default>
        <child />
      </template>
      <template #fallback>
        <loading />
      </template>
    </Suspense>
  </div>
</template>

<script>
import { ref } from 'vue'
import Child from './components/child.vue'
    
export default {
  components: { child },
  setup() {
    return {}
  }
}
</script>
```

- **/src/components/child.vue**

```vue
<template>
  <h3>我是child组件</h3>
</template>

<script>
import axios from "axios";
import { onMounted, ref } from "vue";
    
export default {
  async setup() {
    let films = ref([]);
    let ret = await axios.get("https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10");
    films.value = ret.data.data.films;
    return { films };
  },
};
</script>
```

## h函数

> Vue推荐在绝大数情况下使用模板来创建你的HTML，然后一些特殊的场景，你真的需要JavaScript的完全编程的能力，这个时候你可以使用 渲染函数 ，它比模板更接近编译器
>
> Vue在生成真实的DOM之前，会将我们的节点转换成VNode，而VNode组合在一起形成一颗树结构，就是虚拟DOM（VDOM），事实上，我们之前编写的 template 中的HTML 最终也是使用渲染函数生成对应的VNode，那么，如果你想充分的利用JavaScript的编程能力，我们可以自己来编写 createVNode 函数，生成对应的VNode
>
> h函数是一个用于创建 vnode 的一个函数，其更准确的命名是 createVNode函数，但是为了简便，Vue将之简化为 h() 函数

### **h函数的参数：**

h函数可以接受三个参数，如果没有props，那么通常可以将children作为第二个参数传入，如果会产生歧义，那么可以将null作为第二个参数传入，将children作为第三个参数传入

### **h函数的用途：**

- 在render函数中使用

```js
import { h } from "vue";

export default {
  render() {
    return h("h2", { class: "title" }, "Hello Render");
  },
};
```

- 在setup函数中使用

```js
import { h } from "vue";

export default {
  setup() {
    return () => h("h2", { class: "title" }, "Hello Render");
  },
};
```

> 这两种写法的区别就是render中可以使用this，因此可以使用\$emit、\$root等属性，而在setup中没有this，因此他想要获取属性、插件等内容只能通过参数

### h函数实现插槽

> h函数的第三个参数可以传入一个对象，对象里内容就是要传入子组件插槽中的内容。该对象的key值就表示插槽名，而value是一个函数，返回值是一个VNode，在子组件中可以通过this.$solts[key]来获取该VNode
>
> 在子组件调用this.$solts\[key]()时可以传入参数，这个参数可以传回父组件，这样就可以实现作用域插槽

- **setup写法**

```js
import { h } from "vue";
import HelloWorld from "./HelloWorld.vue";

export default {
  setup() {
    return () =>
      h("div", null, [
        h(HelloWorld, null, {
          default: (props) => h("span", null, `app传入到HelloWorld中的内容: ${props.name}`),
        }),
      ]);
  },
};
```

```js
import { h } from "vue";

export default {
  setup(props, context) {
    return () =>
      h("div", null, [
        h("h2", null, "Hello World"),
        context.slots.default ? context.slots.default({ name: "coderwhy" }) : h("span", null, "default slot"),
      ]);
  },
};
```

- **render写法**

```js
import { h } from "vue";
import HelloWorld from "./HelloWorld.vue";

export default {
  render() {
    return h("div", null, [
      h(HelloWorld, null, {
        default: (props) => h("span", null, `app传入到HelloWorld中的内容: ${props.name}`),
      }),
    ]);
  },
};
```

```js
import { h } from "vue";

export default {
  render() {
    return h("div", null, [
      h("h2", null, "Hello World"),
      this.$slots.default ? this.$slots.default({ name: "coderwhy" }) : h("span", null, "default slot"),
    ]);
  },
};
```

## jsx语法

### jsx文件书写组件

```vue
<template>
  <child :title="title" :setTitle="setTitle"> 
    我是一个内容 
  </child>
</template>

<script>
import { ref } from "vue";
import child from "./components/child";
    
export default {
  components: { child },
  setup() {
    let title = ref("测试");
    const setTitle = (titleStr) => (title.value = titleStr);

    return { title, setTitle };
  },
};
</script>
```

> 这个函数其实和setup属性差不多，也有第二参数context，他有三个属性(slots,emit,attrs,但是没有expose)

```jsx
import { ref } from 'vue'

// 响应式属性，使用该属性的函数会被进行依赖收集，如果Chil函数中使用了这个属性，那么它就会被依赖收集，因此只要num发生改变，Child就会被重新执行
let num = ref(100)
const changeTitle = fn => fn(Date.now() + '')

// 这里的props是可以进行解构的，虽然解析会使title失去响应性，但是title根本就不应该有响应必，因为单向数据流约定，它应该只能被父组件进行修改，也就是使用setTitle进行修改
const Child = ({ title, setTitle }, context) => {
  return (
    <div>
      <h3>jsx -- {title}</h3>
      <h3>{context.slots.default()}</h3>
      <button onClick={() => setTitle(Date.now() + '')}> 
        jsx 
      </button>
      <button onClick={() => changeTitle(setTitle)}>
        jsx
      </button>
    </div>
  )
}

export default Child
```

### setup lang='jsx'

```vue
<script lang='jsx'>
import HelloWorld from "./HelloWorld.vue";
import { ref } from "vue";

export default {
  setup() {
    const counter = ref(0);
    const increment = () => this.counter++;
    const decrement = () => this.counter--;

    return () => (
      <div>
        <h2>当前计数: {this.counter}</h2>
        <button onClick={increment}>+1</button>
        <button onClick={decrement}>-1</button>
        <HelloWorld>
          {{ default: (props) => <button>button</button> }}
        </HelloWorld>
      </div>
    );
  },
};
</script>
```

```vue
<script lang='jsx'>
export default {
  setup(props, context) {
    return () => (
      <div>
        <h2>HelloWorld</h2>
        {
          context.slots.default ? 
          context.slots.default() : 
          <span>哈哈哈</span>
  		}
      </div>
    );
  },
};
</script>
```

### render lang='jsx'

```vue
<script lang='jsx'>
import HelloWorld from "./HelloWorld.vue";

export default {
  data() {
    return {
      counter: 0,
    };
  },
  render() {
    const increment = () => this.counter++;
    const decrement = () => this.counter--;
    return (
      <div>
        <h2>当前计数: {this.counter}</h2>
        <button onClick={increment}>+1</button>
        <button onClick={decrement}>-1</button>
        <HelloWorld>{{ default: (props) => <button>button</button> }}</HelloWorld>
      </div>
    );
  },
};
</script>
```

```vue
<script lang='jsx'>
export default {
  render() {
    return (
      <div>
        <h2>HelloWorld</h2>
        {this.$slots.default ? this.$slots.default() : <span>哈哈哈</span>}
      </div>
    );
  },
};
</script>
```

## v-memo

> 缓存一个模板的子树。在元素和组件上都可以使用。为了实现缓存，该指令需要传入一个固定长度的依赖值数组进行比较。如果数组里的每个值都与最后一次的渲染相同，那么整个子树的更新将被跳过

```vue
<template>
  <!-- v-memo它依赖项没有发生改变，则它子元素不会重新渲染，优化 -->
  <div v-memo="[names.length]">
    <ul v-for="item of names" :key="item"> {{ item }} </ul>
  </div>
  <button @click="addNames">++++</button>
</template>

<script>
import { ref } from 'vue'
export default {
  setup() {
    let names = ref(['乐乐', '英子', '小森', '亮亮'])
    const addNames = () => names.value.push(Date.now())

    return { num, names, addNames }
  }
}
</script>
```

当组件重新渲染，如果 `names.length` 保持不变，这个 `<div>` 及其子项的所有更新都将被跳过。实际上，甚至虚拟 DOM 的 vnode 创建也将被跳过，因为缓存的子树副本可以被重新使用

正确指定缓存数组很重要，否则应该生效的更新可能被跳过。`v-memo` 传入空依赖数组 (`v-memo="[]"`) 将与 `v-once` 效果相同

### **与v-for一起使用**

`v-memo` 仅用于性能至上场景中的微小优化，应该很少需要。最常见的情况可能是有助于渲染海量 `v-for` 列表 (长度超过 1000 的情况)：

```html
<div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
  <p>ID: {{ item.id }} - selected: {{ item.id === selected }}</p>
  <p>...more child nodes</p>
</div>
```

当组件的 `selected` 状态改变，默认会重新创建大量的 vnode，尽管绝大部分都跟之前是一模一样的。`v-memo` 用在这里本质上是 在说“只有当该项的被选中状态改变时才需要更新”。这使得每个选中状态没有变的项能完全重用之前的 vnode 并跳过差异比较。注意这里 memo 依赖数组中并不需要包含 `item.id`，因为 Vue 也会根据 item 的 `:key` 进行判断

> 当搭配 `v-for` 使用 `v-memo`，确保两者都绑定在同一个元素上。**`v-memo` 不能用在 `v-for` 内部**

## Teleport

> 在组件化开发中，我们封装一个组件A，在另外一个组件B中使用，那么组件A中template的元素会被挂载到组件B中template的某个位置，最终会形成一颗DOM树结构
>
> 但是某些情况下，我们希望组件不是挂载在这个组件树上的，可能是移动到Vue app之外的其他位置，比如移动到body元素上，或者我们有其他的div#app之外的元素上，这个时候我们就可以通过teleport来完成：
>
> - 它是一个Vue提供的内置组件，类似于react的Portals，这个组件中的内容可以指定挂载位置
> - 在 teleport 中可以使用组件，并且可以给他传入数据
> - 如果我们将多个teleport应用到同一个目标上（to的值相同），那么这些目标会进行合并
> - teleport组件有两个属性：
>   - to：指定将其中的内容移动到的目标元素，可以使用选择器
>   - disabled：是否禁用 teleport 的功能

```vue
<template>
  <div>
    <button @click="isShow = !isShow">
      点击显示
    </button>
    <Teleport to="#other">
      <div v-if="isShow" class="modal">
        <h3>提示框</h3>
        <div>欢迎登录页面</div>
      </div>
    </Teleport>
  </div>
</template>

<script>
import { ref } from 'vue'
export default {
  setup() {
    let isShow = ref(false)
    return { isShow }
  }
}
</script>
```

## \<script setup\>

### \<script setup\>基础语法

> `<script setup>` 是在单文件组件 (SFC) 中使用组合式 API 的编译时语法糖：

要使用这个语法，需要将 setup attribute 添加到 \<script\> 代码块上：

```html
<script setup>
  console.log('hello script setup')
</script>
```

当使用 \<script setup\> 的时候，任何在 \<script setup\> 声明的顶层的绑定 (包括变量，函数声明，以及 import 引入的内容) 都能在模板中直接使用，import 导入的内容也会以同样的方式暴露。意味着可以在模板表达式中直接使用导入的函数，并不需要通过 methods 选项来暴露它：
```html
<script setup>
    import { capitalize } from './helpers'
    const msg = 'Hello!'
    function log() {
        console.log(msg)
    }
</script>

<template>
    <div>{{ capitalize('hello') }}</div>
    <div @click="log">{{ msg }}</div>
</template>
```

### 响应式

响应式状态需要明确使用响应式 APIs 来创建。和从 setup() 函数中返回值一样，ref值在模板中使用的时候会自动解包：

```html
<script setup>
    import { ref } from 'vue'
    const count = ref(0)
</script>

<template>
    <button @click="count++">{{ count }}</button>
</template>
```

### 使用组件

\<script setup\> 范围里的值也能被直接作为自定义组件的标签名使用，在模板中可以使用kebab-case格式(\<my-component\>)，不过vue建议使用PascalCase格式：

```html
<template>
    <MyComponent />
</template>

<script setup>
    import MyComponent from './MyComponent.vue'
</script>
```

**递归组件**

一个单文件组件可以通过它的文件名被其自己所引用。例如：名为 `FooBar.vue` 的组件可以在其模板中用 `<FooBar/>` 引用它自己

请注意这种方式相比于导入的组件优先级更低。如果有具名的导入和组件自身推导的名字冲突了，可以为导入的组件添加别名：

```js
import { FooBar as FooBarChild } from './components'
```

**动态组件**

由于组件是通过变量引用而不是基于字符串组件名注册的，在 `<script setup>` 中要使用动态组件的时候，应该使用动态的 `:is` 来绑定：

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

**命名空间组件**

可以使用带 `.` 的组件标签，例如 `<Foo.Bar>` 来引用嵌套在对象属性中的组件。这在需要从单个文件中导入多个组件的时候非常有用：

```vue
<script setup>
import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

### 使用自定义指令

> 全局注册的自定义指令将正常工作。本地的自定义指令在 `<script setup>` 中不需要显式注册，但他们必须遵循 `vNameOfDirective` 这样的命名规范

```vue
<script setup>
// 直接定义
const vMyDirective = {
  beforeMount: (el) => {
    // 在元素上做些操作
  }
}
</script>

<script setup>
// 如果指令是从别处导入的，可以通过重命名来使其符合命名规范
import { myDirective as vMyDirective } from './MyDirective.js'
</script>

<template>
  <h1 v-my-directive>This is a Heading</h1>
</template>
```

### defineProps()和defineEmits()

```vue
<template>
  <!-- 由于传入到defineProps的选项会从 setup 中提升到模块的作用域中，因此这里可以直接使用title -->
  <div>{{ title }}</div>
</template>

<script setup>
const props = defineProps({
  title: String,
}); 
const emit = defineEmits(['change', 'delete'])
console.log(props.title) // 打印props参数
</script>
```

- `defineProps` 和 `defineEmits` 都是只能在 `<script setup>` 中使用的**编译器宏**。他们不需要导入，且会随着 `<script setup>` 的处理过程一同被编译掉
- `defineProps` 接收与 `props` 选项相同的值，`defineEmits` 接收与 `emits` 选项相同的值
- `defineProps` 和 `defineEmits` 在选项传入后，会提供恰当的类型推导
- 传入到 `defineProps` 和 `defineEmits` 的选项会从 setup 中提升到模块的作用域。因此，传入的选项不能引用在 setup 作用域中声明的局部变量。这样做会引起编译错误。但是，它可以引用导入的绑定，因为它们也在模块作用域内

- 如果使用了 TypeScript，使用纯类型声明来声明 `prop` 和 `emit` 也是可以的

### defineExpose

使用 `<script setup>` 的组件是**默认关闭**的 —— 即通过模板引用或者 `$parent` 链获取到的组件的公开实例，**不会**暴露任何在 `<script setup>` 中声明的绑定。可以通过 `defineExpose` 编译器宏来显式指定在 `<script setup>` 组件中要暴露出去的属性：

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({ a, b })
</script>
```

当父组件通过模板引用的方式获取到当前组件的实例，获取到的实例会像这样 `{ a: number, b: number }` (ref 会和在普通实例中一样被自动解包)

### useSlots()和useAttrs()

在 `<script setup>` 使用 `slots` 和 `attrs` 的情况应该是相对来说较为罕见的，因为可以在模板中直接通过 `$slots` 和 `$attrs` 来访问它们。在你的确需要使用它们的罕见场景中，可以分别用 `useSlots` 和 `useAttrs` 两个辅助函数：

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` 和 `useAttrs` 是真实的运行时函数，它的返回与 `setupContext.slots` 和 `setupContext.attrs` 等价，它们同样也能在普通的组合式 API 中使用

### 与普通的 `<script>` 一起使用

> \<script setup> 可以和普通的 \<script> 一起使用。普通的 \<script> 在有这些需要的情况下或许会被使用到：
>
> - 声明无法在 `<script setup>` 中声明的选项，例如 `inheritAttrs` 或插件的自定义选项
> - 声明模块的具名导出 (named exports)
> - 运行只需要在模块作用域执行一次的副作用，或是创建单例对象

```vue
<script>
// 普通 <script>, 在模块作用域下执行 (仅一次)
runSideEffectOnce()

// 声明额外的选项
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// 在 setup() 作用域中执行 (对每个实例皆如此)
</script>
```

### 顶层 await

> \<script setup> 中可以使用顶层 await。结果代码会被编译成 async setup()。`async setup()` 必须与 `Suspense` 内置组件组合使用

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

另外，await 的表达式会自动编译成在 `await` 之后保留当前组件实例上下文的格式

### 针对TypeScript 的功能

#### props/emit的类型声明

> props 和 emit 都可以通过给 `defineProps` 和 `defineEmits` 传递纯类型参数的方式来声明。`defineProps` 或 `defineEmits` 要么使用运行时声明，要么使用类型声明。同时使用两种声明方式会导致编译报错。使用类型声明的时候，静态分析会自动生成等效的运行时声明，从而在避免双重声明的前提下确保正确的运行时行为

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
```

#### 使用类型声明时的默认props值

> 针对类型的 `defineProps` 声明的不足之处在于，它没有可以给 props 提供默认值的方式。为了解决这个问题，我们还提供了 `withDefaults` 编译器宏。它会被编译为等价的运行时 props 的 `default` 选项。此外，`withDefaults` 辅助函数提供了对默认值的类型检查，并确保`props`的类型删除了已声明默认值的属性的可选标志（ ? ）

```js
export interface Props {
  msg?: string
  labels?: string[]
}
// 由于msg 和 labels都有默认值，因此withDefaults会删除掉它们两个类型中的可选标志
const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

### setup应用实例

```vue
<template>
    <div>
        <h2>当前计数: {{ counter }}</h2>
        <h2>子组件暴露属性: {{ childRef }}</h2>
        <button @click="increment" v-focus>+1</button>

        <hello-world message="子组件" 
            @increment="chlidIncrement" 
            ref="childRef">
        </hello-world>
    </div>
</template>

<script setup>
    import { ref } from 'vue';
    import HelloWorld from './About.vue';

    const counter = ref(0);
    const childRef = ref(null);

    const increment = () => counter.value++;
    const chlidIncrement = (value) => {
        console.log(value);
        counter.value++
    }

    // 自定义指令
    const vFocus = {
        mounted: (el) => {
            el.focus()
        }
    }
</script>
```

```vue
<template>
    <div>
        <h2>{{ message }}</h2>
        <button @click="emitEvent">emit: +1</button>
    </div>
</template>

<script setup>
    import { ref } from 'vue';

    // 暴露变量
    const expose1 = "exposeValue"
    const expose2 = ref("exposeValue")
    defineExpose({
        expose1,
        expose2
    })

    // 接收props
    const props = defineProps({
        message: {
            type: String,
            default: "default"
        }
    })

    // 发射事件
    const emit = defineEmits(["increment"]);
    const emitEvent = () => {
        emit('increment', "100000")
    }
</script>
```

