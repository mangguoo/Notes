### setupProxy.js

> 在src目录下创建一个setupProxy.js文件，这个文件必须导出一个函数，react脚手架会读取这个文件中导出的函数并执行，并给这个函数传入一个app对象(express对象)，可以通过这个文件来设置代理，或者写一些mock假数据。

```js
/src/setupProxy.js

const userMockFn = require('../mock/user')

module.exports = app => {   
  app.use('/api', (req, res) => {
      ......
  })
}
```

```js
const { createProxyMiddleware } = require('http-proxy-middleware')

module.exports = function (app) {
  app.use(
    '/api',
    createProxyMiddleware({
      target: 'http://waas.dw2nn.com',
      changeOrigin: true
    })
  )

  app.use(
    '/fund',
    createProxyMiddleware({
      target: 'http://waas.dw2nn.com',
      changeOrigin: true
    })
  )

  app.use(
    '/web-api',
    createProxyMiddleware({
      target: 'http://waas.dw2nn.com',
      changeOrigin: true
    })
  )
}
```

### config-overrides.js

> 通过react脚手架[create-react-app]创建的项目，是没有曝露webpack.config.js配置文件的，如果需要在项目中配置一些webpack配置，那么就需要借助第三方库来进行增量配置或配置改写。

**下载依赖：**

```shell
// Customize-cra利用了React-App-Rewired的config-overrides.js文件。
// 这个库中定义了一些实用的程序，可以通过它们轻松地修改ReactApp应用程序的基础配置对象
// （WebPack，WebPack-Dev-Server，babel等）。
$ npm install -D customize-cra 
// 提供调整 create-react-app webpack 配置的能力，通过config-overrides.js文件
$ npm install -D react-app-rewired
```

**改写package.json文件中的指令：**

```json
"scripts": {
   "start": "react-app-rewired start",
   "build": "react-app-rewired build",
   "test": "react-scripts test",
   "eject": "react-scripts eject"
}
```

**引用装饰器特性(babel配置)：**

```js
const { resolve } = require('path')
const { override } = require('customize-cra')

// config就是已有的webpack配置,然后返回修改后的config
const custom = () => config => {
  config.resolve.alias['@'] = resolve('./src')
  return config
}

// override用于覆盖配置
module.exports = override(custom())
```

**配置webpack别名：**

```js
const path = require('path')
const { override } = require('customize-cra')

const customConfig = () => config => {
  config.resolve.alias['@'] = path.resolve('src')
  return config
}

module.exports = override(
  customConfig()
)
```

```js
const path = require('path')
const { override, addWebpackAlias } = require('customize-cra')

module.exports = override(
  addWebpackAlias({
    '@': path.resolve('src')
  })
)
```

### jsconfig.json

> `jsconfig.json`目录中存在文件表明该目录是 JavaScript 项目的根目录。该`jsconfig.json`文件指定了JavaScript 语言服务提供的功能的根文件和选项。
>
> `jsconfig.json`是`tsconfig.json`的后代，`tsconfig.json` 是 TypeScript 的配置文件。`jsconfig.json`相当于`tsconfig.json`把`"allowJs"`属性设置为`true`。

Visual Studio Code 的 JavaScript 支持可以在两种不同的模式下运行：

- **无 jsconfig.json**：在此模式下，在 Visual Studio Code 中打开的 JavaScript 文件被视为独立单元。这种模式下只要文件没有明确的引用关系(使用ESModule或CommonJS)，两个文件之间就没有共同的项目上下文
- **使用 jsconfig.json**：JavaScript 项目是通过`jsconfig.json`文件定义的。目录中存在此类文件表明该目录是 JavaScript 项目的根目录。文件本身可以选择列出属于项目的文件、要从项目中排除的文件以及编译器选项

当您`jsconfig.json`的工作区中有一个定义项目上下文的文件时，JavaScript 体验会得到改善。因此，`jsconfig.json`当您在新工作区中打开 JavaScript 文件时，我们会提供创建文件的提示

**文件配置：**

默认情况下，JavaScript 语言服务将为 JavaScript 项目中的所有文件分析并提供 IntelliSense。您需要指定要排除或包含的文件以提供正确的 IntelliSense

- exclude属性：

  该`exclude`属性（全局模式）告诉语言服务哪些文件不属于您的源代码，用于提高性能

- include属性：

  您可以使用`include`属性（全局模式）显式设置项目中的文件。如果不存在`include`属性，则默认当前目录和子目录中的所有文件都是项目文件

  ```json
  {
    "compilerOptions": {
      "module": "commonjs",
      "target": "es6"
    },
    "exclude": ["node_modules"],
    "include": ["src/**/*"]
  }
  ```

- compilerOptions属性（它是一个对象）：

  | 选项                           | 描述                                                         |
  | ------------------------------ | ------------------------------------------------------------ |
  | `noLib`                        | 不包含默认库文件 (lib.d.ts)                                  |
  | `target`                       | 指定要使用的默认库 (lib.d.ts)。值为“es3”、“es5”、“es6”、“es2015”、“es2016”、“es2017”、“es2018”、“es2019”、“es2020”、“esnext”。 |
  | `module`                       | 生成模块代码时指定模块系统。值是“amd”、“commonJS”、“es2015”、“es6”、“esnext”、“none”、“system”、“umd”。 |
  | `moduleResolution`             | 指定如何解析模块以进行导入。值是“节点”和“经典”。             |
  | `checkJs`                      | 对 JavaScript 文件启用类型检查。                             |
  | `experimentalDecorators`       | 为提议的 ES 装饰器启用实验性支持。                           |
  | `allowSyntheticDefaultImports` | 允许从没有默认导出的模块中默认导入。这不会影响代码发出，只是类型检查。 |
  | `baseUrl`                      | 用于解析非相关模块名称的基目录。                             |
  | `paths`                        | 指定相对于 baseUrl 选项计算的路径映射。                      |

**webpack别名：**

> 为了让 IntelliSense 使用 webpack 别名，需要使用`paths`进行配置。

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### craco.config.js

> **C**reate **R**eact **A**pp **C**onfiguration **O**verride是一个简单易于理解的create-react-app的配置层。相比于customize-cra，它的配置请法与原生webpack完全相同，不需要学习新的语法。
>
> 通过在应用程序的根目录添加单个配置（例如craco.config.js）文件，从而在不弹开create-react-app配置（npm run eject）的情况下能够对其进行增量配置，并且能够自定义ESLINT，BABEL，POSTCSS等配置。

#### 基本使用：

> 使用craco给antd做按需加载：

**安装craco：**

```shell
$ yarn add -D @craco/craco babel-plugin-import
```

**修改package.json文件：**

```json
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test",
}
```

**在项目根目录下面创建一个craco.config.js文件：**

> 此文件的功能是增量配置webpack的，修改了此文件，一定要重启项目
>
> https://github.com/dilanx/craco/blob/master/packages/craco/README.md

```js
const path = require('path')
const mockFn = require('./mock')

module.exports = {
  // antd 按需加载的前提是下载 babel-plugin-import
  babel: {
    plugins: [['import', { libraryName: 'antd', libraryDirectory: 'es', style: 'css' }]]
  },
  
  webpack: {
    alias: {
      '@': path.resolve('./src')
    },
    configure: (webpackConfig, { env, paths }) => {
      // oneOf数组，当规则匹配时，只使用第一个匹配规则
      webpackConfig.module.rules[1].oneOf.unshift({
        test: /\.svg$/,
        include: [path.resolve(__dirname, 'src/assets/svg')],
        issuer: /\.[jt]sx?$/,
        use: [
          {
            loader: '@svgr/webpack',
            options: {
              prettier: false,
              svgo: false,
              svgoConfig: { plugins: [{ removeViewBox: false }] },
              titleProp: true,
              ref: true
            }
          },
          {
            loader: 'url-loader',
            options: {
              name: 'static/media/[name].[hash].[ext]'
            }
          }
        ]
      })
      return webpackConfig
    }
  },
  // 在使用中发现，在这里修改的配置不会生效，不知道是不是bug
  devServer: (devServerConfig, { env, paths, proxy, allowedHost }) => {
    devServerConfig.setupMiddlewares: (mids, { app }) => {
      mockFn(app) // mock接口
      return mids
    }
    return devServerConfig
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
  // ... ...
  "include": ["src", "craco.config.ts", "mock"]
}
```

### .env

> 在react_create_apo创建的react应用中，根目录下可以写一个.env文件，他可以用于设置全局变量。变量名必须以 REACT_APP 开头，单词大写，以 _下划线分割，.env变量名更改之后，项目必须重启才会生效。
>
> ```js
> REACT_APP_MSG = "消息"
> ```

在项目可以通过process.env.变量名的方式获取变量：

```js
process.env.REACT_APP_MSG
```

