# vue打包知识

## Vue源码打包后的不同版本

![d0916b37fe87a61cd6fe9bdb84b99a20d1120c99c70a62ad0de556b8342a2c2d](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/d0916b37fe87a61cd6fe9bdb84b99a20d1120c99c70a62ad0de556b8342a2c2d.png)

**版本命名规则：**

vue(.runtime).global(.prod).js：

- 通过浏览器中的 <script src="..."> 直接使用；
- 我们之前通过CDN引入和下载的Vue版本就是这个版本；
- 这个版本会暴露一个全局变量Vue出来；

vue(.runtime).esm-browser(.prod).js：

- 用于通过原生 ES 模块导入使用 (在浏览器中通过 <script type="module"> 来使用)；

vue(.runtime).esm-bundler.js：

- 用于 webpack，rollup 和 parcel 等构建工具；
- 构建工具中默认是vue.runtime.esm-bundler.js；
- 如果我们需要解析模板template，那么需要手动指定vue.esm-bundler.js；

vue.cjs(.prod).js：

- 服务器端渲染使用；
- 通过require()在Node.js中使用；

**补充：**

- .runtime选项表示仅运行时，他没有包含对template版本的编译代码，相对更小一些。如果不加这个选项则表示运行时+编译器包含了对template模板的编译代码，更加完整，但是也更大一些。
- .prod选项的全称是production，他表示代码是经过压缩后的，这样的代码适合于生产环境，因为他的体积会更小。
- .esm-browser表示ESModule-browser，而esm-bundler表示ESModule-bundler。

**在Vue的开发过程中我们有三种方式来编写DOM元素：**

- template模板的方式；
- render函数的方式，使用h函数来编写渲染的内容；
- 通过.vue文件中的template来编写模板；

**它们的模板分别是如何处理的呢？**

- 方式二中的h函数可以直接返回一个虚拟节点，也就是Vnode节点；
- 方式一和方式三的template都需要有特定的代码来对其进行解析。方式三.vue文件中的template可以通过在vue-loader对其进行编译和处理。而方式一中的template我们必须要通过源码中一部分代码来进行编译；

所以，Vue在让我们选择版本的时候分为 运行时+编译器 和 仅运行时，运行时+编译器包含了对template模板的编译代码，更加完整，但是也更大一些，仅运行时没有包含对template版本的编译代码，相对更小一些。

```js
import { createApp } from "vue"

const App = createApp({
    template: "<h1>Hello Vue!</h1>"
})

App.mount("#app")
```

上面的代码通过webpack打包后，在页面上是没有任何显示的。并且我们查看运行的控制台，会发现如下的警告信息：

```txt
[Vue warn]: Component provided template option but runtime compilation is not supported in this build of Vue. Configure your bundler to alias "vue" to "vue/dist/vue.esm-bundler.js"  
```

大概意思是App组件使用了template选项，但是vue.runtime.esm-bundler.js不支持运行时编译，应该使用vue.esm-bundler.js。这样报错的原因就明了了，就是因为webpack对vue进行打包时，发现我们导入了vue模块(import { createApp } from "vue")。webpack就会默认使用vue.runtime.esm-bundler.js，而不是使用vue.esm-bundler.js，由于vue.runtime.esm-bundler.js不支持运行时编译，因此就会报错。

解决方法也很简单，只需要在导入Vue的语句中指定vue.esm-bundler.js即可：

```js
import { createApp } from "vue/dist/vue.esm-bundler"
```

## 打包vue-sfc

- **src/index.js**

> 下面代码中vue默认导入的是vue.runtime.esm-bundler.js。这里之所以不使用vue.esm-bundler.js，是因为我们这个案例中的vue代码是写进了.vue文件中。而.vue文件中的template可以通过vue-loader对其进行编译和处理，因此就不需要使用vue.esm-bundler.js了。

```js
import { createApp } from "vue"
import ComponentA from "./vue/ComponentA.vue"

const App = createApp(ComponentA)
App.mount("#app")
```

- **src/vue/ComponetA.vue**

```vue
<template>
    <div>
        <h1>{{message}}</h1>
        <component-b></component-b>
    </div>
</template>

<script>
import ComponentB from "./ComponentB.vue"

export default {
    components:{
        ComponentB: ComponentB
    },
    data() {
        return {
            message: "Hello Vue!"
        }
    }
}
</script>

<style scoped>
h1 {
    color: red;
}
</style>
```

- **src/vue/ComponetB.vue**

```vue
<template>
    <h1>{{message}}</h1>
</template>

<script>
export default {
    data() {
        return {
            message: "我是组件B"
        }
    }
}
</script>

<style scoped></style>
```

如果直接对代码进行打包的话，那么webpack会报错：需要合适的Loader来处理.vue文件。所以首先要安装vue-loader：

```js
npm install vue-loader -D
```

然后要在webpack.config.js中进行配置：

```json
{
    test: /\.vue$/,
    loader: "vue-loader"
}
```

但是安装vue-loader后依然会报错，这是因为我们必须添加@vue/compiler-sfc来对template进行解析。这是一个插件，他的作用就是解析编译.vue文件：
```js
npm install @vue/compiler-sfc -D
```

然后要在webpack.config.js中进行如下配置：

```js
const { VueLoaderPlugin } = require("vue-loader/dist/index")

module.exports = {
    // ......
    plugins: [
        new VueLoaderPlugin()
    ]
}
```

## 全局打包配置

> 我们在打包完成后运行项目时，控制台中还是会有一个警告。这个警告是在告诉我们有功能没有被明确定义，定义之后就可以在生产环境中获得更好的”tree-shaking"(Tree Shaking 指的就是当引入一个模块的时候，不引入这个模块的所有代码，而是只引入项目所需要的代码)。那么这个两个标识符代码什么意思呢？
>
> ```txt
> Feature flags __VUE_OPTIONS_API__, __VUE_PROD_DEVTOOLS__ are not explicitly defined. You are running the esm-bundler build of Vue, which expects these compile-time feature flags to be globally injected via the bundler config in order to get better tree-shaking in the production bundle.
> ```
>
> - \_\_VUE_OPTIONS_API\_\_ ： 表示是否对vue2的代码进行适配；
> - \_\_VUE_PROD_DEVTOOLS\_\_ ： 表示是否在生产环境下启用devTools；

这两个配置需要通过defindPlugin插件定义的全局配置来进行定义：

```js
plugins: [
    new DefinePlugin({
        BASE_URL: "'./'",
        __VUE_OPTIONS_API__: true,
        __VUE_PROD_DEVTOOLS__: false
    })
]
```

## vue.config.js

> 在vue脚本架中，webpack的配置文件是没有被直接暴露出来的，但是vue为我们提供了另一个修改webpack配置的方式。
>
> 那就是在项目根目录中的vue.config.js中对 webpack 进行配置，在这里写的配置会被合并到 webpack 的配置中，并且在这里配置优先级更高(会覆盖原先的配置)。

### 四种配置方式

```js
const { defineConfig } = require('@vue/cli-service')
const path = require('path')

module.exports = defineConfig({
  // 配置方式一： vuecli提供的自定义配置属性
  transpileDependencies: true,
  // 当 lintOnSave 是一个真值时，eslint-loader 在开发和生产构建下都会被启用。如果你想要在生产构建时禁用 eslint-loader，你可以用如下配置
  lintOnSave: process.env.NODE_ENV !== 'production',

  // 配置方式二： 和webpack配置属性一致，会与webpack配置合并
  configureWebpack: {
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src'),
        components: '@/components'
      }
    }
  },

  // 配置方式三： 该函数接收的参数是webpack的参数，我们可以通过这个参数重写webpack属性
  configureWebpack: (config) => {
    config.resolve.alias = {
      '@': path.resolve(__dirname, 'src'),
      components: '@/components'
    }
  },

  // 配置方式四： webpackchain，set方法返回config
  chainWebpack: (config) => {
    config.resolve.alias
      .set('@', path.resolve(__dirname, 'src'))
      .set('components', '@/components')
  }
})
```

### 配置插件

```js
const { defineConfig } = require('@vue/cli-service')
const { DefinePlugin } = require('webpack')

module.exports = defineConfig({
  transpileDependencies: true,
  chainWebpack: (config) => {
    config.plugin('html').tap((args) => {
      args[0].title = '电影网站'
      return args
    })
  },
  configureWebpack: {
    plugins: [
      new DefinePlugin({
        BASE_URL: "'./'"
      })
    ]
  }
})
```

### mock数据

```js
const { defineConfig } = require('@vue/cli-service')
const path = require('path')
const fs = require('fs')

const dirs = fs.readdirSync(path.resolve('mock'))
const mocks = dirs.map(dir => require(`./mock/${dir}`))

module.exports = defineConfig({
  transpileDependencies: true,
  // 修改已有的devServer中的web服务器，从而用来模拟web请求，用来mock一些假数据
  devServer: {
    setupMiddlewares(mids, { app }) {
      mocks.forEach(fn => fn(app))
      return mids
    }
  }
})
```

```js
module.exports = app => {
  app.get('/api/news', (req, res) => {
    res.send({
      code: 0,
      msg: 'ok',
      data: [
        { id: 1000, title: '新闻1' },
        { id: 2000, title: '新闻2' },
        { id: 3000, title: '新闻3' }
      ]
    })
  })
}
```

## .env

> vue中的几个.env文件，用来处理某些情况下不同环境需要不同的值，比如：在某个商品在测试环境跳转的域名和正式环境跳转的域名就不同，这个时候就需要用到.env文件，这几个文件都放在package.json同目录下
>
> 不同环境的切换是取决于我们运行的指令，比如`npm run build`就是生产环境，`npm run serve`就是开发环境

### .env

> 无论开发环境还是生成环境都会加载

### .env.development

> .env.development: 本地开发环境(文件名也可以是.env.dev等，没有规定死)

### .env.production

> .env.development: 本地生产环境(文件名也可以是.env.prod等，没有规定死)
