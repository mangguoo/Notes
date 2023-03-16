# publicPath

> 在阶段开始一般使用`根(/)`来访问自己的页面或者接口，但到了部署阶段，往往就会面临变更部署路径的问题。因为使用 `/` 就意味要独占一个域，生产环境中，单个应用通常不可能独占一个域，更常见的是前后端共用一个域：
>
> ```cpp
> http://example.com/xxx --> appA
> http://example.com/yyy --> appB
> http://example.com/zzz --> appC
> ```

## 前端修改部署路径

> 场景：原先开发阶段前端部署在 `/`, 我们可以通过 [http://127.0.0.1:3000/index.html](https://links.jianshu.com/go?to=http%3A%2F%2F127.0.0.1%3A3000%2Findex.html) 访问，部署阶段我们要把应用部署在一个子目录中, 使其能通过 [http://example.com/appA/index.html](https://links.jianshu.com/go?to=http%3A%2F%2Fexample.com%2FappA%2Findex.html) 访问

这时前端要关注两个问题：

1. 资源引用路径（publicPath）
2. router base（basename）

### publicPath

我们的应用一般是被部署在一个域名的根路径上，例如 `https://www.my-app.com/`。如果应用被部署在一个子路径上，就需要用这个选项指定这个子路径。例如，如果你的应用被部署在 `https://www.my-app.com/my-app/`，则设置 `publicPath` 为 `/my-app/`

这样webpack在打包的时候，就会把我们通过`/**`引用的资源编译为向`/my-app/**`进行访问，这样就不会出现在生产环境中资源访问不到的情况

这个值也可以被设置为空字符串 (`''`) 或是相对路径 (`'./'`)，这样所有的资源都会被链接为相对路径，这样打出来的包可以被部署在任意路径，也可以用在类似 Cordova hybrid 应用的文件系统中

> 相对路径的 `publicPath` 有一些使用上的限制。在以下情况下，应当避免使用相对 `publicPath`:
>
> - 当使用基于 HTML5 `history.pushState` 的路由时；
> - 当使用 `pages` 选项构建多页面应用时。

这个值在开发环境下同样生效。如果想把开发服务器架设在根路径，你可以使用一个条件式的值：

```js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production'
    ? '/production-sub-path/'
    : '/'
}
```

### basename

我们可以通过配置createBrowserRouter的basename选项，来达到publicPath和router base统一的目的

```js
const router = createBrowserRouter(
  routes
  ,{
    basename: '/node'
  }
)
```



