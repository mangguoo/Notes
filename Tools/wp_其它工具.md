# webpack 工具

## postcss

> PostCSS本身只有解析能力，他的各种特性(加前缀、样式兼容....)全靠插件，目前至少有两百多个插件可供使用。

PostCSS工具可以直接在命令行中使用，但是需要单独安装一个工具postcss-cli：

```shell
$ npm install postcss postcss-cli
```

然后就可以使用postcss了，但是postcss只能解析css文件，要想使用他的各种功能就需要使用插件：

```shell
$ npx postcss -o o.css ./a.css
// 输出文件与输入文件相同，但是postcss会检查到输入文件中的语法错误
```

### postcss-preset-env

> 该插件会根据browerlist加载指定的css兼容性样式，还可以为一些需要添加前缀的css规则添加前缀

```shell
$ npm install postcss-preset-env -D

$ npx postcss --use postcss-preset-env -o end.css ./src/css/style.css
```

```js
.title {
  color: #12345678;
  font-weight: 700;
  font-size: 30px;
  user-select: none;
}

<- 转换后 ->
    
.title {
  color: rgba(18,52,86,0.47059);
  font-weight: 700;
  font-size: 30px;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
}
```

### autoprefixer

> 该插件可以根据browserlist的需求来给css属性添加前缀：

**安装autoprefixer：**

```shell
$ npm install autoprefixer -D
```

**在postcss.config.js文件中进行配置(与使用命令行配置相同)：**

```js
const autopreFixer = require("autoprefixer")

module.exports = {
    plugins: [
      autopreFixer
    ]
}
```

**使用autoprefixer添加前缀：**

```shell
$ npx postcss -o end.css ./src/css/style.css
```

### postcss-import

> 该插件可以让我们在css中继续导入其它的css文件：

- **reset.css**

```js
* {
    margin: 0;
    padding: 0;
}
```

- **index.css**

```css
@import "./reset.css";

.title {
    color: #12345678;
    user-select: none;
}
```

- **output.css**

```css
* {
    margin: 0;
    padding: 0;
}

.title {
    color: #12345678;
    user-select: none;
}
```

### cssnano

> 该插件可以对css代码进行压缩

- **index.css**

```css
{
	margin: 0;
	padding: 0;
}
body {
	margin: 10px 20px 10px 20px;
}
body {
	color: rgba(111, 111, 111, 0.5);
}
```

- **output.css**

> 可以看到他把相同的选择器进行了合并，并且简化了margin和color，同时删除了文件中所有的空格。

```css
*{margin:0;padding:0}
body{color:hsla(0,0%,44%,.5);margin:10px 20px}
```

### cssnext

> cssnext使用的是下一代css语法，而且高版本的浏览器已支持部分语法，它的使用类似Babel对es6，需要postcss编译。

### precss

> PreCSS是PostCSS的一种插件，用来进行CSS预编译（允许我们使用一些css新特性），它类似于预处理器。

### postcss-pxtorem

> 这个插件可以实现自动渲染px至rem的开发与生产工作，而不需要再手动计算。使用它需要设置一个基准值，即html的font-size的值。然后它会所有非内联样式中的指定属性的单位转换为rem。

**安装：**

> 注：如果报错，则需要指定postcss-pxtorem版本，vue脚手架中安装的postCSS支持的就是5版本（postcss-pxtorem@5）

```shell
$ npm install amfe-flexible --save
$ npm install postcss-pxtorem --save-dev
```

**使用：**

- 在main.js入口文件引入

```js
import 'amfe-flexible'
```

- 创建postcss.config.js配置文件

> rootValue值一般是根据设计稿的大小来设置的，如iphone6，它的物理宽度为750px，dpr为2，也就是说它的实际css宽度为375px。设计稿一般是二倍图（宽度750px），因为市面上的大部分手机屏幕的dpr都是2。
> 这里的rootValue是37.5，它其实只是个默认值，如果html元素上面有fontSize，则这个值会被覆盖，如果使用了类似于像amfe-flexible这样的工具，这个库会动态计算fontSize，因此这里的rootValue设为多少其实无所谓。

```js
module.exports = {
  plugins: {
      "postcss-pxtorem": {
          "rootValue": 37.5, // 设计稿宽度的1/10
           // 需要做转化处理的属性，如`hight``width`等，`*`表示全部
          "propList": ["*"]
      }
  }
}
```

#### 配合amfe-flexible实现rem布局

> postcss-fiexible一般是配合amfe-flexible来使用的，它可以为html、body添加font-size，并且在窗口大小调整时候重新设置font-size，比如手机viewport的宽度为375，那它就会把font-size设置为375/10。

**安装：**

```shell
$ npm install amfe-flexible
```

**使用(在入口文件中引入)：**

```js
import 'amfe-flexible'
```

### postcss-loader

> 真实开发中我们必然不会直接使用命令行工具来对css进行处理，而是可以借助于构建工具。
> 在webpack中配置postcss-loader，webpack就会帮助我们使用postcss来对css文件进行处理了

**安装：**

```shell
$ npm install postcss-loader -D
```

**使用：**

```js
{
  test: /\.css$/,
  use: [
    "style-loader",
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
          postcssOptions: {
              plugins: [
                  require("postcss-preset-env")
              ]
          }
      }
    }
  ]
}
```

也可以为postcss单独创建配置文件，在根目录下创建postcss.config.js：

```js
module.exports = {
      plugins: [
            require("postcss-preset-env")
      ]
}
```

然后就可以在webpack的配置文件中直接使用postcss-loader，而不需要给postcss-loader配置插件信息：

```js
{
    test: /\.css$/,
    use: [
        "style-loader",
        "css-loader",
        "postcss-loader"
    ]
}
```

这是因为postcss-loader是直接调用postcss工具对css文件进行转换。而postcss工具在被使用时会默认读取postcss.config.js文件中的配置。

## babel

> babel对于前端开发来说，目前是不可缺少的一部分。在开发过程中，我们想要使用ES6+的语法，想要使用TypeScript开发React项目，它们都是离不开Babel的。所以，学习Babel对于我们理解代码从编写到线上的转变过程至关重要。Babel是一个工具链，主要用于将ECMAScript 2015+代码转换为向后兼容版本的JavaScript代码。他主要包括语法转换、源代码转换等。
>
> ```js
> [1, 2, 3].map((n) => n + 1);
> <=====ES6 ~ ES5====>
> [1, 2, 3].map(function(n) {
>     return n + 1;
> });
> ```

babel本身可以作为一个独立的工具（和postcss一样），如果想要命令行尝试使用babel，需要安装以下库：

- @babel/core：babel的核心代码，必须安装；
- @babel/cli：可以让我们在命令行使用babel；

**使用babel来处理我们的源代码：**

- source-file/dir：要进行转换的源文件，或者目录；
- --out-file：指定要输出的文件路径及名称；
- --out-dir：指定要输出到的文件夹；

```shell
$ npx babel demo.js --out-file dist/output.js

$ npx babel demo.js --out-dir dist
```

**注意：**--out-file与--out-dir参数不能同时使用。如果使用的是--out-dir参数，那么目标文件默认与源文件的名称相同。

### babel插件

> 这时我们使用babel转换代码后，会发现转换前和转换后一个样，这是因为babel只有解决js代码的能力。如果想要实现代码转换的功能必须要借助于插件（与PostCSS类似）：

比如要转换箭头函数，那么就可以使用箭头函数转换相关的插件：

```shell
// 安装插件
$ npm install @babel/plugin-transform-arrow-functions -D

// 使用插件进行编译转换
$ npx babel src --out-dir dist --plugins=@babel/plugin-transform-arrow-functions
```

再比如要转换const，let等块级作用域：

```shell
// 安装插件
$ npm install @babel/plugin-transform-block-scoping -D

// 使用插件进行编译转换
$ npx babel src --out-dir dist --plugins=@babel/plugin-transform-block-scoping,@babel/plugin-transform-arrow-functions
```

### babel预设

> 如果我们一个个去安装使用插件，那么需要手动来管理大量的babel插件，我们可以直接使用预设preset，预设中包含了大量已经设定的插件来供我们直接使用。比如常见的预设有三个：env、react、TypeScript

**安装@babel/preset-env预设：**

```shell
$ npm install @babel/preset-env -D
```

**使用预设进行编译：**

```shell
$ npx babel src --out-dir dist --presets=@babel/preset-env
```

### babel-loader

> 在实际开发中，我们通常会在构建工具中通过配置babel来对其进行使用，比如在webpack中：

**首先需要去安装相关的依赖：**

```shell
$ npm install babel-loader @babel/core
```

**在webpack.config.js中进行配置：**

- 使用指定插件

```js
{
  test: /\.js$/,
  use: {
    loader: "babel-loader",
    options: {
       plugins: [
         "@babel/plugin-transform-arrow-functions",
         "@babel/plugin-transform-block-scoping",
      ]
    }
  }
}
```

- 使用预设

```js
{
  test: /\.js$/,
  use: {
    loader: "babel-loader",
    options: {
      presets: [
        "@babel/preset-env"
      ]
    }
  }
}
```

**使用babel.config.js配置：**

与PostCSS相同，Babel可以将配置信息单独放到配置文件中：

```js
module.exports = {
  presets: [
    "@babel/preset-env"
  ]
}
```

babel在运行时会自动在这个文件中读取配置。这样不管是用命令行还是使用babel-loader调用babel时，babel都会自动读取这个配置文件:

- 命令行调用：

```shell
$ npx babel demo.js --out-dir dist
```

- babel-loader调用：

```js
{
  test: /\.js$/,
  loader: "babel-loader"
}
```

### babel-plugin-import

> 平时在使用 antd、element 等组件库的时候，都会使用到一个 Babel 插件：babel-plugin-import。antd 和 element 都是通过 ES6 Module 的 export 来导出带有命名的各个组件（类似 antd 的组件库提供了 ES Module 的构建产物，直接通过 import {} from 的形式也可以 tree-shaking）。
>
> 所以，我们可以通过 ES6 的 import { } from 的语法来导入单组件的 JS 文件。但是，我们还需要手动引入组件的样式。如果仅仅是只需要一个 Button 组件，却把所有的样式都引入了，这明显是不合理的。当然也可以只使用单个组件的样式，这样可以减少代码体积，但是有一个不好的地方就是代码不够优雅，那就是使用多少的组件，就需要引用多少个单独的样式，而babel-plugin-import 就是解决了上面的问题，为组件库实现单组件按需加载并且自动引入其样式。如：

```js
import { Button } from 'antd';
  ↓ ↓ ↓ ↓ ↓ ↓
var _button = require('antd/lib/button');
require('antd/lib/button/style');
```

**配置bebel-plugin-import：**

```js
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  plugins: [['import', { libraryName: 'vant', libraryDirectory: 'es', style: true }, 'vant']]
}
```

