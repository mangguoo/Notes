#### CSR

> CSR(Client Side Rendering)：是一种目前流行的渲染方式，它依赖的是运行在客户端的JS，用户首次发送请求只能得到小部分的指引性HTML代码。第二次请求将会请求更多包含HTML字符串的JS文件。

![1c4b6f901c61a8cc517b11fe4b169a5e](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fsdafsdfasdf.jpg)

- ##### 客户端：

  ``` js
  <form target="_self">
    <input type="text" name="user">
    <button type="submit">提交</button>
  </form>
  <div></div>
  
  <script>
    let form, input, div;
    init();
    function init() {
      form = document.querySelector("form")
      input = document.querySelector("input")
      div = document.querySelector("div")
      form.addEventListener("submit", submitHandler)
    }
  
    function submitHandler(e) {
      e.preventDefault()
      ajax("user=" + input.value)
    }
  
    function ajax(param) {
      const xhr = new XMLHttpRequest()
      xhr.addEventListener("load", loadHandler)
      xhr.open("GET", "http://10.9.46.180:4000?" + param)
      xhr.send()
    }
  
    function loadHandler(e) {
      console.log(this);
      div.innerHTML = this.response;
    }
  </script>
  ```

- ##### 服务端：

  ``` js
  const http = require('http')
  http
    .createServer((request, response) => {
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write('<div>你好</div>')
      response.write(`
          <style>
              ul{
                  list-style:none;
                  margin:0;
                  padding:0;
              }
          </style>
      `)
      response.write('<ul>')
      for (let i = 0; i < 10; i++) {
        response.write(`<li><a href='#${i}'>${i}</a></li>`)
      }
      response.write('</ul>')
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  ```

#### SSR

> (Server Side Rendering) ：传统的渲染方式，由服务端把渲染的完整的页面吐给客户端。这样减少了一次客户端到服务端的一次http请求，加快相应速度，一般用于首屏的性能优化。

![40b8d2d56ddd96f0b58ce286f7f8fa8c](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/40b8d2d56ddd96f0b58ce286f7f8fa8c.jpg)

- ##### 客户端

  ``` js
  <form action="http://10.9.46.180:4000" target="_self">
    <input type="text" name="user">
    <button type="submit">提交</button>
  </form>
  ```

- ##### 服务端

  ``` js
  const http = require('http')
  http
    .createServer((request, response) => {
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write(`<!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Document</title>
            <style>
                div{
                    width: 600px;
                    height: 400px;
                    background-color: red;
                    color: white;
                }
                ul{
                    list-style: none;
                    margin: 0;
                    padding: 0;
                }
                li{
                    float: left;
                }
            </style>
        </head>
        <body>
            <div>
                <ul>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                </ul>
            </div>
        </body>
        </html>`)
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  ```

#### CSR SSR优缺点

> - load（Onload Event），它代表页面中依赖的所有资源加载完的事件
> - DCL（DOMContentLoaded），DOM解析完毕
> - FP（First Paint），表示渲染出第一个像素点。FP一般在HTML解析完成或者解析一部分时候触发
> - FCP（First Contentful Paint），表示渲染出第一个内容，这里的“内容”可以是文本、图片、canvas
> - FMP（First Meaningful Paint），首次渲染有意义的内容的时间，“有意义”没有一个标准的定义，FMP的计算方法也很复杂
> - LCP（largest contentful Paint），最大内容渲染时间

- CSR 优点：
  - FP最快，客户端体验较好，因为在数据没有更新之前，页面框架和元素是可以再 dom 生成的
  - FP：仅有一个 div 根节点。以 vue 为例，div#app 注册一个空的 div
  - FCP：包含页面的基本框架，但没有数据内容。以 vue 为例，每个 template 的 div 框架，对于 vue 生命周期的 mounted
  - FMP： 包含页面所有元素及数据，以 vue 为例，通过接口更新到页面的数据后完整的页面展示，对应生命周期的 updated
- CSR 缺点：
  - 不利于 SEO-爬虫数据，整体加载完速度慢
- SSR 优点
  - 返回的页面全部是 HTML 结构，包含所有需要呈现的数据，利于搜索引擎或者爬虫的数据抓取。
  - 目前使用的 MVVM架构，大多都是前后端分离，数据都是动态生成，不利于 SEO 优化。而SSR 利于首屏渲染，首屏的加载来自于服务器，不会依赖服务端的接口请求再数据请求。
- SSR 缺点
  - 性能依赖服务器，前端界面开发可操作性不高。
- 两者的不同之处
  - 服务端渲染的优势在于首屏渲染速度快，简单来讲他不需要来回往返于客户端和服务端。但是其性能等众多因素会影响用户体验。比如说，网速，在线活跃人数，服务器的物理位置等等
  - 而客户端和服务端渲染相反，因为多次和服务器的交互导致首屏加载速度慢。但是一旦这都些请求都加载完成之后，用户和页面之间的交互时用户的体验就会好很多

#### 网页优化方案

1. 优化首屏加载，减少白屏时间，提升加载性能

   - 首屏所有图片懒加载，啥时候用啥时候加载
   - 因为引入 css，js 都需要加载时间，所以首屏的 css 要独立存放，所有的 css 和 js 要合压缩，优先加载首屏的 css 和 js，其余不重要的异步加载
   - js 可以用 async 或 defer 属性做到懒加载，js 和 css 都可以使用 lazyLoad 方法

2. 加速或者减少 HTTP 请求损耗

   > - 使用 CDN 加载公共库
   >
   > - 使用强缓存和协商缓存
   > - 使用域名收敛
   > - 小图片可以使用 Base64 代替
   > - 使用 Get 请求代替 Post 请求
   > - 设置 Access-Control-Max-Age 减少预检请求
   > - 页面跳转其他域名或请求其他域名资源的时候使用浏览器的 prefetch 预解析等
   > - 扩大带宽加速网络，减少 HTTP 请求（首屏使用 SSR，减少请求量）

   - CDN：将程序上传到服务器，CDN 会拉取到所有的服务器站点客户访问时会自动选择近的服务器访问，例如 Jquery 库就放到了 CDN 上面，我们使用的时候就不需要自己上传，直接使用人家存好的就可以

   - 强缓存：HTTP 头上有Expires和cache-control。这个是强制设置缓存，设置缓存时长

   - 协商缓存：也是在 HTTP 头上有一个last-modified。会存程序的修改时间，每次请求都会带着，如果修改了就会重新请求

   - 域名收敛：就是将静态资源放在一个域名下减少 DNS 解析的开销，这主要是为了适应移动端的发展需求。通常 DNS 是一个开销较大的操作，而移动端由于网络带宽和实时性、资源等的限制，这些开销对移动端的用户体验是致命的

   - 域名发散是 pc 端为了利用浏览器的多线程并行下载能力，而域名收敛多用与移动端，提高性能，因为 dns 解析是是从后向前迭代解析，如果域名过多性能会下降，增加 DNS 的解析开销

   - DNS 预解析：浏览器会在加载网页时对网页中的域名进行解析缓存，这样在你单击当前网页中的连接时就无需进行 DNS 的解析，减少用户等待时间，提高用户体验

     ``` js
     <link rel="dns-prefetch" href="//g.alicdn.com" />
     <link rel="dns-prefetch" href="//img.alicdn.com" />
     <link rel="dns-prefetch" href="//tce.alicdn.com" />
     ```

   - 图片可以先试用小图片放大，当图片加载完毕在替换，也可以将图片的 Base64 直接放在页面中

   - 使用 Get 代替 Post 请求，因为 Get 请求会自动产生缓存，而且 Get 比 Post 少发送一次请求

3. 延迟加载

   - 非重要的库、非首屏图片延迟加载，SPA 组件懒加载

4. 减少请求内容的体积

   - 开启服务器 Gip 压缩，JS,CSS 问价压缩合并，减少 cookoe 大小，SSR 直接渲染

5. 优化关键渲染路径，就可能减少阻塞渲染的 JS 和 CSS

6. 优化用户体验

   - 白屏使用时间加载进度条，loading 图，骨架屏代替