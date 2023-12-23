# webpack

> ==webpack is a static module bundler for modern JavaScript applications==
>
> - 打包bundler：webpack可以将帮助我们进行打包，所以它是一个打包工具；
> - 静态的static：这样表述的原因是我们最终可以将代码打包成最终的静态资源（部署到静态服务器）；
> - 模块化module：webpack默认支持各种模块化开发，ES Module、CommonJS、AMD等；
> - 现代的modern：我们前端说过，正是因为现代前端开发面临各种各样的问题，才催生了webpack的出现和发展；
>
> webpack到底是如何对我们的项目进行打包的呢？
>
> - 事实上webpack在处理应用程序时，它会根据命令或者配置文件找到入口文件；
> - 从入口开始，会生成一个依赖关系图，这个依赖关系图会包含应用程序中所需的所有模块（比如.js文件、css文件、图片、字体等）；
> - 然后遍历图结构，打包一个个模块（根据文件的不同使用不同的loader来解析）；

## 安装webpack

### 全局安装

> webpack的安装目前分为两个：webpack、webpack-cli。全局安装这两个包后就可以在用命令行在任何文件位置使用webpack进行打包操作。

**全局安装方法：**

```shell
$ npm install webpack webpack-cli –g // 全局安装
```

**webpack、webpack-cli之间的关系：**

- 执行webpack命令，会执行全局环境中的webpack;

  `User\AppData\Roaming\npm\node_modules\webpack\bin`

- webpack在执行时是依赖webpack-cli的，如果没有安装就会报错；
- webpack-cli用于在命令行中运行webpack，cli即命令行接口（Command Line Interface）。在webpack-cli中代码执行时，才是真正利用webpack进行编译和打包的过程；
- 所以在安装webpack时，我们需要同时安装webpack-cli（第三方的脚手架事实上是没有使用webpack-cli的，而是类似于自己的vue-service-cli的东西）；

### 局部安装

- 创建package.json文件，用于管理项目的信息、库依赖等

```js
npm init
```

- 安装局部的webpack

``` js 
npm install webpack webpack-cli --save-dev
```

如何使用局部的webpack?

- 通过npx命令来使用局部webpack：

```js 
npx webpack
```

- 在package.json中创建scripts脚本：

```js
"scripts": {
    "build": "webpack"
}
// 运行打包命令
npm run build
```

## webpack打包方式

### 默认打包

直接在项目根目录下执行 webpack 命令，webpack就会自动生成一个dist文件夹，里面存放着一个main.js的文件，这就是项目打包之后的文件。这个文件中的代码会被压缩和丑化，另外代码中依然存在ES6的语法，比如箭头函数、const等，这是因为默认情况下webpack并不清楚我们打包后的文件是否需要转成ES5之前的语法，需要我们通过babel来进行转换和设置。

**webpack是如何确定项目入口文件的呢？**

- 当运行webpack时，webpack会查找当前目录下的 src/index.js作为入口；
- 所以，如果当前项目中没有存在src/index.js文件，那么会报错；

### 指定出入口

**直接通过webpack命令的参数来指定：**

```shell
$ npx webpack --entry ./src/main.js --output-path ./build
```

**通过webpack的配置文件来指定：**

在webpack.config.js配置文件中进行配置，webpack会读取这个文件中的配置进行打包。

## webpack配置文件

> 在通常情况下，webpack需要打包的项目是非常复杂的，并且需要一系列的配置来满足要求，默认配置必然是不可以的。
>
> 我们可以在根目录下创建一个webpack.config.js文件，来作为webpack的配置文件。webpack会默认读取这个名称的文件作为他的配置文件。
>
> 但是如果配置文件的名称并不是webpack.config.js，而是叫wk.config.js，那webpack就读取不到了，需要在运行webpack时手动指定配置文件（`webpack --config wk.config.js`）。

```js
const path = require('path');

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "./build"),
    filename: "bundle.js"
  }
}
```

### 常见配置

**target：**

> Target用于指定Webpack构建的环境，也就是针对不同的环境进行不同的构建：

- web：针对浏览器环境；
- node：针对node.js环境

**mode：**

> 打包模式配置

- production是生产模式，会把打包好后的代码进行压缩，可阅读性不好，但是代码体积小；
- development是开发模式，不会压缩代码，可阅读性好，但是代码体积大；
- none表示不使用任何默认优化选项；

在开发过程中，我们一般会选用开发模式进行打包。因为生产模式会把我们的代码压缩成一行，并且会对代码进行一些混淆操作。这样如果程序中Bug，在控制台中报的错误栈全部都是在bundle.js(打包后的js文件)中，这样排错就会变得非常困难。

而使用开发模式进行打包的话，我们就可以在控制台的错误栈信息中看到是源代码中的哪一个文件中的哪一段代码在报错，这样就大大方便了排错。

**devtool：**

> 开发者工具

- source-map ：产生一个单独的source-map文件，功能最完全，但会减慢打包速度；
- eval-source-map ：使用eval打包源文件模块，直接在源文件中写入干净完整的source-map，不影响构建速度，但影响执行速度和安全，建议开发环境中使用，生产阶段不要使用；
- cheap-module-source-map：会产生一个不带映射到列的单独的map文件，开发者工具就只能看到行，但无法对应到具体的列（符号），对调试不便；
- eval-cheap-module-source-map：不会产生单独的map文件，（与eval-source-map类似）但开发者工具就只能看到行，但无法对应到具体的列（符号），对调试不便；

虽然用开发模式打包项目已经可以让我们在控制台中看到对应的错误栈信息，但是不是特别清楚。这里我们使用dev-tool: source-map的话，就可以给打包出来的文件单独生成一个映射文件。这个文件中保存着打包后代码片段与源代码之间的映射。

```js
module.exports = {
  mode: "development",
  devtool: "source-map",
}
```

**entry：**

> 配置入口文件：该属性对应的是一个字符串

**output：**

> 配置输出文件，该属性对应的是一个对象，包含以下属性：
>
> - path：输出文件存储路径
> - filename：输出文件的名称

**rules：**

> 配置模块打包规则

module.rules的配置如下：

rules属性对应的值是一个数组：[Rule]，数组中存放的是一个个的Rule，Rule是一个对象，对象中可以设置多个属性：

- test属性：用于对 resource（资源）进行匹配的，通常会设置成正则表达式
- use属性：对应的值时一个数组：[UseEntry]
  - UseEntry是一个对象，可以通过对象的属性来设置一些其他属性
    - loader：必须有一个 loader属性，对应的值是一个字符串
    - options：可选的属性，值是一个字符串或者对象，值会被传入到loader中
    - query：目前已经使用options来替代
  - 传递字符串（如：use: [ 'style-loader' ]）是 loader 属性的简写方式（如：use: [ { loader: 'style-loader'} ]）
- loader属性： Rule.use: [ { loader } ] 的简写

**plugins：**

> 配置要使用的插件，该属性对应的是一个数组，里面包含着一个个插件对象。

**watch：**

> 观察者模式，在开发环境中，如果想要实时在网页中看到代码的更改，又不想要每次修改代码都手动执行npm run build，就可以开启观察者模式。它会观察文件的变化，每当文件发生变化时，就会自动打包项目。
> watch是一个布尔值，true为开启，false为关闭。

**devServer：**

> 配置webpack开发服务器

**resolve：**

> 模块解析，resolve用于设置模块如何被解析，在项目开发中会依赖各种各样的模块，这些模块可能来自于自己编写的代码，也可能来自第三方库。resolve可以帮助webpack从每个 require/import 语句中找到需要引入的模块。

```js
resolve: {
    extensions: [".js", ".json", ".mjs", ".vue", ".ts", ".jsx", ".tsx"],
    mainFiles: ["index"],
    modules: ["node_modules"],
    alias: {
        "@": path.resolve(__dirname, "./src"),
        "js": path.resolve(__dirname, "./src/js")
    }
}
```

- modules：

  webpack能解析三种文件路径，有绝对路径、相对路径和模块路径(import vue from "vue")。webpack之所以能够解析模块路径，就是因为它会在resolve.modules中指定的所有目录检索模块。而resolve.modules的默认值为 ['node_modules']，所以默认会从node_modules中查找模块。

- extensions：解析到文件时自动添加扩展名

  - 默认值是 ['.wasm', '.mjs', '.js', '.json']；
  - 也可以手动添加.vue、jsx、ts 等文件；

- mainFiles：如果导入模块的路径写的是一个文件夹，那么webpackp会在文件夹中根据 resolve.mainFiles配置选项中指定的文件顺序查找：

  - resolve.mainFiles的默认值是 ['index']；
  - 找到相应的文件后，再根据 resolve.extensions来解析扩展名

- alias：当一个项目的目录结构比较深的时候，或者一个文件的路径可能需要 ../../../这种路径片段时，就可以给某些常见的路径起一个别名。比如：

  ```js
  alias: {
      “js”: path.resolve(__dirname, "./src/js")
  }
  
  import xx from "js/math"
           < 等价于 >
  import xx from "./src/js/math.js"
  ```

### PlaceHolders

> 有时候我们通过webpack打包处理文件后，想让文件名称按照一定的规则进行显示，比如保留原来的文件名、扩展名，同时为了防止重复，包含一个hash值等。这个时候我们可以使用PlaceHolders来完成，webpack给我们提供了大量的PlaceHolders来显示不同的内容：

- [ext]： 处理文件的扩展名；
- [name]：处理文件的名称；
- [hash]：文件的内容，使用MD4的散列函数处理，生成的一个128位的hash值（32个十六进制）；
- [contentHash]：在file-loader中和[hash]结果是一致的（在webpack的一些其他地方不一样，后面会讲到）；
- [hash:<length>]：截图hash的长度，默认32个字符太长了；
- [path]：文件相对于webpack配置文件的路径；

## devServer

> 目前我们开发的代码，为了运行需要有两个操作。首先要运行npm run build编译相关的代码，然后通过live server打开index.html代码查看效果。
> 但是这个过程会影响开发效率，我们可以通过webpack自带的express服务器实现当文件发生变化时，可以自动的完成编译和展示的功能。

### webpack watch

> webpack给我们提供了watch模式。在该模式下，webpack依赖图中的所有文件，只要有一个发生了更新，那么代码将被重新编译，不再需要手动去运行npm run build指令了。
>
> 开启watch的两种方式：
>
> - 在导出的配置中，添加 watch: true；
> - 在启动webpack的命令中，添加 --watch的标识；

### webpack dev server

**devServer的功能：**

> webpack watch可以监听到文件的变化，但是它本身是没有开启本地http服务器的功能的。
> 虽然可以在VSCode中使用live-server来完成这样的功能，但是我们希望在不使用live-server的情况下，可以给打包后的项目打开本地服务器，并且实现浏览器live reloading（实时重新加载）的功能。
>
> 注意：webpack-dev-server 在编译之后不会写入到任何输出文件，他会将打包后的文件保留在内存中。webpack-dev-server使用了一个库叫memfs（内存文件系统），专门用于在内存中保存打包后的项目。

**devServer的使用方法：**

- 安装webpack-dev-server

  ```js
  npm install webpack-dev-server -D
  ```

- 修改package.json

  ```js
  "scripts": {
     "build": "webpack",
     "serve": "webpack serve"
  }
  ```

- 运行webpack-dev-server

  ```js
  npm run serve
  ```

**devServer的配置：**

- hot

  > 模块热替换（HMR）。HMR的全称是Hot Module Replacement，翻译为模块热替换。模块热替换是指在应用程序运行过程中，替换、添加、删除模块，而无需重新刷新整个页面。
  >
  > 比如说在某个项目中，A模块引用了B、C两个模块，当C模块发生更新时，devServer会重新打包项目，并使浏览器刷新页面，这会导致没有发生改变的模块也重新执行，而HMR可以实现只重新执行更新了的模块。
  >
  > HMR还有一个好处，就是会保留某些应用程序的状态不丢失。比如模块B中写了一个计数器，在运行过程中，这个计数器的数值发生了改变，而这时我更改了模块C中的程序，这就会导致所有模块中的程序都会被重新执行，从而造成状态的丢失。而HMR只更新需要变化内容的特性就会避免这种情况。

  webpack-dev-server会创建两个服务：提供静态资源的服务（express）和Socket服务（net.Socket）：

  express server负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）。这种http服务器是由客户端发起请求，然后由服务端向客户端发送请求资源的，在请求完成后http连接就会被关闭。下次服务端再有文件更新时，就得再次向服务端请求静态资源，请求完成后由浏览器重新解析加载页面。

  HMR Socket Server，会建立一个socket的长连接。长连接有一个最好的好处是建立连接后双方可以通信（服务器可以直接发送文件到客户端），当服务器监听到对应的模块发生变化时，会生成两个文件.json（manifest文件）和.js文件（update chunk），通过长连接，可以直接将这两个文件主动发送给客户端（浏览器），浏览器拿到两个新的文件后，通过HMR runtime机制，加载这两个文件，并且针对修改的模块进行更新。

  ![7d517c64ab6a7a9538673a973f2c6c18b5b202c9208340f7a53825cecf16203c](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/7d517c64ab6a7a9538673a973f2c6c18b5b202c9208340f7a53825cecf16203c.png)

  默认情况下，webpack-dev-server已经支持HMR，只需要开启即可：

  ```js
  // target属性表示打包后的项目是要在什么环境下运行的，web表示在前端环境(浏览器)下运行。不加这个属性使用热替换的话可能会有一些问题。
  module.exports = {
    target: "web",
    devServer: {
      static: "./public",
      hot: true,
    }
  }
  ```

  虽然已经开启了HMR，但是当我们修改了某一个模块的代码时，依然会刷新整个页面。这是因为我们需要去指定哪些模块发生更新时需要进行HMR：

  ```js
  =>  src/index.js
  const { priceFormat } = require("./js/format")
  if(module.hot) {
      module.hot.accept("./js/format.js", () => {
          console.log("模块A更新了")
      })
  }
  
  =>  src/js/format.js
  const priceFormat = function() {
    return "¥99.888";
  }
  module.exports = {
    priceFormat
  }
  ```

  现在如果修改format.js的话，就会触发热替换。并且指定的回调函数会被执行。

  在开发项目时，如果想要实现热替换，就需要给每个模块写module.hot.accpet方法，这样显然会让开发变得非常麻烦。

  而在开发Vue、React项目时，对于热替换，这两个框架已经有了很成熟的解决方案。比如vue开发中，一般会使用vue-loader打包vue文件，而此loader支持vue组件的HMR，提供开箱即用的体验。

- host

  > 该属性表示devServer监听的本机地址，默认值是localhost，表示只有本机可以访问。如果希望其他地方也可以访问，可以设置为 0.0.0.0，当前也可以设为本机的真实地址，如192.168.10.1。

  - localhost：本质上是一个域名，通常情况下会被解析成127.0.0.1，127.0.0.1表示回环地址，向回环地址发出去的包，会直接被自己接收。正常的数据包经过 应用层 - 传输层 - 网络层 - 数据链路层 - 物理层，而回环地址，是在网络层直接就被获取到了，是不会经过数据链路层和物理层的。当监听 127.0.0.1时，在同一个网段下的主机中通过ip地址是不能访问该服务器的。
  - IPV4中，0.0.0.0地址被用于表示一个无效的，未知的或者不可用的目标。
    - 在服务器中，0.0.0.0指的是本机上的所有IPV4地址，如果一个主机有两个IP地址，192.168.1.1 和 10.1.2.1，并且该主机上的一个服务监听的地址是0.0.0.0,那么通过两个ip地址都能够访问该服务。
    - 在路由中，0.0.0.0表示的是默认路由，即当路由表中没有找到完全匹配的路由的时候所对应的路由。

- port

  > 表示监听的端口，默认情况下是8080

- open

  > 表示是否默认打开浏览器，它是一个布尔值，也可以设置为类似于Google Chrome等值。

- compress

  > 该选项表示是否为静态文件开启gzip compression。默认值是false，可以设置为true。当开启时，webpack会对打包后的静态文件进行gzip压缩。

- proxy

  > proxy是开发中常用的配置选项，它的目的设置代理来解决跨域访问的问题。比如我们的一个api接口地址是 http://localhost:8888，但是本地启动devServer的域名是 http://localhost:5050，这个时候发送网络请求就会出现跨域的问题。那么我们可以将请求先发送到一个代理服务器，代理服务器和API接口服务器不会产生跨域的问题，就可以解决我们的跨域问题了。
  >
  > 代理服务器和API接口服务器不会产生跨域的问题是由于javascript等脚本的主动http请求才会出现跨域问题，后端获取http数据不会存在跨域问题。
  >
  > 在生产环境中，可以通过Nginx的反向代理来解决跨域的问题，而在开发环境中，为了方便可以使用devServer的Proxy来解决这个问题。由于devServer会开启一个express服务器，我们可以让这个express服务器代理我们去访问会产生跨域冲突的接口，这样跨域的问题就解决了。
  >
  > 什么是跨域？跨域就是指跨域名访问，以下情况都属于跨域：
  >
  > - 域名不同：www.jd.com与www.baidu.com；
  > - 域名相同，端口不同：www.jd.com:8080与www.jd.com:8081；
  > - 二级域名不同：item.jd.com与www.jd.com

  ```js
  devServer: {
      contentBase: "./public",
      hot: true,
      host: "0.0.0.0",
      port: 5050,
      open: true,
      compress: true,
      proxy: {
          "/api": {
              // 表示的是代理到的目标地址，比如 /api/moment会被代理到 http://localhost:8888/api/moment
              target: "https://localhost:8888",
              // 默认情况下，我们的 /api 也会被写入到URL中，如果希望删除，可以使用pathRewrite
              pathRewrite: {
                  "^/api": ""
              },
              // 默认情况下不接受在HTTPS上运行且证书无效的后端服务器(使用https协议的话，必须携带证书，如果没有证书的话，会导致无法访问)，如果想要使用HTTPS协议访问，但是又没有证书的话，可以把该选项设置为false
              secure: false,
              // 它表示是否把代理后请求的headers中host地址变成target中的地址。比如有的服务器会通过请求的header中的host地址来判定是否是通过代理来访问的，也就是判断一下请求头中的host地址是不是本机地址，如果不是，那么就禁 止访问。为了避免这个问题，我们可以通过changeOrigin选项来更改代理后请求的headers中的host地址
              changeOrigin: true
          }
      }
  }
  ```

  配置代理后，再访问当前域名下的“/api/moment”接口，就会先被发送给当前的devServer：
  http://localhost:5050，devServer收到请求后，检索代理规则时发现“/api”被代理了，然后devServer就会把“/api”代理到相应的target：https://localhost:8888

  ```js
  axios.get("/api/moment").then(res => {
    console.log(res);
  }).catch(err => {
    console.log(err);
  })
  ```

- static

  > 指定devServer从什么位置查找文件。类型于express的静态文件服务器。在以前的项目中，index.html文件会引用一个叫favicon.ico的图片作为页面图标。但是这个图片并不会被webpack所打包，因为webpack以src/index.js作为打包的入口文件，而favicon.ico并不在项目的依赖图中，所以不会被打包。
  >
  > 想要在打包后的项目后正确显示这个图标文件，解决方法就是使用CopyWebpackPlugin把这个图标文件复制到打包后的目标文件中。但是在开发环境中，每次打包都要复制一次，如果要复制的文件很大，那么势必会影响打包效率。
  >
  > 要解决上面的问题就可以用到devServer的static属性，它表示如果在打包后的项目中遇到了找不到的文件，就在contentBase所指定的目录下寻找。这样webpack在打包过程中就省去了复制文件的步骤。
  >
  > 该配置项允许配置从目录提供静态文件的选项（默认是 'public' 文件夹）。
  >
  > - publicPath：告诉服务器在哪个 URL 上提供 [`static.directory`](https://webpack.docschina.org/configuration/dev-server/#directory) 的内容
  > - directory：告诉服务器从哪里提供内容

  ```js
  module.exports = {
      devServer: {
          static: "./public"
      }
  }
  ```

  ```js
  const path = require('path');
  
  module.exports = {
    //...
    devServer: {
      static: [
        {
          directory: path.join(__dirname, 'assets'),
          publicPath: '/serve-public-path-url',
        },
        {
          directory: path.join(__dirname, 'css'),
          publicPath: '/other-serve-public-path-url',
        },
      ],
    },
  };
  ```

- setupMiddlewares

  ```js
  const path = require('path')
  const fs = require('fs')
  
  const dirs = fs.readdirSync(path.resolve('mock'))
  const mocks = dirs.map(dir => require(`./mock/${dir}`))
  
  module.exports = {
    // ...
    devServer: {
      setupMiddlewares: (middlewares, devServer) => {
        if (!devServer) {
          throw new Error('webpack-dev-server is not defined');
        }
  
        // 发起网络请求
        devServer.app.get('/setup-middleware/some/path', (_, response) => {
          response.send('setup-middlewares option GET');
        });
          
        // mock假数据
        setupMiddlewares(mids, server) {
          mocks.forEach(fn => fn(server.app))
          return mids
        }
        
        // 如果你想在所有其他中间件之前运行一个中间件可以使用 `unshift` 方法
        middlewares.unshift({
          name: 'first-in-array',
          path: '/foo/path', // `path` 是可选的
          middleware: (req, res) => {
            res.send('Foo!');
          },
        });
  
        // 如果你想在所有其他中间件之后运行一个中间件可以使用 `push` 方法
        middlewares.push({
          name: 'hello-world-test-one',
          path: '/foo/bar', // `path` 是可选的
          middleware: (req, res) => {
            res.send('Foo Bar!');
          },
        });
        middlewares.push((req, res) => {
          res.send('Hello World!');
        });
  
        return middlewares;
      },
    },
  };
  ```

  ```js
  // mock假数据
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

## 区分环境对webpack进行配置

> 目前我们所有的webpack配置信息都是放到一个配置文件中的：webpack.config.js，当配置越来越多时，这个文件会变得越来越不容易维护，并且某些配置是在开发环境需要使用的，某些配置是在生成环境需要使用的，当然某些配置是在开发和生成环境都会使用的，所以最好对配置进行划分，方便维护和管理。
>
> 配置文件的划分方案：
>
> - webpacl.dev.config.js： 用于保存开发环境独有的配置；
> - webpacl.prod.config.js： 用于生产开发环境独有的配置；
> - webpacl.comm.config.js： 用于保存开发和生产环境都有的配置；
>
> 把配置文件分成了三份，生产环境中使用prod和comm，开发环境中使用dev和comm，但是打包时不能同时指定两个配置文件。要解决这个问题就可以用一个webpack配套的模块，把prod和comm、dev和comm合并起来：
>
> - 安装webpack-merge： npm install webpack-merge -D；
> - 使用webpack-merge模块中的merge方法把配置文件合并起来；

### webpack.prod.config.js

```js
const {CleanWebpackPlugin} = require("clean-webpack-plugin")
const CopyWebpackPlugin = require("copy-webpack-plugin")
const path = require("path")

const {merge} = require("webpack-merge")
const commonConfig = require("./webpack.comm.config")

// 合并commonConfig文件
module.exports = merge(commonConfig, {
    mode: "production", // 开启生产环境
    plugins: [
        // 生产环境中每次打包时清除一次打包的内容，开发环境则不需要，因为开发环境使用devServer，它是把打包内容存储在内存中
        new CleanWebpackPlugin(),
        // 生产环境中，要把静态文件存放进相应的目录中，不能像生产环境一样在devServer中配置static静态目录，因为生产环境中没有devServer
        // 模板html文件要通过HtmlWebpackPlugin进行处理后再进行打包，所以要把它忽略掉
        new CopyWebpackPlugin({
            patterns: [{
                from: path.resolve(__dirname, "../public"),
                to: "./",
                
                globOptions: {
                    ignore: [ 
                        '**/index.html'
                    ]
                }
            }]
        })
    ]
})
```

### webpack.dev.config.js

```js
const path = require("path")

const {merge} = require("webpack-merge")
const commonConfig = require("./webpack.comm.config")

// 合并commonConfig文件
module.exports = merge(commonConfig, {
    mode: "development", // 开启开发环境
    devtool: "source-map", // 生成映射文件
    devServer: {
        static: path.resolve(__dirname, "../public"), // 静态文件目录
        hot: true, // HMR
        host: "0.0.0.0", // 主机地址
        port: 5050, // 监听端口
        open: true, // 开启后自动打开浏览器
        compress: true, // 对打包文件进行压缩
        proxy: { // 开启代理
            "/api": {
                target: "http://localhost:8888",
                pathRewrite: {
                    "^/api": ""
                },
                secure: false,
                changeOrigin: true
            }
        }
    },
})
```

### webpack.comm.config.js

```js
const path = require("path")
const HtmlWebpackPlugin = require("html-webpack-plugin")
const {DefinePlugin} = require("webpack")
const {VueLoaderPlugin} = require("vue-loader/dist/index")

module.exports = {
    target: "web", // 目录为浏览器环境
    entry: path.resolve(__dirname, "../src/index.js"), // 入口
    output: { // 出口
        path: path.resolve(__dirname, "../build"),
        filename: "js/bundle.js",
    },
    resolve: { // 解析配置
        extensions: [".js", ".json", ".mjs", ".vue", ".ts", ".jsx", ".tsx"],
        alias: {
            "@": path.resolve(__dirname, "../src"),
            "js": path.resolve(__dirname, "../src/js")
        }
    },
    module: { // 对于打包不同文件的规则
        rules: [。。。。。。]
    },
    plugins: [
        new HtmlWebpackPlugin({ // 配置模板html文件
            title: "webpack项目",
            template: path.resolve(__dirname, "../public/index.html")
        }),
        new DefinePlugin({ // 配置变量
            BASE_URL: "'./'",
            __VUE_OPTIONS_API__: true,
            __VUE_PROD_DEVTOOLS__: false
        }),
        new VueLoaderPlugin()
    ]
}
```

### package.json

```json
"scripts": {
  "build": "webpack --config ./config/webpack.prod.config.js",
  "start": "webpack serve --config ./config/webpack.dev.config.js"
},
```

## browserlist

> browserlist用于在不同前端工具之间共用目标浏览器和node版本的配置工具。简单来说，就是用来控制浏览器版本的一个插件。它可以干什么?
>
> - 搭配babel转义代码，将老浏览器不支持的新API转义为可运行代码（polyfill）；
> - 搭配Autoprefixer编译CSS代码，自动添加浏览器前缀；

### 在哪里配置？

**在package.json中加入browserslist 配置项：**

```json
"browserslist": [
    "last 1 version",
    "> 1%",
    "maintained node versions",
    "not dead"
]
```

**在项目根目录或父目录下的.browserslistrc配置文件：**

```txt
# Browsers that we support

last 1 version
> 1%
maintained node versions
not dead
```

### 查询条件：

- \> 0.5% or last 2 versions：取并集。
- \> 0.5% , last 2 versions：取并集。
- \> 0.5% and last 2 versions：取交集。
- \> 0.5% not last 2 versions：取> 0.5%的差集。
- not last 2 versions：范围取反。
- \> 5%: 基于全球使用率统计而选择的浏览器版本范围。>=,<,<=同样适用。
- \> 5% in US : 同上，只是使用地区变为美国。支持两个字母的国家码来指定地区。
- \> 5% in alt-AS : 同上，只是使用地区变为亚洲所有国家。这里列举了所有的地区码。
- \> 5% in my stats : 使用定制的浏览器统计数据。
- cover 99.5% : 使用率总和为99.5%的浏览器版本，前提是浏览器提供了使用覆盖率。
- cover 99.5% in US : 同上，只是限制了地域，支持两个字母的国家码。
- cover 99.5% in my stats :使用定制的浏览器统计数据。
- maintained node versions :所有还被 node 基金会维护的 node 版本。
- node 10 and node 10.4 : 最新的 node 10.x.x 或者10.4.x 版本。
- current node :当前被 browserslist 使用的 node 版本。
- extends browserslist-config-mycompany :来自browserslist-config-mycompany包的查询设置。
- ie 6-8 : 选择一个浏览器的版本范围。
- Firefox > 20 : 版本高于20的所有火狐浏览器版本。>=,<,<=同样适用。
- ios 7 :ios 7自带的浏览器。
- Firefox ESR :最新的火狐 ESR（长期支持版） 版本的浏览器。
- unreleased versions or unreleased Chrome versions : alpha 和 beta 版本。
- last 2 major versions or last 2 ios major versions :最近的两个发行版，包括所有的次版本号和补丁版本号变更的浏览器版本。
- since 2015 or last 2 years :自某个时间以来更新的版本（也可以写的更具体since 2015-03或者since 2015-03-10）。
- dead :通过last 2 versions筛选的浏览器版本中，全球使用率低于0.5%并且官方声明不在维护或者事实上已经两年没有再更新的版本。目前符合条件的有 IE10,IE_Mob 10,BlackBerry 10,BlackBerry 7,OperaMobile 12.1。
- last 2 versions :每个浏览器最近的两个版本。
- last 2 Chrome versions :chrome 浏览器最近的两个版本。
- defaults :默认配置> 0.5%, last 2 versions, Firefox ESR, not dead。
- not ie <= 8 : 浏览器范围的取反。

## tree-shaking

> 在前端的性能优化中，es6 推出了tree shaking机制，tree shaking就是当我们在项目中引入其他模块时，他会自动将我们用不到的代码，或者永远不会执行的代码摇掉，在Uglify阶段查出，不打包到bundle中。
>
> 只支持ES6 Module代码。并且在production 环境默认开启。如果想要做到tree shaking，在引入模块时就应该避免将全部引入，应该引入局部才可以触发tree shaking机制。

```js
// Import everything (not tree-shaking)
import lodash from 'lodash';

// Import named export (can be tree-shaking)
import { debounce } from 'lodash';

// Import the item directly (can be tree-shaking)
import debounce from 'lodash/lib/debounce';
```

### 开启tree-shaking:

**开发环境配置tree shaking：**

```js
module.exports = {
  // ...
  mode: 'development',
  optimization: {
    usedExports: true,
  }
};
```

**生产环境下的配置：**

> 生产环境下只需要把mode配置成‘production’即可

```js
module.exports = {
  // ...
  mode: 'production',
};
```

### sideEffects

> 可以package.json 中，添加字段：sideEffects告诉 Webpack 哪些代码可以处理。配置sideEffects的原因是在进行树摇时可能会对导致一些文件的错误，比如css文件。
>
> 对于那些直接引入到 js 文件的全局 css，它们并不会被转换成一个 CSS 模块。原因在于将 sideEffects 设置为 false后，所有的文件都会被 Tree Shaking，通过 import 这样的形式引入的 CSS 就会被当作无用代码处理掉。
>
> 因为它没有导出任何东西，并且它也没有被命名导入(import "./styles/reset.css")，webpack会认为它是无用的。
>
> 除了在package.json中进行配置，可以在 loader 的规则配置中，添加 sideEffects: true，告诉 Webpack 这些文件不要Tree Shaking。

```js
// All files have side effects, and none can be tree-shaken
{
 "sideEffects": true
}

// No files have side effects, all can be tree-shaken
{
 "sideEffects": false
}

// Only these files have side effects, all other files can be tree-shaken, but these must be kept
{
 "sideEffects": [
  "./src/file1.js",
  "./src/file2.js"
 ]
}
```

## require.conntext

> 它是一个 webpack 的 api ，通过该函数可以获取一个上下文，从而实现工程自动化（遍历文件夹的文件，从中获取指定文件，自动导入模块）。在前端工程中，如果一个文件夹中的模块需要频繁引用时可以使用该中方式一次性引入。它有三个参数：
>
> - dirname：String类型，需要读取模块的文件的所在目录
> - useSubdirectories：Boolean类型，是否遍历子目录
> - RegExp：RegExp类型，匹配的规则
>
> 该方法返回一个函数（这个函数就相当于require或者import），这个函数中还包含一些静态属性和方法：
>
> - resolve：Function类型，接受一个参数request，request为dirname路径下面匹配文件的相对路径，返回这个匹配文件相对于整个工程的相对路径
> - keys：Function类型，返回一个数组，由匹配成功的文件所组成的数组
> - id：String类型，执行环境的 id

### 案例：

```js
const path = require('path')
const moduleList = {}

function reqContext() {
  // 匹配特定 js 文件
  const moduleFn = require.context('@/apis', true, /(api).js$/)
  let fileList = moduleFn.keys()
  if (!fileList.length) return
  fileList.forEach((paths) => {
    let content = moduleFn(paths)
    // 把导入对象里的内容全部复制进moduleList中，注意这里没有考虑默认导出的情况，默认导出的内容存储在moduleFn(paths).default中
    assignment(content)
  })
}
// 将模块的数据加载进 moduleList 对象中
function assignment(obj) {
  if (!Object.keys(obj).length) return
  let arr = Object.keys(obj)
  arr.forEach((key) => {
    moduleList[key] = obj[key]
  })
}

module.exports = moduleList
```

**注意：**

>  ts环境下需要安装声明文件，然后在tsconfig.json中进行配置
>
> ```shell
> $ yarn add @types/webpack-env -D
> ```
>
> 在tsconfig.json的compilerOptions添加配置：
>
> ```json
> {
>     "compilerOptions": {
>       ... ...
>       "types": ["webpack-env"]
>     }
> }
> ```

> require.context的第一个参数只能传一个常量，不能是变量，否则会报如下错误：
>
> ```txt
> __webpack_require__(...).context is not a function
> ```
