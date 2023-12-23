# Nuxt

> Nuxt.js 是一个基于 Vue.js 的通用应用框架。通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的是应用的 UI渲染。Nuxt.js 预设了利用 Vue.js 开发服务端渲染的应用所需要的各种配置。作为框架，Nuxt.js 为 客户端/服务端 这种典型的应用架构模式提供了许多有用的特性，例如异步数据加载、中间件支持、布局支持等
>
> **Nuxt.js 集成了以下组件/框架，用于开发完整而强大的 Web 应用：**
>
> - Vue 2
>
> - Vue-Router 【约定式路由】
>
> - Vuex (当配置了 Vuex 状态树配置项 时才会引入)
>
> - Vue 服务器端渲染 
>
> - Vue-Meta
>
> Nuxt脚手架压缩并 gzip 后，总代码大小为：57kb （如果使用了 Vuex 特性的话为 60kb）。另外，Nuxt.js 使用 **Webpack** 和 vue-loader 、 babel-loader 来处理代码的自动化构建工作（如打包、代码分层、压缩等等）

## **Nuxt工作流程**

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/f1ad213as.png" alt="f1ad213as" style="zoom:50%;" />

## **布置开发环境**

安装：`npm init nuxt-app <project-name>`

## components全局组件

> Nuxt脚手架中有一个components文件夹，约定这个文件夹中定义的组件会在打包时自动被引入为全局组件，因此我们在这里定义的组件不需要引入就可以直接使用：

- **/components/filmItem.vue**

```vue
<template>
  <ul>
    <li v-for="item in films" :key="item.filmId">{{ item.name }}</li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      films: []
    }
  },
  // fetch它可以运行在任何的组件中
  async fetch() {
    let ret = await this.$axios.get('/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10')
    this.films = ret.data.data.films
  }
}
</script>
```

- **/pages/film.vue**

```vue
<template>
  <film-item />
  <nuxt-link to="/">首页</nuxt-link>
</template>
```

## Nuxt路由(约定式路由)

> Nuxt.js 依据 pages 目录结构自动生成 vue-router 模块的路由配置。要在页面之间使用路由，建议使用**<nuxt-link>**标签
>
> 虽然写法和单页面应用没什么区别，但是我们在访问路由时，它渲染出来的其实是一个个单独的html文件

### 声明式路由

> 访问`/about`会匹配到这条路由,`nuxt-link`组件与`router-link`组件的功能完全相同

- **/pages/about.vue**

```vue
<template>
  <nuxt-link to="/about">关于</nuxt-link> |
  <nuxt-link to="/film">电影</nuxt-link>
</template>
```

### 编程式路由

> 访问`/index`会匹配到这条路由

- **/pages/index.vue**

```vue
<template>
  <button @click="goHome">首页</button>
</template>

<script>
export default {
  methods: {
    goHome() {
      this.$router.push('/')
    }
  }
}
</script>
```

### 动态路由参数

> 访问`/news/:id => /news/100`会匹配到这条路由

- **/pages/news/_id.vue**

```vue
<template>
  <h3>根据id来得到当前页面</h3>
</template>

<script>
export default {
  // 服务器渲染时可以对参数进行验证
  validate({ params, redirect }) {
    // 当为true时，则继续向下渲染，false停止
    if (/\d+/.test(params.id)) {
      return true
    }
    return redirect('/err') // 重定向到err页面
  }
}
</script>
```

### 嵌套路由

> 访问`/admin/user`会匹配到这条路由

- **/pages/admin/user.vue**

```vue
<template>
  <h3>user</h3>
</template>

<script>
export default {}
</script>
```

- **/pages/admin.vue**

> 如果它是一个嵌套路由，则在此处一定要放置挂载点`nuxt-child`

```vue
<template>
  <div>
    <h3>我是一个后台</h3>
    <nuxt-child />
  </div>
</template>
```

### 全匹配路由

> `_.vue`表示匹配所有路由，nuxt在编译时会把它放在所有路由配置的最后面(除了动态添加的路由)，因此可以把它用做404页面路由

- **/pages/_.vue**

```vue
<template>
  <h3>404页面</h3>
</template>

<script>
export default {}
</script>
```

### 配置式路由

> Nuxt虽然是约定式路由，但是也可以使用配置式路由，在nuxt.config.js中进行配置：

- **/nuxt.config.js**

```js
export default {
  // 进行配置式路由
  router: {
    extendRoutes(routes, resolve) {
      routes.push({
        path: '/abc',
        component: resolve(__dirname, 'pages/about.vue')
      })
    }
  },
}
```

如果要动态添加路由，一定要注意在pages中不能有\_.vue这样的文件，否则路由匹配不成功(使用push动态添加)。因为如果使用了\_.vue，编译后的路由其实是这样的

- **/.nuxt/router.js**

```js
export const routerOptions = {
  mode: 'history',
  base: '/',
  linkActiveClass: 'nuxt-link-active',
  linkExactActiveClass: 'nuxt-link-exact-active',
  scrollBehavior,

  routes: [{ // 嵌套路由
    path: "/admin",
    component: _6c333972,
    name: "admin",
    children: [{
      path: "user",
      component: _67b77dd2,
      name: "admin-user"
    }]
  }, { // 动态路由参数
    path: "/news/:id?",
    component: _72dda982,
    name: "news-id"
  }, { // 根路由
    path: "/",
    component: _cf11e16c,
    name: "index"
  }, { // 全匹配路由
    path: "/*",
    component: _cf12e16c,
    name: "all"
  }, { // 动态插入的路由
    path: "/abc",
    component: _78bf3485
  }],
  fallback: false
}
```

可以看到动态插入的路由被插入到了全匹配路由的下面，因此它永远不会被匹配到，要解决也很简单 ，只需要把它插入到全匹配路由上面即可(使用unshift方法)

```js
export default {
  // 进行配置式路由
  router: {
    extendRoutes(routes, resolve) {
      routes.unshift({
        path: '/abc',
        component: resolve(__dirname, 'pages/about.vue')
      })
    }
  },
}
```

## Nuxt视图

> Nuxt在nuxt.config.js文件下配置了应用模板的一些信息，但是这种配置不是很直观，并且不能写内容(没有body)。因此nuxt还赋予了我们定制化 Nuxt.js 默认应用模板的能力。定制化默认的 html 模板，只需要在 src 文件夹下（默认是应用根目录）创建一个 app.html 的文件

- **默认模板配置 - /nuxt.config.js**

```js
export default {
  // Global page headers: https://go.nuxtjs.dev/config-head
  // vue-meta
  head: {
    title: 'nuxt实现服务器端渲染项目',
    htmlAttrs: {
      lang: 'en'
    },
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: '' },
      { name: 'format-detection', content: 'telephone=no' }
    ],
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  }
}
```

- **/app.html**

> 这个模板就是所有路由匹配到页面的基页，可以在这里写一些公共的东西，比如reset.css等

```html
<!DOCTYPE html>
<html {{ HTML_ATTRS }}>
  <head {{ HEAD_ATTRS }}>
    {{ HEAD }}
  </head>
  <body {{ BODY_ATTRS }}>
    {{ APP }} // APP表示路由匹配到的组件
  </body>
</html>
```

## Nuxt布局

> Nuxt.js 允许你扩展默认的布局，或在 layout 目录下创建自定义的布局。可通过添加 layouts/**default**.vue 文件来扩展应用的默认布局。别忘了在布局文件中添加 **<nuxt/>** 组件用于显示页面的主体内容

### 全局布局

> 在根目录下创建一个layouts文件夹，并在下面创建一个default.vue文件，文件夹与文件的名字都是约定好的，不能改变。全局布局与全局模板app.html的功能是相同的，如果组件之间有相同的ui内容或页面逻辑就可以在这里定义

- **/layouts/default.vue**

```vue
<template>
  <div>
    <h3>我是一个默认的全局布局文件</h3>
    <nuxt />
  </div>
</template>

<script>
export default {}
</script>
```

### 局部布局

> 定义了全局布局后就有一个问题，那就是所有组件都会应用这个布局，非常不灵活，因此nuxt给我们的组件中提供了一个属性：layout，让我们可以自定义指定布局文件

- **/pages/about.vue**

``` vue
<template>
  <h3>关于页面 /about </h3>
</template>

<script>
export default {
  layout: 'empty' // 指定布局文件
}
</script>
```

- **/layouts/empty.vue**

```vue
<template>
  <nuxt />
</template>

<script>
export default {}
</script>
```

### 错误页面

> 你可以通过编辑`layouts/error.vue`文件来定制化错误页面。这个布局文件不需要包含 `<nuxt/>` 标签。你可以把这个布局文件当成是显示应用错误（404，500 等）的组件
>
> 它与`pages/_.vue`完全不同，`pages/_.vue`表示的是全匹配路由，我们只是利用这个全匹配路由的特性匹配所有找没有定义的路由，然后来实现404页面，但是其实Nuxt认识这是匹配到路由了，返回的状态码还是200，也就是说`pages/_.vue`根本就没有表示404的语义
>
> 因此Nuxt专门提供了一个显示错误的自定义布局文件，它会接收一个error参数，然后我们就可以通过error属性的错误码和错误信息来自定义显示错误页面了
>
> **注意：**全匹配路由的优先级肯定是要比错误页面的优先级高的，因为只要使用了全匹配路由，就意味着没有匹配不到的路由，因此也就不可能出现页面404的情况

```vue
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">页面不存在</h1>
    <h1 v-else>应用发生错误异常</h1>
    <nuxt-link to="/">首 页</nuxt-link>
  </div>
</template>

<script>
  export default {
    props: ['error'],
    layout: 'blog' // 你可以为错误页面指定自定义的布局
  }
</script>
```

## 异步数据

> nuxt.config.js下面进行了如下配置，也就是说它默认帮我们引入了axios，这样我们就可以直接在组件内通过this.$axios来获取到axios对象：

```js
export default {
  // Modules: https://go.nuxtjs.dev/config-modules
  modules: [
    // https://go.nuxtjs.dev/axios
    '@nuxtjs/axios',
  ],
  // Axios module configuration: https://go.nuxtjs.dev/config-axios
  axios: {
    baseURL: 'https://api.iynn.cn/film',
  }
}
```

### mounted

> `Nuxt`的数据请求是放在`asyncData`钩子里的，而`Vue`项目一般是放在`mounted`钩子里，区别在于：
>
> - `asyncData`是早在页面挂在前就执行的，即在服务端就拿到数据然后渲染
> - `mounted`是页面在客户端挂载完成后去请求的数据，然后在客户端进行的前端渲染

- **/pages/film.vue**

```vue
<script>
export default {
  data() {
    return { films: [] };
  },
  async mounted() {
    // 这里要设置跨域(cors=T)，因为在客户端进行请求是一定会跨域的
    let ret = await this.$axios.get("https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10");
    this.films = ret.data.data.films;
  }
};
</script>
```

### asyncData

> asyncData 可以用来在客户端加载 Data 数据之前对其做一些处理，也可以在此发起异步请求，提前设置数据，这样在客户端加载页面的时候，就会直接加载提前渲染好并带有数据的 DOM，完成服务端渲染，有助于搜索引擎的抓取
>
> **注意事项：**
>
> 1. 由于在客户端创建实例化之前加载，所以**不能使用 this**，钩子提供一个参数，可以获取上下文对象(`{isDev, route, store, env, params, query, req, res, redirect, error}`等)，从而做一些简单操作
> 2. **只能在路由页面组件中使用**(每次加载页面都会调用)，在自定义组件中无效
> 3. 返回的数据最终将与 data 数据合并，为了保证不发生页面渲染错误，**返回的键应事先在 data 里声明好**(如果 template 中没有使用所需属性，则并不必声明)
> 4. 钩子在路由转换期间解析，所以在 return 之前会一直等待内部逻辑处理，**会阻滞页面加载**。如果要抛出异常，可以使用参数提供的 error 方法
> 5. **asyncData 必须返回一个对象**(`Object`)，否则将导致页面无法加载

- **/pages/film.vue**

```vue
<template>
  <film-item :films="films" />
  <nuxt-link to="/">首页</nuxt-link>
</template>

<script>
export default {
  data() {
    return {
      films: [],
    };
  },
  async asyncData({ $axios }) {
    let ret = await $axios.get("https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cityId=110100&pageNum=1&pageSize=10")
    return ret.data.data;
  },
};
</script>
```

- **/components/filmItem.vue**

```vue
<template>
  <ul>
    <li v-for="item in films" :key="item.filmId">
      {{ item.name }}
    </li>
  </ul>
</template>

<script>
export default {}
</script>
```

### fetch

> fetch 方法用于在渲染页面前填充应用的状态树（store）数据，运行于组件实例创建之后(created)页面渲染完成之前(mounted)，与 asyncData 方法类似，不同的是它不会设置组件的数据
>
> 它可以在任何组件中使用，无论是页面组件(pages文件夹下的组件)还是自定义组件(components文件夹下的组件)

- **components/filmItem.vue**

```vue
<template>
  <ul>
    <li v-for="item in films" :key="item.filmId">
      {{ item.name }}
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      films: []
    }
  },
  async fetch() {
    let ret = await this.$axios.get('/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10')
    this.films = ret.data.data.films
  }
}
</script>
```

- **/pages/film.vue**

```vue
<template>
  <film-item />
  <nuxt-link to="/">首页</nuxt-link>
</template>
```

### Nuxt路由切换后asyncData跨域报错

> `asyncData` 方法会在组件（限于页面组件）每次加载之前被调用。它可以在服务端或路由更新之前被调用。 但这个方法并不是每次都从服务端渲染，只有当页面刷新时才从服务端渲染，而在客户端点击链接跳转时并不从服务端渲染，而是和标准 Vue 单页面应用一样，从客户端进行渲染
>
> 所以这就是为什么每次我们点击路由跳转时，如果我们的目标页面有 `asyncData` 获取异步数据的时候，就会在浏览器端报跨域错误，因此就算使用了nuxt做ssr，也是需要做跨域的，如果是开发环境就直接用代理服务器，如果是生产环境，就找后端做跨域

```js
export default {
  // ... ...
  axios: {
    // 可以通过判断环境来配置baseURL
    // baseURL: 'https://api.iynn.cn/film',
    proxy: true
  },
  proxy: {
    '/api': {
      target: 'https://api.iynn.cn/film',
      pathRewrite: {
        '^/api' : ''
      },
      changeOrigin: true
    }
  },
  server: { // 配置开发服务器的端口
    port: 8080, // 这块要自定义,否则多个肯定有端口冲突
    host: '0.0.0.0', // 注意：这里有个坑，host 写成127.0.0.1 无法访问
  }
}
```

## vuex

> Nuxt.js 会尝试找到 src 目录（默认是应用根目录）下的 store 目录，如果该目录存在，它将做以下的事情：
>
> - 引用 vuex 模块
>
> - 将 vuex 模块 加到 vendors 构建配置中去
>
> - 设置 Vue 根实例的 store 配置项

### 计数器实例

- **/pages/count/index.vue**

```vue
<template>
  <div>{{ num }} -- {{ showNum }}</div>
  <button @click="addNum">加一</button>
</template>

<script>
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
export default {
  computed: {
    ...mapState("count", ["num"]),
    ...mapGetters("count", ["showNum"]),
    num1() {
      return this.$store.state.count.num;
    },
    showNum1() {
      return this.$store.getters.showNum;
    },
  },
  methods: {
    ...mapMutations("count", ["addNum"]),
    ...mapActions("count", ["asyncAddNum"]),
    addNum1() {
      this.$store.commit("count/addNum", 1);
    },
    asyncAddNum1() {
      this.$store.dispatch("count/asyncAddNum", 1);
    },
  },
};
</script>
```

- **/store/count/state.js**

```js
export default () => ({
  num: 200
})
```

- **/store/count/getter.js**

```js
export default {
  showNum(state) {
    return state.num + '@@@'
  }
}
```

- **/store/count/mutations.js**

```js
export default {
  addNum(state, payload) {
    state.num += payload
  }
}
```

- **/store/count/actions.js**

```js
export default {
  asyncAddNum({ commit }, payload) {
    setTimeout(() => {
      commit('addNum', payload)
    }, 1000);
  }
}
```

### vuex持久化

> 只要是在服务端渲染，那么一定是通过cookie进行保存数据的，因为cookie是保存在客户端的，并且用户请求页面时都会携带cookie，因为非常适合用于保存用户数据

- **/pages/login.vue**

> 在nuxt生命周期中，一个组件会先在服务端进行注入数据并渲染，然后发送到客户端进行挂载。这也就是说如果在组件中引入一个库，那么它会在服务端和客户端都引入一次，这样就会影响性能，因此nuxt给我们提供了两个个变量`process.server`和`process.client`，它们用于渲染环境的判断，比如在客户端渲染时process.client就为true

- **/static/login.json**

```json
{
  "code":0,
  "msg":"ok",
  "data":{
    "uid":1000,
    "token":"fewjlfjewklfewj;fewj;few;"
  }
}
```

- **/pages/login.vue**

```vue
<template>
  <input type="text" v-model="formData.username">
  <input type="text" v-model="formData.password">
  <button @click="doLogin">进入系统</button>
</template>

<script>
// 只在客户端渲染时引入，因为它只在客户端时使用
const cookie = process.client ? require('js-cookie') : null

export default {
  layout: 'empty',
  data() {
    return {
      formData: {
        username: 'admin',
        password: 'admin888'
      }
    }
  },
  methods: {
    async doLogin() {
      // 进行登录请求的验证
      let ret = await this.$axios.get('/login.json')
      // 登录成功，写入到cookie中，进行vuex数据持久化处理
      cookie.set('token', ret.data.data.token)
      cookie.set('uid', ret.data.data.uid)
      // 写入到vuex中
      this.$store.commit('user/setUserInfo', ret.data.data)
      // 跳转到后台
      this.$router.push('/admin')
    }
  }
}
</script>
```

- **/store/user/state.js**

```js
export default () => ({
  uid: 0,
  token: ''
})
```

- **/store/user/mutations.js**

```js
export default {
  setUserInfo(state, payload) {
    state.uid = payload.uid
    state.token = payload.token
  }
}
```

- **/store/index.js**

> 在nuxt生命周期中，请求页面时(刷新页面也会重新请求)，nuxtServerInit就会被执行一次，因此可以在这里解析客户端发来的cookie，把解析出来的用户数据补回vuex中，这样就实现了vuex的数据持久化

```js
export const actions = {
  nuxtServerInit({ commit }, { req }) {
    // 这里要try一下是因为可能cookie是undefined，对undefined使用split会报错
    try {
      let cookieStr = req.headers.cookie
      let jsonCookie = cookieStr.split(';').reduce((p, c) => {
        let [key, value] = c.split('=')
        p[key.trim()] = value
        return p
      }, {})
      // 同步到vuex中 -- 解决刷新丢失问题
      commit('user/setUserInfo', jsonCookie)
    } catch (error) {}
  }
}
```

## middleware

> 所有的中间件都应该放在middleware/目录下，文件名的名称将成为中间件的名称(middleware/auth.js将成为auth中间件)，**所以的请求都会经过该中间件**

- **/middleware/checklogin.js**

```js
export default ({ store, redirect }) => {
  if (!store.state.user.token) {
    redirect('/login')
  }
}
```

### nuxt.config.js

> 所有的路由页面都会经过该中间件

```js
export default {
  // ... ...
  router: {
    middleware: 'checklogin', // 全局中间件注册
    extendRoutes(routes, resolve) {
      routes.unshift({
        path: '/abc',
        component: resolve(__dirname, 'pages/about.vue')
      })
    }
  },
}
```

### 匹配布局

> **引入该布局(layout)的路由页面会经过该中间件**

- **/layouts/default.vue**

```vue
<template></template>

<script>
export default {
  middleware: 'checklogin'
}
</script>
```

### 匹配页面

> **指定路由页面和它的子路由页面才会经过该中间件**

- **/pages/admin.vue**

```vue
<template>
  <div>
    <h3>我是一个后台</h3>
    <!-- 如果它是一个嵌套路由，则在此处一定要放置挂载点 -->
    <nuxt-child />
  </div>
</template>

<script>
export default {
  // 写在父路由中，他的所有子路由页面都会经过该中间件
  middleware: 'checklogin'
}
</script>
```

## 上线流程

### 检查跨域配置

```js
export default {
  // ... ...
  axios: {
    // 可以通过判断环境来配置baseURL
    // baseURL: 'https://api.iynn.cn/film',
    proxy: true
  },
  proxy: {
    '/api': {
      target: 'https://api.iynn.cn/film',
      pathRewrite: {
        '^/api' : ''
      },
      changeOrigin: true
    }
  },
  server: { // 配置开发服务器的端口
    port: 8080, // 这块要自定义,否则多个肯定有端口冲突
    host: '0.0.0.0', // 注意：这里有个坑，host 写成127.0.0.1 无法访问
  }
}
```

### 打包nuxt项目

- **npm run build**
- 将.nuxt、static、nuxt.config.js、package.json、app.html等文件打包上传到服务器中
- 在服务器中执行npm install安装依赖
- 接着npm start启动项目

这时页面就能正常运行了，此时输入你服务器的ip地址和刚刚配置的端口ip:port就可以访问页面了。 不过这个时候，我们的项目运行依赖终端，不能关闭终端，所以我们要使用pm2守护进程

### 守护进程

- npm install pm2 -g

| 命令                   | 描述                   |
| ---------------------- | ---------------------- |
| pm2 list               | 当前守护的进程列表     |
| pm2 stop app_name      | 停止守护 app_name 进程 |
| pm2 delete app_name    | 删除 app_name 进程     |
| pm2 start process_name | 守护 process_name 进程 |
| pm2 restart app-name   | 重启一个应用           |

- 打开`package.json`，配置一个 `pm2` 命令：

```bash
# start表示进程名 
# --interpreter bash表示解释程序使用bash
# --name oitboy-front表示进程名叫oitboy-front
# -- start表示传给要守护的进程yarn的参数
pm2 start yarn --interpreter bash --name oitboy-front -- start
```

### 反向代理

> 当前我们nuxt项目的服务器端口为8080，不是浏览器的默认访问端口，我们可以直接把它改成80，但是这样不是最优解，我们一般会在服务器中再开启一个nginx服务，用于反向代理所有的web服务

```txt
upstream nodenuxt {
    server 127.0.0.1:3001; #nuxt项目 监听端口
    keepalive 64;
}
server {
    listen 80;
    server_name oitboy.com;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_cache_bypass $http_upgrade;
        proxy_pass http://nodenuxt; #反向代理
    }
}
```

