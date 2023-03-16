# webpack plugin

> Loader是用于特定的模块类型进行转换。Plugin可以用于执行更加广泛的任务，比如打包优化、资源管理、环境变量注入等。

## CleanWebpackPlugin

> 每次修改了项目，重新打包项目时，都需要手动删除dist文件夹。我们可以借助于一个插件来帮助我们完成这个工作，这个插件就是CleanWebpackPlugin。配置完成后，再重新打包时，该插件就会先把上次打包生成的文件删除。

**安装：**

```shell
$ npm install clean-webpack-plugin -D
```

**配置webpack：**

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
  // ....
  plugins: [new CleanWebpackPlugin()]
}
```

## HtmlWebpackPlugin

> 使用了HtmlWebpackPlugin再进行打包时，该插件就会在我们配置的output.path目录下生成一个index.html文件，并且这个文件中会引用打包生成的js文件。
>
> 默认情况下该index.html是根据ejs的一个模块来生成的，在html-webpack-plugin的源码中，有一个default-index.ejs模块。
>
> - 我们的HTML文件是编写在根目录下的，而最终打包的dist文件夹中是没有index.html文件的；
> - 在进行项目部署的时，也必须要有对应的入口文件index.html；
> - 所以我们也需要对index.html进行打包处理；

**安装：**

```shell
$ npm install html-webpack-plugin -D
```

**配置webpack：**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  // ....
  plugins: [new HtmlWebpackPlugin()]
}
```

### 自定义html模板：

> 如果我们想在自己的模块中加入一些比较特别的内容。比如添加一个noscript标签，在用户的JavaScript被关闭时，给予响应的提示。再比如在开发vue或者react项目时，我们需要一个可以挂载后续组件的根标签 <div id="app"></div>。想要实现这样的功能的话就需要一个属于自己的index.html模块(最好放在public文件夹)：

```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work 
      properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

上面的代码中，会有一些类似这样的语法<% 变量 %>，这个是EJS模块填充数据的方式。在配置HtmlWebpackPlugin时，我们可以添加如下配置：

- template：指定我们要使用的模块所在的路径；
- title：在进行htmlWebpackPlugin.options.title读取时，就会读到该信息；

```js
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
  // ......
  plugins: [
    new HtmlWebpackPlugin({
      title: "webpack项目",
      template: "./public/index.html"
    })
  ]
}
```

在上面的ejs模板中，用到了两种模板数据填充的语法：

- `<%= htmlWebpackPlugin.options.title %>`  (<% 变量 %>)
- `<%= BASE_URL %>`  (<% 常量 %>)

前者的值是通过对插件的配置传入的，而对于后者的传值，就需要用到DefinePlugin插件。这个插件允许在编译时创建配置的全局常量，是一个webpack内置的插件(不需要单独安装)。使用方法如下：

```js
const { DefinePlugin } = require('webpack')

module.exports = {
  // ...
  plugins: [
    new DefinePlugin({
      BASE_URL: "'./'"
    })
  ]
}
```

注意这里对常量BASE_URL的定义：

```js
BASE_URL: "'./'"
```

这里使用了两重引号，这是由于如果只用一层引号的话，DefinePlugin会把他当做一个变量来解析。如果不想被当做变量来解析，而是想直接写入一个字符串的话，就要嵌套两层引号。

`<link rel="icon" href="<%= BASE_URL %>favicon.ico">`最后会被解析为`<link rel="icon" href="./favicon.ico">`。
由于我们一般把favicon.ico文件放在webpack-test/public文件夹下，因此想要让打包后的index.html能够引用到这个图标文件的话，还需要通过CopyWebpackPlugin插件把favicon.icon文件复制到与index.html同级目录下(output.path目录下)。

## CopyWebpackPlugin

> 在项目开发中，我们会将一些文件放到public目录下。但是在打包过程中，希望把这里面的文件也复制进打包文件目录下。这个复制文件的功能，我们可以使用CopyWebpackPlugin来完成

**安装CopyWebpackPlugin插件：**

```js
npm install copy-webpack-plugin -D
```

**配置CopyWebpackPlugin：**

> - from：设置从哪一个源中开始复制
> - to：复制到的位置，这个位置会与output.path: "build" 进行拼接。比如to: "public"，那么实际复制到的位置就是build/public目录下。如果不指定这个参数的话，那就会默认复制到output.path目录下
>
>    +  globOptions：设置一些额外的选项，其中可以编写需要忽略的文件
>
> 这里示例中 ”**/index.html“ 表示from目录及其子目录下所有的index.html文件。这就避免了from的子目录下还有其他index.html文件，从而误复制的情况。
>
> index.html不需要复制，因为我们已经通过HtmlWebpackPlugin完成了index.html的生成。如果是mac操作系统的话，还需要ban掉.DS_Store文件，因为mac会在每个目录下自动生成一个这样的文件。

```js
const { copyFile } = require("fs")

module.exports = {
  // ......
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          from: "public",
          to: "./",
          globOptions: {
            ignore: [
              '**/index.html'
            ]
          }
        }
      ]
    })
  ]
}
```

## MiniCssExtractPlugin

> 使用MiniCssExtractPlugin插件后重新打包，webpack把css打包成了独立文件，并自动把对css文件的引用(link标签)插入到了html模板中。

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const devMode = process.env.NODE_ENV !== 'production'
 
module.exports = {
  module: {
    rules: [
      {
        test: /\.(sa|sc|c)ss$/,  // 可以打包后缀为sass/scss/css的文件
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              // 默认使用 webpackOptions.output中的publicPath
              // 这个配置为为css内部的图像、文件等外部资源指定自定义公共路径，如果打包后，
              // background属性中的图片显示不出来，请检查publicPath的配置是否有误
              publicPath: './',  
              // publicPath: devMode ? './' : '../',   // 根据不同环境指定不同的publicPath
            },
          },
          'css-loader',
          'sass-loader'
        ],
      },
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      // 此选项确定每个输出 CSS 文件的名称
      filename: devMode ? 'css/[name].css' : 'css/[name].[hash].css',
      // 此选项确定非入口chunk文件的名称
      chunkFilename: devMode ? 'css/[id].css' : 'css/[id].[hash].css',
    })
  ]
}
```

