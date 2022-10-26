# webpack loader

## css-loader

> 如果项目中使用import导入了css文件，对项目打包时，webpack会报错：“模块解析失败，你需要一个合适的loader来处理这个文件类型”。
>
> 这是由于webpack默认只能解析js和json文件。因此遇到css文件他就没办法了，这个时候就需要用到loader了。各种loader增强了webpack处理文件的能力。css-loader可以将css文件也看成是一个模块，让我们可以通过import来加载这个模块。

**安装css-loader：**

```js
npm install css-loader -D
```

**css-loader的使用：**

- 内联方式：

  ```js
  import "css-loader!../css/style.css";
  ```

- 配置方式： 配置方式指在webpack.config.js文件中写明配置信息。module.rules中允许我们配置多个loader。这种方式可以更好的表示loader的配置，也方便后期的维护，同时也让你对各个Loader有一个全局的概览

  ```js
  module: {
    rules: [
      {
        test: /\.css$/, //正则表达式
        loader: "css-loader", // 1.loader的写法(语法糖)
        use: [ // 2.完整的写法（UseEntry）
          {loader: "css-loader"}
        ],
        use: ["css-loader"] // 3.完整的写法（传递字符串）
      }
    ]
  }
  ```

### css module

> CSS Modules 不是官方规范或浏览器中的实现，而是构建步骤中的一个过程（在 Webpack 或 Browserify 的帮助下），它改变了类名和选择器的作用域（即有点像命名空间）。
>
> 下面这个例子中，很显然A，B两组件中导入的css样式有了冲突(.title)，而由于B组件是后导入的，因此他应该是B.css中的.title会覆盖掉A.css中的.title样式。

- **src/index.js**

```js
import componentA from "./js/componentA";
import componentB from "./js/componentB";

document.addEventListener("DOMContentLoaded", function(){
    const app = document.querySelector("#app")
    componentA(app)
    componentB(app)
})
```

- **src/css/A.css**

```css
.title {
    color: red;
}
```

- **src/js/componentA.js**

```js
import "../css/A.css"

export default function($dom) {
    const p = document.createElement("p")
    p.innerHTML = "componentA"
    p.className = "title"
    $dom.appendChild(p)
}
```

- **src/css/B.css**

```css
.title {
    color: blue;
}
```

- **src/js/componentB.js**

```js
import "../css/B.css"

export default function($dom) {
    const p = document.createElement("p")
    p.innerHTML = "componentA"
    p.className = "title"
    $dom.appendChild(p)
}
```

要解决这个问题，就可以使用css module，开启它需要在webpack中进行配置：

```js
{
  test: /\.css$/,
  use: [
    "style-loader",
    {
      loader: "css-loader",
      options: {
        modules: true // 开启css module
      }
    },
    "postcss-loader"
  ]
}
```

配置完成后再重新打包，会发现页面中的css样式的类名全部变成了hash值，这样就会造成新的问题，就是html中引用不到这些样式了，因为他们全部都变成了随机生成的hash值，每次打包后都会变得不一样。而我们在页面中使用的className还是它转换成hash之前的名称(p.className = "title")，因此样式是绝对引用不到的。那该怎么办？要解决这个问题很简单，只要能获取到css-loader生成的这个hash类名，一切问题就迎刃而解了，获取方式如下：

```js
import style from '../css/A.css'

export default function ($dom) {
  const p = document.createElement('p')
  p.innerHTML = 'componentA'
  // 通过类名来获取转换后的hash
  p.className = style.title
  $dom.appendChild(p)
}
```

```js
import style from '../css/B.css'

export default function($dom) {
  const p = document.createElement("p")
  p.innerHTML = "componentA"
  // 通过类名来获取转换后的hash
  p.className = style.title
  $dom.appendChild(p)
}
```

## style-loader

> 我们通过css-loader来加载css文件，但是会发现这个css在我们的代码中并没有生效（页面没有效果）。这是因为css-loader只是帮我们把css文件正确加载并打包，但并不会将打包之后的css插入到页面中。如果希望再完成插入style的操作，那么还需要另外一个loader，就是style-loader。它会帮我们把css样式转换为页内样式。

**安装：**

```js
npm install style-loader -D
```

**注意：**

因为loader的执行顺序是从右向左（或者说从下到上，或者说从后到前的），所以我们需要将style-loader写到css-loader的前面。这样就是先通过css-loader对css样式文件进行转换，再通过style-loader把转换后的样式插入到页面中。

style-loader是把转换后的样式通过页内样式的方式添加进来的，并不是将css抽取到单独的文件中。

## less-loader

> 在我们开发中，我们可能会使用less、sass、stylus的预处理器来编写css样式，这样效率会更高。但是想让环境支持这些预处理器，就必须让less、sass等编写的css通过工具转换成普通的css。
>
> - 下载less编译工具：
>
>   ```js
>npm install less -D
>   ```
>
> - 编译less为css：
> 
> ```js
>npx lessc ./src/css/title.less title.css
> ```

但是在项目中我们会编写大量的less，为了避免手动转换，就可以使用less-loader，来自动使用less工具转换less到css：

- 安装less-loader

```js
npm install less-loader 
```


- 配置webpack.config.js

```js
// less-loader会先把less文件转换成css文件，然后css-loader会对css进行加载并打包，最后style-loader会把打包起来的样式通过页面样式的方式的方式插入进来。
{
  test: /\.less$/,
  use: [
    "style-loader",
    "css-loader",
    "less-loader"
  ]
}
```

## scss-loader

> 我们可以使用node-sass工具来完成它的编译转换：
>
> - 下载编译工具：
>
> ```js
> npm install node-sass -D
> ```
>
> - 执行编译：
>
> ```js
> npx node-sass a.scss>a.css
> ```

- 安装sass-loader

```js
npm install sass-loader
```

- 配置webpack.config.js

```js
// sass-loader会先把sass文件转换成css文件，然后css-loader会对css进行加载并打包，最后style-loader会把打包起来的样式通过页面样式的方式的方式插入进来。
{
  test: /\.sass$/,
  use: [
    "style-loader",
    "css-loader",
    "sass-loader"
  ]
}
```

## file-loader

> 要处理jpg、png等格式的图片，我们也需要有对应的loader：file-loader。
> file-loader的作用就是帮助我们处理 import/require 方式引入的文件资源，或是通过css规则引入的文件资源(如背景图片和字体)，并且会将它放到输出文件夹中。
>
> 注意：css-loader 6.0.0以上版本。对引入背景图片的url解析方式不一样，会导致生成了两个图片（一个正常由file-loader解析生成，一个仅由css-loader解析引入）。
>
> 这个情况下webpack会引用css-loader打包的图片文件，但是css-loader打包生成的图片文件是不可用的，这就会导致页面中图片不可见。解决方法就是把css-loader版本由6降为5，或者直接使用webpack5的asset module type。

**安装:**

```js
npm install file-loader -D
```

**配置处理图片的Rule：**

```js
{
  test: /\.(jpe?g|png|gif|svg)$/,
  use: {
    loader: "file-loader",
    options: {
      // outputPath: "img", // 表示存放路径
      // 直接写path/name，这样就可以省去outputPath属性
      name: "img/[name]_[hash:6].[ext]",
    }
  }
}
```

## url-loader

> url-loader和file-loader的工作方式是相似的，都是用于打包文件资源。但是与file-loader不同的是，url-loader可以把图片转成base64的URI。

**安装：**

```js
npm install url-loader -D
```

**配置处理图片的Rule：**

> url-loader有一个options属性limit，可以用于设置转换的限制。当图片小于所设置的限制时，图片才会转换成base-64编码的uri，而大于所设置的限制时，会直接使用图片。

```js
{
  test: /\.(jpe?g|png|gif|svg)$/,
  use: {
    loader: "url-loader",
    options: {
      name: "img/[name]_[hash:6].[ext]",
      limit: 100 * 1024
    }
  }
}
```

## asset module type

> 在webpack5之前，加载一些文件资源时需要使用一些loader，比如raw-loader 、url-loader、file-loader。但是在webpack5开始，我们可以直接使用资源模块类型（asset module type），来替代上面的这些loader。
>
> 资源模块类型通过添加 4 种新的模块类型，来替换所有这些 loader：
>
> - asset/resource 发送一个单独的文件并导出 URL。之前通过使用 file-loader 实现；
> - asset/inline 导出一个资源的 data URI。之前通过使用 url-loader 实现；
> - asset/source 导出资源的源代码。之前通过使用 raw-loader 实现；
> - asset 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用url-loader，并且配置资源体积限制实现；

### 代替file-loader

```js
{
  test: /\.(jpe?g|png|gif|svg)$/,
  type: "asset/resource",
  generator: {
    filename: "img/[name]_[hash:6][ext]"
  }
}
```

### 代替url-loader

```js
{
  test: /\.(jpe?g|png|gif|svg)$/,
  type: "asset",
  generator: {
    filename: "img/[name]_[hash:6][ext]"
  },
  parser: {
    dataUrlCondition: {
      maxSize: 100 * 1024
    }
  }
}
```

### 打包字体文件

> 如果我们需要使用某些特殊的字体或者字体图标，那么我们会引入很多字体相关的文件，这些文件的处理和图片是类似的。
>
> - 首先，我从阿里图标库中下载了几个字体图标：
> - 然后在在component中引入，并且添加一个i元素用于显示字体图标;

```js
import "./font/iconfont.css"

const iEl = document.createElement('i');
iEl.className = "iconfont icon-icons06";
document.body.appendChild(iEl)
```

这个时候打包会报错，因为webpack无法正确的处理eot、ttf、woff等文件。我们可以选择使用file-loader来处理这些文件，也可以选择直接使用webpack5的资源模块类型来处理。

**使用file-loader解决：**

```js
{
  test: /\.(eot|ttf|woff2?)$/,
  use: {
    loader: "file-loader",
    options: {
      name: "font/[name]_[hash:6].[ext]"
    }
  }
}
```

**使用asset module type解决：**

```js
{
  test: /\.(eot|ttf|woff2?)$/,
  type: "asset/resource",
  generator: {
    filename: "font/[name]_[hash:6][ext]"
  }
}
```

