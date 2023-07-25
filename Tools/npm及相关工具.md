## npm

 1. 分别使用以下命令设置全局的安装包目录：

``` shell
$ npm config set prefix "D:\Program Files\nodejs\node_global"
$ npm config set cache "D:\Program Files\nodejs\node_cache"
```

**配置条目含义：**

- registry：npm intall默认下载位置
- prefix：npm全局模块安装位置
- cache：npm缓存文件保存位置

**配置文件位置：**

- userconfig：npm的个性化配置，npm会为电脑中每个用户创建一个个性化配置文件，这个文件就放在每个用户的home路径下。我们使用npm config set命令修改配置时，npm并不会去修改全局配置文件，而是会修改这个修改化配置文件

```shell
prefix=C:\Program Files\nodejs
cache=F:\.npm-cache
registry=https://registry.npmmirror.com/
home=https://npm.taobao.org
```

- globalconfig：全局配置，所有用户共享的配置文件

  配置文件位置：D:\nvm\v16.13.0\node_modules\npm\.npmrc

2. npm 常用命令简写说明：

```shell
-g： 为 --global 的缩写，表示安装到全局目录里
-S： 为 --save 的缩写，表示安装的包将写入package.json里面的dependencies
-D： 为 --save-dev 的缩写，表示将安装的包将写入packege.json里面的devDependencies
i： 为install的缩写，表示安装
```

3. npm 安装模块：

``` shell
$ npm init  				// npm 初始化当前目录
$ npm i  					// 安装所有依赖
$ npm i express 		    // 安装模块到默认dependencies
$ npm i express -g 		// 会安装到配置的全局目录下
$ npm i express -S  		// 安装包信息将加入到dependencies生产依赖
$ npm i express -D  		// 安装包信息将加入到devDependencies开发依赖
$ npm i jquery@1.8.3  	// 安装jquery指定的1.8.3版本
$ npm i jquery@latest     // 安装jquery最新版本

$ npm install git+https://github.com/EricXie79/GP22.git
# 可以将git上的内容拉下来，但是要有keygen权限
```

4. npm 卸载模块：

``` shell
$ npm uninstall express  				// 卸载模块，但不卸载模块留在package.json中的对应信息
$ npm uninstall express -g  			// 卸载全局模块
$ npm uninstall express --save  		// 卸载模块，同时卸载留在package.json中dependencies下的信息
$ npm uninstall express --save-dev    // 卸载模块，同时卸载留在package.json中devDependencies下的信息
```

5. npm 更新模块：

``` shell
$ npm update jquery 			  // 更新最新版本的jquery
$ npm update jquery@2.1.0  	  // 更新到指定版本号的jquery
$ npm install jquery@latest     // 可以直接更新到最后一个新版本

# 使用 ‘npm i 包名@版本号’ 形式来更新包的方式更常见，也更方便
```

6. npm 查看命令：

``` shell
$ npm root                     		 // 查看项目中模块所在的目录
$ npm root -g                  		 // 查看全局安装的模块所在目录
$ npm list 或者 npm ls          	    // 查看本地已安装模块的清单列表
$ npm view jquery dependencies 		 // 查看某个包对于各种包的依赖关系
$ npm view jquery version      		 // 查看jquery最新的版本号
$ npm view jquery versions     		 // 查看所有jquery历史版本号（很实用）
$ npm view jquery              		 // 查看最新的jquery版本的信息
$ npm info jquery 			 		 // 查看jquery的详细信息，等同于上面的npm view jquery
$ npm list jquery 或 npm ls jquery     // 查看本地已安装的jquery的详细信息
$ npm view jquery repository.url       // 查看jquery包的来源地址
```

7. npm 其他命令：

``` shell
$ npm cache clean             // 清除npm的缓存
$ npm prune                   // 清除项目中没有被使用的包
$ npm outdated                // 检查模块是否已经过时
$ npm repo jquery             // 会打开默认浏览器跳转到github中jquery的页面
$ npm docs jquery             // 会打开默认浏览器跳转到github中jquery的README.MD文件信息
$ npm home jquery             // 会打开默认浏览器跳转到github中jquery的主页
```

8. 清除缓存缓存：

``` shell
# 错误码是-4048需要清除
$ npm cache verify
$ npm cache clean  --force
```

9. 重设代理：

``` shell
# 查看代理
$ npm config get proxy
$ npm config get https-proxy

# 设置代理，错误内容 ECONNREFUSED 一直连接不上
$ npm config set proxy null
$ npm config set https-proxy null
```

10. npm初始化package.json文件：

``` shell
$ npm  init [-y]              // -y表示静默初始化，自动使用默认值
```

11. 发布npm包：

- 注册npm账号： https://www.npmjs.com/

- 在命令行登录： npm login

- 修改package.json：

``` json
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

``` shell
$ npm publish
```

13. 删除发布的包：

``` shell
$ npm unpublish <pkg>[@<version>]
```

14. 让发布的包过期：

``` shell
# 语法
$ npm deprecate <pkg>[@<version range>] <message>

$ npm deprecate my-thing@"< 0.2.3" "critical bug fixed in v0.2.3"
# 提示下载该包0.2.3以前版本的用户:"critical bug fixed in v0.2.3"

$ npm deprecate my-thing@1.x "1.x is no longer supported"
# 提示下载该包1.x版本的用户:"1.x is no longer supported"
```

15. 给npm包打tag

> 当我们使用npm publish发布包的时候，该版本的包会被自动打上latest的tag，也可以手动npm publish --tags beta，这样打的就是beta的tag了
>
> 除了发包的时候可以指定tag，平时也可以通过npm dist-tag命令来给某个版本的包打上tag

```shell
# 添加tag
$ npm dist-tag add <package-spc> <tag>
# 删除tag
$ npm dist-tag rm <package-spc> <tag>
# 打印所有tag
$ npm dist-tag ls <package-spc>
```

给包的某个版本打上tag之后，就可以通过tag来安装指定版本了:

```shell
$ npm install typescript@beta
```

16. 使用package.json中的config给页面传参：

``` json
// package.json
"config": {
  "PORT": 8080
}
```

```shell
# 开始运行
$ node main.js
```

```js
// main.js
console.log(process.env.npm_package_config_PORT)
// 8080
```

17. 使用cross-env设置环境变量给页面传参：

> 当设置环境变量为 `NODE_ENV=production` 时，易造成 Windows 命令的阻塞。（除了 Windows 上的 Bash，因其使用本机 Bash）
>
> `cross-env` 使用单个命令，而不用担心环境变量的设置。像运行在 `POSIX系统` 上一样进行设置，`cross-env` 可以进行正确的设置。
>
> 简而言之就是通过cross-env设置环境变量，可以很好的是适配多个平台，无语根据平台来设置相对应的变量。

- 1. 安装

```shell
$ npm install --save-dev cross-env
```

- 2. 用法

``` json
// package.json
"scripts": {
    "start": "cross-env ENV=position node main.js",
}
```

``` shell
# 开始执行
$ npm run start
```

```js
// main.js
console.log(process.env.ENV)
// production
```

## nvm

1. nvm配置文件：

``` 
文件位置：D:\nvm

root: D:\nvm
path: C:\Program Files\nodejs
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

2. nvm常用命令：

``` shell
$ nvm list                       //查看已安装的nodejs版本
$ nvm list available             //查看网络上可以安装的版本
$ nvm on                         // 启用node.js版本管理
$ nvm off                        // 禁用node.js版本管理(不卸载任何东西)
$ nvm install <version>          // 安装node.js的命名 version是版本号 例如：nvm install 8.12.0
$ nvm use <version>              //使用某一version的nodejs
$ nvm uninstall <version>        // 卸载指定版本的nodejs
```

## nrm

> ‘npm registry manager’ (即：指的是‘npm’的镜像源管理工具)。nrm的诞生就是为了使用者在各个不同npm源之间来回切换，npm是全世界最大的软件注册表，每天有数以万计的人从这个地方下载软件，nrm的作用就是切换下载npm中资源的服务器。例如：你在国内，你使用npm谷歌 的源下载软件就龟速，但是你使用npm淘宝的源就比较快，这时候你就可以使用nrm来快速切换源了

> 原来下载的包，刚开始是只存在于国外的‘npm’服务器上,但是由于网络原因，经常访问不到，这时候，我们可以在国内创建一个和官网完全一样的‘npm’服务器，只不过数据都是从人家那里拿过来的，除此之外，使用方式与方法完全一样

- `nrm ls` 查看所有源，显示结果包括：npm 源名称、源地址等信息

- `npm use <registry>`：切换 npm 源地址
- `nrm test [registry]`：测试源的访问速度； 不加 ‘registry’ 时，默认测试所有的源速度

## npx

**一、用途一：**

npx主要用于调用项目中的某个模块的指令，以webpack为例，比如全局安装的是webpack5.1.3，项目安装的是webpack3.6.0这时在终端执行`webpack --version`的话，显示结果会是 webpack 5.1.3，也就是说使用的是全局的。原因非常简单，我们在使用webpack命令时，其实是在环境变量中去寻找，而项目中的webpack是安装在`node_modules/.bin`文件夹中的，这个文件夹是不存在于环境变量中的，所以找不到。如何使用项目（局部）中安装的webpack？

- 方式一：明确查找到node_module下面的webpack：

```text
./node_modules/.bin/webpack --version
```

- 方式二：在 scripts定义脚本，来执行webpack：

``` json
"scripts": {
    "webpack": "webpack --version"
}
```

- 方式三：使用npx，npx的原理非常简单，它会到当前目录的node_modules/.bin目录下查找对应的命令：

``` shell
$ npx webpack --version
```

**二、用途二：**

另外，使用`npx`可以避免全局安装模块，比如：`create-react-app`这个模块是全局安装，`npx`可以运行它，而且不进行全局安装:

```shell
$ npx create-react-app my-react-app
```

上面代码运行时，`npx`将`create-react-app`下载到一个临时目录，使用以后再删除。所以以后再次执行上面的命令，会重新下载`create-react-app`提供使用后再移除。并且在下载全局模块时，`npx`允许指定版本：

```shell
$ npx webpack@4.44.1 ./src/index.js -o ./dist/main.js
```

**注意：**只要`npx`后面的模块无法在本地发现，就会下载同名模块。如果想让`npx`强制使用本地模块，不下载远程模块，可以使用`--no-install`参数，如果本地不存在该模块，就会报错：

```shell
$ npx --no-install webpack-dev-server
```

反过来，如果本地存在同名的模块，但是还是想使用远程的新版本模块，可以使用`--ignore-existing`参数:

```shell
$ npx --ignore-existing create-react-app my-react-app
```

**三、用途三：**

除此之外，还可以利用`npx`指定`node`版本运行脚本：

```shell
$ npx node@14.15.0 -v
```

上面命令会使用`v14.15.0`版本的`node`执行脚本。原理是从`npm`下载这个版本的`node`，使用后再删掉

## nodemon

[nodemon](https://links.jianshu.com/go?to=http%3A%2F%2Fnodemon.io%2F)是一种工具，可在检测到目录中的文件更改时通过自动重新启动节点应用程序来帮助开发基于node.js的应用程序

- **全局安装**

``` shell
$ npm i -g nodemon
```

- **本地安装**

``` shell
$ npm i -D nodemon
```

``` json
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

```shell
nodemon -e ts,js --exec npx ts-node ./src/app.ts
```

## anywhere

> Anywhere 随启随用的静态文件服务器， 随时随地将你的当前目录变成一个静态文件服务器的根目录。

- 安装：

``` shell
$ npm install anywhere -g
```

- 使用：

``` shell
$ anywhere -p 8000  // 指定端口
$ anywhere -h localhost -p 8888  // 指定端口和主机名
$ anywhere -s  // or start it but silent(don't open browser)
$ anywhere --help
```

## http-server

> `http-server`是一个简单的、零配置的命令行静态 HTTP 服务器。它对于生产使用来说足够强大，但它足够简单和可破解，可以用于测试、本地开发和学习。

- 使用 `npx` 可以运行http-server而无需先安装它：

``` shell
$ npx http-server [path] [options]
// [path]默认为./public(如果文件夹存在)，不存在则为./
```

- 全局安装：

``` shell
$ npm install --global http-server
```

- **options**

``` shell
-p 端口号 (默认 8080)
-a IP地址 (默认 0.0.0.0) 指定的地址必须是空闲的，并且是本机的网卡地址
-o 在开始服务后打开浏览器
```

## ts-node

> Ts-node 是 Node.js 的 TypeScript 执行引擎和 REPL（交互式解释器）。
>
> 它将 TypeScript 转换为 JavaScript，使我们能够直接在 Node.js 上执行 TypeScript 而无需预编译。

**安装：**

```shell
$ npm install -D typescript
$ npm install -D ts-node
```

## node-dev

> Node-DEV是Node.js的开发工具，当文件被修改时，它会自动重新启动nodejs进程。
>
> 与supervisor或Nodemon之类的工具相反，它不会扫描文件系统以查看要监听的文件。取而代之的是通过require()来判断项目依赖的文件，它将仅监听实际需要的文件。
>
> 这意味着我们不需要配置任何includes或excludes rule。如果修改仅在客户端上使用但从未在服务器上运行的JS文件，Node-Dev将知道这一点并且不会重新启动该过程。
>
> 这也意味着我们不必配置需要监听文件的扩展名。例如，仅需一个.json文件或.ts文件作为入口即可自动的进行监听。

**安装：**

```shell
$ npm install -g node-dev
```

**使用：**

```shell
$ node-dev server.js
```

## ts-node-dev

> 可以理解为ts-node的升级版，他不仅可以直接编译运行ts文件，并且会监测文件的变化，并重新执行，它其实就是node-dev + ts-node

**安装：**

```shell
$ npm i ts-node-dev --save-dev
```

**使用：**

使用它时可以随意使用`node-dev`和`ts-node`的选项：

```shell
$ ts-node-dev --respawn --transpile-only server.ts
```

`ts-node-dev`有一个别名`tsnd`：

```shell
$ tsnd --respawn server.ts
```

## concurrently

> 如果想要使用npm实现任务自动化，通常并发运行多个命令的方法是使用`npm run watch-js & npm run watch-css`。这很好，但是很难跟踪不同的输出。此外，如果一个进程失败，其他进程仍然在运行，我们甚至无法发现。另一种选择是在不同的终端中运行所有命令，但是这样做很麻烦。使用concurrently的优势：
>
> - 跨平台(包括windows)
>
> - 输出后面加上了前缀，理容易区分是哪个指令的输出
>
> - 使用`--kill-others`选项后,一旦其中一个指令死亡，所有指令都会被终止

**安装：**

```shell
$ npm install concurrently --save
```

**使用：**

```shell
$ concurrently "command1 arg" "command2 arg"
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

## 通过ESModule获取package.json中的数据

```js
// 获取版本号
import { version } from "@walletconnect/universal-provider/package.json";
```

## npm link

> 如果库包在开发或迭代后，不适合发布到线上进行调试（过程繁琐且会导致版本号膨胀），这里`npm link`就派上用场了。`npm link`可以帮助我们模拟包安装后的状态，它会在系统中做一个快捷方式映射，让本地的包就好像install过一样，可以直接使用

### 局部链接

> `npm link`操作会在`project1`的`node_modules`目录下创建一个`module1`的超链接（类似Windows的快捷方式），链接到`project_npmlink/module1`，生成的虚拟包名会根据`module1`的`package.json`进行指定

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/71749ab720ef459aa29f4fa46fbca4bf.png" alt="71749ab720ef459aa29f4fa46fbca4bf" style="zoom: 67%;" />

```bash
$ cd xxx/project_npmlink/project1
$ npm link ../module1
```

### 全局链接

> 项目和模块不在同一个目录下，需要先把模块链接到全局，然后再在项目中链接模块

```bash
$ cd xxx/project_npmlink/module1
$ npm link
```

npm link 操作会在全局 node_modules 目录下创建一个 module-name 的超链接。

```bash
$ cd xxx/project_npmlink/project1
$ npm link module1
```

此时只需要指定module-name，在项目的node_modules目录下创建一个module-name的超链接，链接到全局链接的包，然后再由全局目录下的超链接，链接到具体的代码目录下

### 删除npm link的链接

- **解除项目和模块的链接**

```bash
# 进入项目目录，解除链接
$ cd xxx/project_npmlink/project1
$ npm unlink module1
```

- **解除模块的全局链接**

```bash
# 进入项目目录，解除链接
$ cd xxx/project_npmlink/module1
$ npm unlink module1
```



