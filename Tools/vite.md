# vite

> Vite号称是 下一代的前端开发和构建工具，它采用了全新的unbundle思想利用浏览器ESM特性导入组织代码，在服务器端进行按需编译返回，对于生产环境使用rollup打包。比起传统的webpack构建，在性能速度上都有了质的提高

## vite工作原理

> esbuild 使用go编写，cpu密集下更具性能优势，编译速度更快，相比较其他打包工具的速度提升10~100倍的差距。Vite通过esbuild带来的好处(预构建)：
>
> - **模块化兼容：** 现仍共存多种模块化标准代码，比如commonJs依赖，Vite在预构建阶段将依赖中各种其他模块化规范(CommonJS、UMD)转换 成ESM，以提供给浏览器进行加载
> - **性能优化：** npm包中大量的ESM代码，大量的import请求，会造成网络拥塞。Vite使用esbuild，将有大量内部模块的ESM关系转换成单个模块，以减少 import模块请求次数
>
> Vite预编译之后，会将文件缓存在node_modules/.vite/文件夹下。根据package.json中:dependencies是否发生变化以及包管理器的lockfile是否发生变化来重新执行预构建。如果想强制让Vite执行预构建依赖，可以使用–force启动开发服务器，或者直接删掉node_modules/.vite/文件夹

### 编译与打包

在Vite项目中的index.html中，我们可以看到如下代码，它用于请求js入口文件main.js：

```js
 <script type="module" src="/src/main.js"></script>
```

此时会向当前服务器发送一个`GET`请求用于请求main.ts

```js
import { createApp } from 'vue'
import App from './App.vue'
createApp(App).mount('#app')
```

请求到了main.js文件，检测到内部含有import引入的包，又会import 引用发起HTTP请求获取模块的内容文件，如App.vue或者其他的vue文件

Vite其核心原理是利用浏览器现在已经支持ES6的import，碰见import就会发送一个HTTP请求去加载文件，Vite启动一个 koa 服务器拦截这些请求，并在后端进行相应的处理将项目中使用的文件通过简单的分解与整合（比如将 Vue 文件拆分成 template、style、script 三个部分），然后再以ESM格式返回返回给浏览器。Vite整个过程中没有对文件进行打包编译，做到了真正的按需加载，所以其运行速度比原始的webpack开发时还需进行编译编译打包的速度相比快出许多

对于webpack传统打包构建工具，在服务器启动之前，需要从入口文件完整解析构建整个应用。因此，有大量的时间都花在了依赖生成，构建编译上

![fsda165fasd](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fsda165fasd.png)

而vite主要遵循的是使用ESM(Es modules模块)的规范来执行代码，只需要从main.js入口文件， 在遇到对应的 import 语句时，将代码执行到对应的模块再进行加载到到浏览器中，本质上实现了动态加载，那么对于灰色的部分是暂时没有用到的路由，所以这部分不会进行加载，通过这种方式我们实现了按需引入

![f1ads651fasd](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/f1ads651fasd.png)

对于webpack来讲初始化时，需要进行编译与打包，将多个文件进行打包为一个bundle.js(当然我们可以进行分包)，那么如果进行热更新的话，试想如果依赖越来越多，就算只修改一个文件，理论上热更新的速度也会越来越慢

而对于Vite来讲，是只编译不打包，当浏览器解析到浏览器解析到 import { x } from ‘./x’ 时，会发起HTTP请求去请求x，就算不用打包，也可以加载到所需要的代码，在热更新的时候只需要重新编译被修改的文件即可，对于其他未修改的文件可以从缓存中拿到编译的结果
当然两种方法对比起来，虽然Vite不需要打包，但是如果初始化时依赖过多则需要发送多个Http请求，也会带来初始化过慢的情况

### vite运行web应用

当我们运行起我们的Vite应用时，我们会发现，我们在main.ts的源码是这样的.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

 但是在请求中的main.ts发生了变化，如下：

```js
import { createApp } from "/node_modules/.vite/deps/vue.js?v=7b676d59"
import App from "/src/App.vue"

createApp(App).mount('#app')
```

> 我们请求vue模块的内容发生了变化，去请求了node_modules中.vite的缓存模块，那么明明我们的代码是这样子的，为什么在浏览器中请求的代码却不一样呢？

我们平时我们写代码，如果直接使用 import xxx from ‘xxx’，此时是通过Webpack来帮我们去寻找这个路径。但是浏览器不知道你项目里有 node_modules，它只能通过相对路径去寻找模块
我们在`localhost:3000`中去打开网页，此时我们的请求是请求的`localhost:3000`这个端口，此时会被Vite的开启的服务进行拦截，他会对直接引用 node_modules 进行路径替换，然后换成了 `/@modules/ `并返回回去。而后浏览器收到后，会发起对 `/@modules/xxx `的请求，然后被 Vite 再次拦截，并由 Vite 内部去访问真正的模块，并将得到的内容再次做同样的处理后，返回给浏览器

### vite的缓存

- `HTTP缓存`： 充分利用http缓存做优化，依赖（不会变动的代码）部分用强缓存，源码部分用304协商缓存，提升页面打开速度
- `文件系统缓存`： Vite在预构建阶段，将构建后的依赖缓存到node_modules/.vite ，相关配置更改时，或手动控制时才会重新构建，以提升预构建速度

### vite的热更新

对于Vite的热更新主要是通过WebSocket创建浏览器和服务器的通信监听文件的改变，当文件被修改时，服务端发送消息通知客户端修改相应的代码，客户端对应不同的文件进行不同操作的更新，Vite与Webpack相比较来讲，热更新还有所不同：

- `Webpack`：重新编译打包，请求打包后的文件，客户端进行重新加载
- `Vite`:请求变更后的模块，浏览器直接重新加载

## hyVite

> 可以将其配置为指令，并通过npm link链接到本地进行使用：
>
> ```json
> "bin": {
>   "hy-vite": "./bin/vite.js"
> }
> ```

- **/bin/vite.js**

```js
#!/usr/bin/env node
const Koa = require("koa");
const static = require("koa-static");
const fs = require("fs").promises;
const path = require("path");
const compilerSFC = require("@vue/compiler-sfc");
const compilerDOM = require("@vue/compiler-dom");

const app = new Koa();
app.listen(8080, () => console.log("http://127.0.0.1:8080")); // 开启koa服务器
const root = process.cwd(); // 得到当前命令行执行的工作目录路径

// 中间件，接收所有请求
app.use(async (ctx, next) => {
  let { path: urlPath, url, query } = ctx; // 请求地址
  if (urlPath.endsWith(".js")) {
    // 拦截到的请求为js文件
    let filepath = path.join(root, urlPath);
    // 设置响应头
    ctx.type = "application/javascript";
    // 读取文件内容
    let content = await fs.readFile(filepath, "utf-8");
    // 解析并替换js文件中import导入模块的相对路径
    ctx.body = resolveImports(content);
    // 直接退出，而不进入下一个中间件，因为下一个中间件是静态文件服务器
    // 在这里的逻辑已经把文件内容返回，如果再进行静态文件中间件的逻辑则会重复
    return;
  } else if (urlPath.startsWith("/@modules/")) {
    // 以modules开头说明这是模块替换结果的加载
    // 设置请求头
    ctx.type = "application/javascript";
    // 解析替换路径 /@modules/axios => axios
    let moduleName = urlPath.replace("/@modules/", "");
    // 去node_modules文件夹中查找 axios模块的package.json文件
    let modulePath = path.join(root, "node_modules", moduleName, "package.json");
    // 导入package.json文件会返回一个对象，里面包含了很多当前模块的基本信息
    // 获取模块main字段对应的值，因为main字段对应的值为作者规定的程序入口文件
    let module = require(modulePath).main;
    // 得到模块的入口文件
    let filepath = path.join(root, "node_modules", moduleName, module);
    // 读取文件内容
    let content = await fs.readFile(filepath, "utf-8");
    // 解析并替换js文件中import导入模块的相对路径
    ctx.body = resolveImports(content);
    // 直接退出，而不进入下一个中间件，因为下一个中间件是静态文件服务器
    // 在这里的逻辑已经把文件内容返回，如果再进行静态文件中间件的逻辑则会重复
    return;
  } else if (url.indexOf(".vue") > -1) {
    // sfc请求，读取vue文件，解析为js
    const filepath = path.join(root, url.split("?")[0]);
    // 得到一个ast
    const ast = compilerSFC.parse(fs.readFileSync(filepath, "utf8"));
    // 如果query中没有type，则表示是要去请求其对应的vue文件
    // 如果有type则表示去请求css或者template(根据type决定)
    if (!query.type) {
      // 从ast语法中获取
      const scriptContent = ast.descriptor.script.content;
      // 替换默认导出为一个常量，方便后续修改
      const script = scriptContent.replace("export default", "const __script = ");
      // 设置响应头
      ctx.type = "application/javascript";
      // 将对应的模块进行替换，这里加入了解析template的代码，当浏览器请求一个vue文件时，我们将其解析为ast树
      // 拆分出它的js部分，并加入一个import语句，其query中包含type，type为template，这样js文件被解析时就会
      // 再次请求一次，这次请求就会进入template逻辑，从而解析并返回dom(css部分也同理)
      ctx.body = `
        ${resolveImports(script)}
        // 解析template
        import {render as __render} from '${url}?type=template'
        __script.render = __render
        export default __script
      `;
    } else if (query.type === "template") {
      // 获取template的内容
      const tpl = ast.descriptor.template.content;
      // 编译为 render 函数
      const render = compilerDOM.compile(tpl, { mode: "module" }).code;
      ctx.type = "application/javascript";
      ctx.body = rewriteImport(render);
    } else if (query.type === "style") {
      // ... ..
    }
  }
  await next();
});

// 静态资源服务，如果文件请求在上面的逻辑中没有被处理，则会进入到下面的静态资源服务
// 比如请求public文件夹中的公共资源，或者请求根目录下的index.html模板
app.use(static(root));
app.use(static(path.join(root, "./public")));

// 解析import语句
function resolveImports(content) {
  content = content.replace(/\s+from\s+['"]([^'"]+)['"]/g, function (m0, m1) {
    if (m1.startsWith("./") || m1.startsWith("/") || m1.startsWith("../")) {
      return m0;
    } else {
      return ` from '/@modules/${m1}'`;
    }
  });
  return content;
}
```

## vite的基本使用

- **/package.json**

```json
scripts:{
 “dev”:”vite”
}
```

- **/index.html(vite构建时以该文件为入口，因此需要在这里直接通过esmodule引入js文件)**

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>vite使用</title>
</head>

<body>
  <div id="app"></div>
  <!-- 注意这里使用了esmodule -->
  <script type="module" src="./src/main.js"></script>
</body>

</html>
```

- **/src/views/main.js**

```js
//  -------------------- vite支持打包js和ts
import { hello } from './views/hello' // 后缀可写可不写
import axios from 'axios' // 导入第在方模块，vite会帮我们补全路径
import { msg } from './views/world.ts' // 支持ts

//  -------------------- vite支持打包css
import './assets/style.css'
// sass它也是支持的，只是需要安装sass编译工具  sass  npm i -D sass
import './assets/style.scss'


// new Vue({}) ==> 类，里面有很多的方法或模块，但是有时候工作中用不到，但它也打包进来了
// vue3创建vue实例的方式变化了，不是直接导入vue，而是使用一个createApp方法来创建实例，这样就可以在打包的时候进行tree-shaking
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

## 解析sfc与jsx

> 使vite能够解析单文件组件(sfc)，需要安装 `@vitejs/plugin-vue`
>
> 如果要使vite能够解析jsx，则需要安装 `@vitejs/plugin-vue-jsx`

- **/vite.config.js**

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'

export default defineConfig({
  plugins: [
    vue(), // 让vite能解析vue文件
    vueJsx() // 让vite能解析jsx文件
  ]
})
```

## Vite中编译JSX时在后缀(.vue)单文件报错解决

### 问题原因

1. `Vite`在启动时会做依赖的预构建
2. 预构建，运行时默认都只会对`jsx`与`tsx`做语法转换。不会对`.js`文件做`jsx`的`语法转换`

### 解决方法

**解决方法一：**（不推荐）

- 把`.vue后缀`文件更改为`.jsx` 或 `.tsx`文件 就可以执行

**解决方法二：**（不推荐）

- 配置@vitejs/plugin-vue-jsx，使其会解析vue文件（这个插件默认只对拓展名为.jsx/.tsx的文件进行babel解析，如果需要它解析其它扩展名文件下的jsx，可以进行如下配置）

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'

export default defineConfig({
  plugins: [
    vue(), // 让vite能解析vue文件
    vueJsx({
      include: /\.[jt]sx|vue/
    }) // 让vite能解析jsx文件
  ]
})
```

**解决方法三:** （推荐！）

- `.vue`文件的script标签加上`lang='jsx' 或 lang='tsx'`就可以解析vue文件。但是首先要确保你已经安装了**解析jsx的plugin**: `@vitejs/plugin-vue-jsx`

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from "@vitejs/plugin-vue-jsx"
export default defineConfig({
  plugins: [
    vue(),
    vueJsx()
  ]
})
```

```html
<script lang="jsx">
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
        <h2>script标签要加上lang='jsx'才能解析</h2>
        <h2>当前计数: {this.counter}</h2>
        <button onClick={increment}>+1</button>
        <button onClick={decrement}>-1</button>
      </div>
    );
  },
};
</script>
```

## vite基础配置

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import path from 'path'

export default defineConfig({
  plugins: [vue(), vueJsx({
    // 默认只对扩展名为 .jsx/.tsx 进行babel解析，需要它解析.vue扩展名下面的jsx，可以进行如下配置
    include: /\.[jt]sx|vue/
  })],
  resolve: {
    alias: { // 添加别名
      '@': path.resolve('src')
    }
  },
  server: {
    port: 8080, // 配置端口
    open: false, // 自动打开浏览器
    // 通过配置开发时，代理服务器，在开发时进行跨域解决
    proxy: {
      '/api': {
        target: 'https://api.iynn.cn/film',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})
```

如果配置了别名，想要让vscode能够识别这个别名，还需要在jsconfig.json中进行一些配置：

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "exclude": ["node_modules", "dist"]
}
```

## 情景配置

> 如果配置文件需要基于（`dev`/`serve` 或 `build`）命令或者不同的模式（`development`，`production`）来决定选项，亦或者是一个 SSR 构建（`ssrBuild`），则可以给defineConfig传入一个函数：

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import path from 'path'
import { viteMockServe } from 'vite-plugin-mock'

export default defineConfig((command, mode, ssrBuild) => {
  let config = {
    plugins: [vue(), vueJsx(), viteMockServe({})],
    resolve: {
      alias: {
        '@': path.resolve('src')
      }
    }
  }

  // 开发环境中添加网络代理
  if ('development' === mode) {
    // 开发环境
    config = {
      ...config,
      server: {
        port: 8080,
        open: false,
        proxy: {
          '/api': {
            target: 'https://api.iynn.cn/film',
            changeOrigin: true
          }
        }
      }
    }
  }
  return config
})
```

需要注意的是，在 Vite 的 API 中，在开发环境下 `command` 的值为 `serve`（在 CLI 中， `vite dev` 和 `vite serve` 是 `vite` 的别名），而在生产环境下为 `build`（`vite build`）

`ssrBuild` 仍是实验性的。它只在构建过程中可用，而不是一个更通用的 `ssr` 标志，因为在开发过程中，我们唯一的服务器会共享处理 SSR 和非 SSR 请求的配置。某些工具可能没有区分浏览器和 SSR 两种构建目标的命令，那么这个值可能是 `undefined`，因此需要采用显式的比较表达式

## mock数据

- 首先要安装`vite-plugin-mock`和`mockjs`

- 配置`vite.config.js`

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import path from 'path'
import { viteMockServe } from 'vite-plugin-mock'
  
export default defineConfig({
  plugins: [
    viteMockServe({})
  ]
})
```

- 在根目录创建mock文件夹，在mock文件夹中创建index.js，然后以固定格式书写接口：

```js
import Mockjs from 'mockjs'

const mockData = [
  {
    url: '/api/films',
    method: 'get',
    response: ({ query }) => {
      const data = Mockjs.mock({
        'films|10': [
          {
            "filmId|+1": 1,
            'name': '@cname'
          }
        ]
      })
      return {
        code: 0,
        msg: 'ok',
        data
      }
    }
  },
  {
    url: '/api/login',
    method: 'post',
    response: () => ({
      code: 0,
      msg: 'ok',
      data: {
        uid: 1000,
        nickname: '张三',
        token: 'afewlfjewlfjewlfejlfejl;fejlf;e'
      }
    })
  }
]

export default mockData
```

## import.meta.glob

- **为true表示直接导入（同步导入）**

```js
import { createStore, useStore } from 'vuex'

const moduleFiles = import.meta.glob('./modules/*', { eager: true })
let modules = {}
for (let key in moduleFiles) {
  let prop = /\.\/modules\/(\w+)\.js/.exec(key)[1]
  let value = moduleFiles[key].default
  value['namespaced'] = true // 开启命名空间
  modules[prop] = value
}
const store = createStore({ modules })

export default store
```

- **为false表示动态导入，在构建时会分离为独立的chunk**

```js
import { createStore, useStore } from 'vuex'

const moduleFiles = import.meta.glob('./modules/*')
let modules = {}
for (let key in moduleFiles) {
  moduleFiles[key]().then(res => {
    let prop = /\.\/modules\/(\w+)\.js/.exec(key)[1]
    let value = res.default
    value['namespaced'] = true // 开启命名空间
    modules[prop] = value
  })
}
const store = createStore({ modules })

export default store
```

