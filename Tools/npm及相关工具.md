#### npm

 1. 分别使用以下命令设置全局的安装包目录：

    ``` 
    npm config set prefix "D:\Program Files\nodejs\node_global"
    npm config set cache "D:\Program Files\nodejs\node_cache"
    
    配置条目含义：
    => registry：npm intall默认下载位置；
    => prefix：npm全局模块安装位置；
    => cache：npm缓存文件保存位置；
    
    配置文件位置：
    => userconfig：npm的个性化配置，npm会为电脑中每个用户创建一个个性化配置文件，这个文件就放在每个用户的home路径下。我们使用npm config set命令修改配置时，npm并不会去修改全局配置文件，而是会修改这个修改化配置文件。
        配置文件位置：C:\Users\ilmango\.npmrc
        prefix=C:\Program Files\nodejs
        cache=F:\.npm-cache
        registry=https://registry.npmmirror.com/
        home=https://npm.taobao.org
       
    => globalconfig：全局配置，所有用户共享的配置文件。
       配置文件位置：D:\nvm\v16.13.0\node_modules\npm\.npmrc
    ```

2. npm 常用命令简写说明：

   ```js
   -g： 为 --global 的缩写，表示安装到全局目录里
   -S： 为 --save 的缩写，表示安装的包将写入package.json里面的dependencies
   -D： 为 --save-dev 的缩写，表示将安装的包将写入packege.json里面的devDependencies
    i： 为install的缩写，表示安装
   ```

3. npm 安装模块：

   ``` js
   npm init  				// npm 初始化当前目录
   npm i  					// 安装所有依赖
   npm i express 		    // 安装模块到默认dependencies
   npm i express -g 		// 会安装到配置的全局目录下
   npm i express -S  		// 安装包信息将加入到dependencies生产依赖
   npm i express -D  		// 安装包信息将加入到devDependencies开发依赖
   npm i jquery@1.8.3  	// 安装jquery指定的1.8.3版本
   npm i jquery@latest     // 安装jquery最新版本
   
   npm install git+https://github.com/EricXie79/GP22.git
   //可以将git上的内容拉下来，但是要有keygen权限
   ```

4. npm 卸载模块：

   ``` js
   npm uninstall express  				// 卸载模块，但不卸载模块留在package.json中的对应信息
   npm uninstall express -g  			// 卸载全局模块
   npm uninstall express --save  		// 卸载模块，同时卸载留在package.json中dependencies下的信息
   npm uninstall express --save-dev    // 卸载模块，同时卸载留在package.json中devDependencies下的信息
   ```

5. npm 更新模块：

   ``` js
   npm update jquery 			  // 更新最新版本的jquery
   npm update jquery@2.1.0  	  // 更新到指定版本号的jquery
   npm install jquery@latest     // 可以直接更新到最后一个新版本
   
   // 使用 ‘npm i 包名@版本号’ 形式来更新包的方式更常见，也更方便
   ```

6. npm 查看命令：

   ``` js
   npm root                     		 // 查看项目中模块所在的目录
   npm root -g                  		 // 查看全局安装的模块所在目录
   npm list 或者 npm ls          	    // 查看本地已安装模块的清单列表
   npm view jquery dependencies 		 // 查看某个包对于各种包的依赖关系
   npm view jquery version      		 // 查看jquery最新的版本号
   npm view jquery versions     		 // 查看所有jquery历史版本号（很实用）
   npm view jquery              		 // 查看最新的jquery版本的信息
   npm info jquery 			 		 // 查看jquery的详细信息，等同于上面的npm view jquery
   npm list jquery 或 npm ls jquery     // 查看本地已安装的jquery的详细信息
   npm view jquery repository.url       // 查看jquery包的来源地址
   ```

7. npm 其他命令：

   ``` js
   npm cache clean             // 清除npm的缓存
   npm prune                   // 清除项目中没有被使用的包
   npm outdated                // 检查模块是否已经过时
   npm repo jquery             // 会打开默认浏览器跳转到github中jquery的页面
   npm docs jquery             // 会打开默认浏览器跳转到github中jquery的README.MD文件信息
   npm home jquery             // 会打开默认浏览器跳转到github中jquery的主页
   ```

8. 清除缓存缓存：

   ``` js
   // 错误码是-4048需要清除
   npm cache verify
   npm cache clean  --force
   ```

9. 重设代理：

   ``` js
   // 查看代理
   npm config get proxy
   npm config get https-proxy
   
   // 设置代理，错误内容 ECONNREFUSED 一直连接不上
   npm config set proxy null
   npm config set https-proxy null
   ```

10. npm初始化package.json文件：

   ``` js
   npm  init [-y]              // -y表示静默初始化，自动使用默认值
   ```

11. 发布npm包：

    - 注册npm账号： https://www.npmjs.com/

    - 在命令行登录： npm login

    - 修改package.json：

      ``` js
      {
          "name": "hy_test_utils", # 模块名一定要唯一
          "version": "1.0.0", # 版本要符合semver规范
          "description": "a test utils",
          "main": "index.js", # 入口文件，在script中使用脚本不指定文件时默认就是main指定的文件
          "scripts": {
              "test": "echo \"Error: no test specified\" && exit 1"
          },
          "config": {
              "PORT": 8080
          }
          "author": "coderwhy", # 模块作者
          "license": "MIT", # 模块开源协议
          "repository": { # 存储模块的仓库
              "type": "git",
              "url": "https://github.com/coderwhy/coderwhy"
          },
          "homepage": "https://github.com/coderwhy/coderwhy", # 模块主页
          "keywords": [ # 关键字，用于帮助在npmjs.com中搜索
              "why",                                   
              "coderwhy",
              "utils"
          ]
          devDependencies :{ # 在开发、测试环境中用到的依赖
              // 模块可以从git上下拉
              "gp22" :"git+ssh://git@github.com:EricXie79/GP22.git"
          }  
          dependencies: { # 在生产环境中需要用到的依赖
              // dependencies版本中的锁定
              "jQuery" : "^3.1.0"     锁定大版本major
              "jQuery" : "~3.1.0"     锁定大版本和小版本minor
              "jQuery" : "3.1.0"      锁定所有版本
              "jQuery" : "*"          最新版本  
          }
      }
      ```

    - 撰写README.md文件：这个文件的内容会显示在模块页中

    - 发布到npm registry上：npm publish [<folder>]

12. 更新npm仓库：修改版本号(最好符合semver规范)，因为一个包不能存在两个相同版本号，然后重新发布：

    ``` js
    npm publish
    ```

13. 删除发布的包：

    ``` js
    npm unpublish <pkg>[@<version>]
    ```

14. 让发布的包过期：

    ``` js
    // 语法
    npm deprecate <pkg>[@<version range>] <message>
        
    npm deprecate my-thing@"< 0.2.3" "critical bug fixed in v0.2.3"
    // 提示下载该包0.2.3以前版本的用户:"critical bug fixed in v0.2.3"
    
    npm deprecate my-thing@1.x "1.x is no longer supported"
    // 提示下载该包1.x版本的用户:"1.x is no longer supported"
    ```

15. 使用package.json中的config给页面传参：

    ``` js
    // package.json
    "config": {
      "PORT": 8080
    }
    ```

    ```js
    // 开始运行
    node main.js
    
    // main.js
    console.log(process.env.npm_package_config_PORT)
    // 8080
    ```

16. 使用cross-env设置环境变量给页面传参：

    > 当设置环境变量为 `NODE_ENV=production` 时，易造成 Windows 命令的阻塞。（除了 Windows 上的 Bash，因其使用本机 Bash）
    >
    > `cross-env` 使用单个命令，而不用担心环境变量的设置。像运行在 `POSIX系统` 上一样进行设置，`cross-env` 可以进行正确的设置。
    >
    > 简而言之就是通过cross-env设置环境变量，可以很好的是适配多个平台，无语根据平台来设置相对应的变量。

    - 1. 安装

    ```js
    npm install --save-dev cross-env
    ```

    - 2. 用法

    ``` js
    // package.json
    "scripts": {
        "start": "cross-env ENV=position node main.js",
    }
    ```

    ``` js
    // 开始执行
    npm run start
    
    // main.js
    console.log(process.env.ENV)
    // production
    ```

#### nvm

1. nvm配置文件：

   ``` 
   文件位置：D:\nvm
   
   root: D:\nvm
   path: C:\Program Files\nodejs
   node_mirror: https://npm.taobao.org/mirrors/node/
   npm_mirror: https://npm.taobao.org/mirrors/npm/
   ```

2. nvm常用命令：

   ``` js
   nvm list                       //查看已安装的nodejs版本
   nvm list available             //查看网络上可以安装的版本
   nvm on                         // 启用node.js版本管理
   nvm off                        // 禁用node.js版本管理(不卸载任何东西)
   nvm install <version>          // 安装node.js的命名 version是版本号 例如：nvm install 8.12.0
   nvm use <version>              //使用某一version的nodejs
   nvm uninstall <version>        // 卸载指定版本的nodejs
   ```


#### nrm

> ‘npm registry manager’ (即：指的是 ‘npm’ 的镜像源管理工具)。nrm 的诞生就是为了使用者在各个不同 npm 源之间来回切换，npm 是全世界最大的软件注册表，每天有数以万计的人从这个地方下载软件，nrm 的作用就是切换下载 npm 中资源的服务器。例如：你在国内，你使用 npm谷歌 的源下载软件就龟速，但是你使用 npm淘宝 的源就比较快，这时候你就可以使用 nrm 来快速切换源了。

> 原来下载的包，刚开始是只存在于国外的 ‘npm’ 服务器上,但是由于网络原因，经常访问不到，这时候，我们可以在国内创建一个和官网完全一样的 ‘npm’ 服务器，只不过数据都是从人家那里拿过来的，除此之外，使用方式与方法完全一样。

- `nrm ls` 查看所有源，显示结果包括：npm 源名称、源地址等信息；

- `npm use <registry>`：切换 npm 源地址；
- `nrm test [registry]`：测试源的访问速度； 不加 ‘registry’ 时，默认测试所有的源速度；

#### npx

**一、用途一：**

npx主要用于调用项目中的某个模块的指令，以webpack为例，比如全局安装的是webpack5.1.3，项目安装的是webpack3.6.0这时在终端执行`webpack --version`的话，显示结果会是 webpack 5.1.3，也就是说使用的是全局的。原因非常简单，我们在使用webpack命令时，其实是在环境变量中去寻找，而项目中的webpack是安装在`node_modules/.bin`文件夹中的，这个文件夹是不存在于环境变量中的，所以找不到。如何使用项目（局部）中安装的webpack？

- 方式一：明确查找到node_module下面的webpack：

  ```js
  ./node_modules/.bin/webpack --version
  ```

- 方式二：在 scripts定义脚本，来执行webpack：

  ``` js
  "scripts": {
      "webpack": "webpack --version"
  }
  ```

- 方式三：使用npx，npx的原理非常简单，它会到当前目录的node_modules/.bin目录下查找对应的命令：

  ``` shell
  $ npx webpack --version
  ```

**二、用途二：**

另外，使用 `npx` 可以避免全局安装模块，比如：`create-react-app` 这个模块是全局安装，`npx` 可以运行它，而且不进行全局安装:

```shell
$ npx create-react-app my-react-app
```

上面代码运行时，`npx` 将 `create-react-app` 下载到一个临时目录，使用以后再删除。所以以后再次执行上面的命令，会重新下载 `create-react-app` 提供使用后再移除。并且在下载全局模块时，`npx` 允许指定版本：

```shell
$ npx webpack@4.44.1 ./src/index.js -o ./dist/main.js
```

**注意：**只要 `npx` 后面的模块无法在本地发现，就会下载同名模块。如果想让 `npx` 强制使用本地模块，不下载远程模块，可以使用 `--no-install` 参数，如果本地不存在该模块，就会报错：

```shell
$ npx --no-install webpack-dev-server
```

反过来，如果本地存在同名的模块，但是还是想使用远程的新版本模块，可以使用 `--ignore-existing` 参数:

```shell
$ npx --ignore-existing create-react-app my-react-app
```

**三、用途三：**

除此之外，还可以利用 `npx` 指定 `node` 版本运行脚本：

```shell
$ npx node@14.15.0 -v
```

上面命令会使用 `v14.15.0` 版本的 `node` 执行脚本。原理是从 `npm` 下载这个版本的 `node`，使用后再删掉

#### pnpm

- ##### pnpm的优势：

  1. pnpm 内部使用`基于内容寻址`的文件系统来存储磁盘上所有的文件：
     - 不会重复安装同一个包。使用`npm/yarn` 的时候，如果100个包依赖`lodash` ，那么就可能安装了100次`lodash` ，磁盘中就有100个地方写入了这部分代码。但是`pnpm`会只在一个地方写入这部分代码，后面使用会直接使用`hard link`
     - 即使一个包的不同版本，pnpm 也会极大程度地复用之前版本的代码。举个例子，比如 lodash 有 100 个文件，更新版本之后多了一个文件，那么磁盘当中并不会重新写入 101 个文件，而是保留原来的 100 个文件的 `hardlink`，仅仅写入那`一个新增的文件`。
  2. monorepo 的宗旨就是用一个 git 仓库来管理多个子项目，所有的子项目都存放在根目录的`packages`目录下，那么一个子项目就代表一个`package`。
  3. 之前在使用 npm/yarn 的时候，由于 node_module 的扁平结构，如果 A 依赖 B， B 依赖 C，那么 A 当中是可以直接使用 C 的，但问题是 A 当中并没有声明 C 这个依赖。因此会出现这种非法访问的情况。 但 pnpm 自创了一套依赖管理方式，很好地解决了这个问题，保证了安全性。

- ##### 依赖管理

  > pnpm 会在全局的 store 目录里存储项目 `node_modules` 文件的 `hard links` 。因为这样一个机制，导致每次安装依赖的时候，如果是个相同的依赖，有好多项目都用到这个依赖，那么这个依赖实际上最优情况(即版本相同)只用安装一次。

  **第一阶段：npm@3 之前版本**

  ```text
  node_modules
  └─ foo
     ├─ index.js
     ├─ package.json
     └─ node_modules
        └─ bar
           ├─ index.js
           └─ package.json
  ```

  - 依赖树层级太深，会导致 Windows 上的目录路径过长问题
  - 相同包在不同的依赖项中需要时，会存在多个相同副本

  **第二阶段：npm@3 版本，扁平化处理**

  所有的依赖都被拍平到`node_modules`目录下，不再有很深层次的嵌套关系。这样在安装新的包时，根据 node require 机制，会不停往上级的`node_modules`当中去找，如果找到相同版本的包就不会重新安装，解决了大量包重复安装的问题，而且依赖层级也不会太深。

  ```text
  node_modules
  ├─ foo
  |  ├─ index.js
  |  └─ package.json
  └─ bar
     ├─ index.js
     └─ package.json
  ```

  但还是存在一些问题

  - 依赖结构的**不确定性**。
  - 扁平化算法本身的**复杂性**很高，耗时较长。
  - 项目中仍然可以**非法访问**没有声明过依赖的包

  这就是为什么会产生依赖结构的`不确定`问题，也是 `lock 文件`诞生的原因，无论是`package-lock.json`(npm 5.x才出现)还是`yarn.lock`，都是为了保证 install 之后都产生确定的`node_modules`结构。尽管如此，npm/yarn 本身还是存在`扁平化算法复杂`和`package 非法访问`的问题，影响性能和安全。

  **第三阶段：pnpm**

  由于扁平化算法的极其复杂，以及会存在多项目间相同依赖副本的情况。pnpm 在尝试解决这些问题时，放弃了扁平化处理 node_modules 的方式。而是采用 **硬链+软链** 方式。

  ```text
  node_modules
  ├─ .pnpm
  |  ├─ foo@1.0.0/node_modules/foo
  |  |  └─ index.js
  |  └─ bar@2.0.0/node_modules/bar
  ├─ foo -> .pnpm/foo@1.0.0/node_modules/foo
  └─ bar -> .pnpm/bar@2.0.0/node_modules/bar
  ```

  node_modules 根目录中的包只是一个符号链接。`require('foo')` 将执行 `node_modules/.pnpm/foo@1.0.0/node_modules/foo/indexjs` 中的文件（这里是硬链接），而不是 `node_modules/foo/index.js` 中的文件。

  **这种布局结构的一大好处是只有真正在依赖项中（package.json dependences）的包才能访问**，**举个🌰**：

  安装一个`express` 依赖，会在 node_modules 中形成这样两个目录结构:

  ```text
  node_modules/express/...
  node_modules/.pnpm/express@4.17.1/node_modules/xxx
  ```

  其中第一个路径是 nodejs 正常寻找路径会去找的一个目录，如果去查看这个目录下的内容，会发现里面连个 `node_modules` 文件都没有：

  ```text
  ▾ express
      ▸ lib
        History.md
        index.js
        LICENSE
        package.json
        Readme.md
  ```

  实际上这个文件只是个软连接，它会形成一个到第二个目录的一个软连接(类似于软件的快捷方式)，这样 node 在找路径的时候，最终会找到 .pnpm 这个目录下的内容。

  其中这个 `.pnpm` 是个虚拟磁盘目录，然后 express 这个依赖的一些依赖会被平铺到 `.pnpm/express@4.17.1/node_modules/` 这个目录下面，这样保证了依赖能够 require 到，同时也不会形成很深的依赖层级。在保证了 nodejs 能找到依赖路径的基础上，同时也很大程度上保证了依赖能很好的被放在一起。

#### nodemon

[nodemon](https://links.jianshu.com/go?to=http%3A%2F%2Fnodemon.io%2F) 是一种工具，可在检测到目录中的文件更改时通过自动重新启动节点应用程序来帮助开发基于 node.js 的应用程序。

- **全局安装**

  ``` js
  npm i -g nodemon
  ```

- **本地安装**

  ``` js
  npm i -D nodemon
  ```

  ``` js
  // 本地安装后可以配置脚本使用nodemon,这里nodemon后面没有加文件
  // 它会默认加载package.json文件中main属性配置的文件
  "script": {
      "dev": "nodemon"
  }
  ```

**配置文件：**

> 这个文件是nodemon的配置文件，可以在这里对nodemon进行各种配置，比如说下面就是对nodemon监听的文件进行忽略

```json
// nodemon.json
{
    "ignore": [
        "log/*"
    ]
}
```

**nodemon骚操作：**

> 使用nodemon配合ts-node实时运行ts项目：

```json
nodemon -e ts,js --exec npx ts-node ./src/app.ts
```

#### anywhere

> Anywhere 随启随用的静态文件服务器， 随时随地将你的当前目录变成一个静态文件服务器的根目录。

- 安装：

  ``` js
  npm install anywhere -g
  ```

- 使用：

  ``` js
  $ anywhere -p 8000  // 指定端口
  $ anywhere -h localhost -p 8888  // 指定端口和主机名
  $ anywhere -s  // or start it but silent(don't open browser)
  $ anywhere --help
  ```

#### http-server

> `http-server`是一个简单的、零配置的命令行静态 HTTP 服务器。它对于生产使用来说足够强大，但它足够简单和可破解，可以用于测试、本地开发和学习。

- 使用 `npx` 可以运行http-server而无需先安装它：

  ``` js
  npx http-server [path] [options]
  // [path]默认为./public(如果文件夹存在)，不存在则为./
  ```

- 全局安装：

  ``` js
  npm install --global http-server
  ```

- **options**

  ``` js
  -p 端口号 (默认 8080)
  -a IP地址 (默认 0.0.0.0) 指定的地址必须是空闲的，并且是本机的网卡地址
  -o 在开始服务后打开浏览器
  ```


#### ts-node

> Ts-node 是 Node.js 的 TypeScript 执行引擎和 REPL（交互式解释器）。
>
> 它将 TypeScript 转换为 JavaScript，使我们能够直接在 Node.js 上执行 TypeScript 而无需预编译。

**安装：**

```json
npm install -D typescript
npm install -D ts-node
```

#### node-dev

> Node-DEV是Node.js的开发工具，当文件被修改时，它会自动重新启动nodejs进程。
>
> 与supervisor或Nodemon之类的工具相反，它不会扫描文件系统以查看要监听的文件。取而代之的是通过require()来判断项目依赖的文件，它将仅监听实际需要的文件。
>
> 这意味着我们不需要配置任何includes或excludes rule。如果修改仅在客户端上使用但从未在服务器上运行的JS文件，Node-Dev将知道这一点并且不会重新启动该过程。
>
> 这也意味着我们不必配置需要监听文件的扩展名。例如，仅需一个.json文件或.ts文件作为入口即可自动的进行监听。

**安装：**

```json
npm install -g node-dev
```

**使用：**

```json
node-dev server.js
```

#### ts-node-dev

> 可以理解为ts-node的升级版，他不仅可以直接编译运行ts文件，并且会监测文件的变化，并重新执行，它其实就是node-dev + ts-node

**安装：**

```json
npm i ts-node-dev --save-dev
```

**使用：**

使用它时可以随意使用`node-dev`和`ts-node`的选项：

```
ts-node-dev --respawn --transpile-only server.ts
```

`ts-node-dev`有一个别名`tsnd`：

```
tsnd --respawn server.ts
```

#### concurrently

> 如果想要使用 npm 实现任务自动化，通常并发运行多个命令的方法是使用 `npm run watch-js & npm run watch-css`。这很好，但是很难跟踪不同的输出。此外，如果一个进程失败，其他进程仍然在运行，我们甚至无法发现。另一种选择是在不同的终端中运行所有命令，但是这样做很麻烦。使用concurrently的优势：
>
> - 跨平台(包括windows)
>
> - 输出后面加上了前缀，理容易区分是哪个指令的输出
>
> - 使用了`--kill-others` 选项后,一旦其中一个指令死亡，所有指令都会被终止

**安装：**

```js
npm install concurrently --save
```

**使用：**

```js
concurrently "command1 arg" "command2 arg"
```

在package.json中使用时，要注意转义双引号：

```json
"scripts": {
  "start": "concurrently \"command1 arg\" \"command2 arg\""
}
```

**示例：**

> 使用concurrently实现先通过tsc编译ts文件为js文件，然后使用nodemon实时监听编译后的入口文件。如果使用`tsc -w & nodemon dist/app.js`会导致出错==原因未知==。

```json
"scripts": {
  "start": "concurrently \"tsc -w\" \"nodemon dist/app.js\""
}
```

