# NextJS

## 创建Next.js项目

```bash
npm install create-next-app -g
create-next-app next-app
npm run dev
```

Next.js官方脚手架初始目录结构如下：

```text
├─ /.next              <-- 用于SSR运行的工程，执行npm run dev或npm run build后才会出现
├─ /node_modules
├─ /pages              <-- Next.js指定的页面目录
|  ├─ /api             <-- Next.js指定的API服务目录，可以删除
|  |  └─ hello.js      <-- API服务的hello接口
|  ├─ _app.js          <-- Next.js指定的项目入口文件
|  └─ index.js         <-- 项目首页（会生成index.html）
├─ /public             <-- 静态目录，放在这里的文件可通过"/"直接访问（没有public这一层级）
|  ├─ favicon.ico
|  └─ vercel.svg
├─ /styles             <-- 项目全局样式
|  ├─ globals.css
|  └─ Home.module.css  <-- Home组件样式
├─ .eslintrc.json
├─ .gitignore
├─ next.config.js      <-- Next.js配置文件
├─ package.json
├─ README.md
└─ yarn.lock
```

删掉一些无用的文件，并进行一些调整整合，目录结构就会变成这样:

```txt
├─ /.next           <-- 用于SSR运行的工程
├─ /node_modules
├─ /public          <-- 静态目录
|  ├─ favicon.ico
|  └─ vercel.svg
├─ /common          <-- 公用目录
|  ├─ /images       <-- 公用图片目录
|  └─ /styles       <-- 公用样式目录
├─ /components      <-- 公用组件目录
├─ /pages           <-- Next.js指定的页面目录
|  ├─ /api          <-- Next.js指定的API服务目录（不会生成api页面目录）
|  ├─ _app.js       <-- Next.js指定的项目入口文件（不会生成_app.html）
|  └─ index.js      <-- index页面（会生成index.html）
├─ .eslintrc.json
├─ .gitignore
├─ next.config.js    <-- Next.js配置文件
├─ package.json
├─ README.md
└─ yarn.lock
```

Next.js的官方脚手架虽然没有src目录，但考虑到src目录是普遍存在于大多数脚手架工程中，所以Next.js也对src目录做了支持。当然，如果新建的不是src目录，把pages、styles放进去是无法被正确识别的。**关于src目录，官方的规则如下**：

1. 如果根目录下有pages，则src/pages将被忽略
2. public目录以及next.config.js、jsconfig.json、tsconfig.json不能放到src目录里

## next项目配置

### 设置路径别名

> 自从Next.js 9.4以来，Next.js自动支持tsconfig.json和jsconfig.json中的“`path`”和“`baseUrl`”选项

`baseUrl`配置选项用于指定项目根目录，配置后我们则可以直接从项目的根目录中导入

```json
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": "."
  }
}
```

```json
// components/button.js
export default function Button() {
  return <button>Click me</button>
}
```

```jsx
// pages/index.js
import Button from 'components/button'

export default function HomePage() {
  return (
    <>
      <h1>Hello World</h1>
      <Button />
    </>
  )
}
```

`paths`配置别名后可以方便我们引用某一个特殊路径

```json
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"]
    }
  }
}
```

```jsx
// components/button.js
export default function Button() {
  return <button>Click me</button>
}
```

```json
// pages/index.js
import Button from '@/components/button'

export default function HomePage() {
  return (
    <>
      <h1>Hello World</h1>
      <Button />
    </>
  )
}
```

### 配置SourceMap

development环境是开启sourceMap的，production环境默认不开启sourceMap。如果需要在production环境开启sourceMap，在next.config.js进行以下配置：

```js
module.exports = {
  productionBrowserSourceMaps: true,
}
```

### 设置页面title

设置页面的title很简单，只需要把项目入口文件的title加上就可以了。修改src/pages/_app.js：

```js
import Head from 'next/head'

function MyApp({ Component, pageProps }) {
    return (
        <>
           <Head>
              <title>My Next App</title>
           </Head>
           <Component {...pageProps} />
        </>
     )
}
    
export default MyApp
```

### 设置HTML框架代码

与传统的react，vue脚手架不同，next项目的public文件夹中没有index.html模板文件，如果想要自定义模板，则需要新建src/pages/_document.js，代码如下：

```js
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html>
       <Head>
          <link rel="icon" href="/favicon.ico" />
          <meta name="description" content="Next.js演示项目" />
       </Head>
       <body>
          <Main />
          <NextScript />
       </body>
    </Html>
  )
}
```

\_document.js也是Next.js的指定文件名，且必须在pages目录下才可生效。那既然\_document.js可以设置<head>的内容，那为什么<title>却在_app.js中设置呢？

这是因为\_document.js只会在初始时进行预渲染，因此可能会导致标题无法正常显示。因此官方不建议把<title>放到\_document.js中。如果你在\_document.js中的<head>里设置了<title>，在build的时候会收到warning。

> 注意：在项目启动后，打开浏览器devTool，会发现_document.js文件中设置的模板并没有生效，这是因为这些内容只有在build后才会生效，dev模式中是看不到刚刚设置的内容的

### 以SSR模式运行项目

执行`npm run build`，build项目。执行后，在项目根目录下会生成一个```.next```的目录。这个目录就是用于运行SSR的代码，仅能运行在服务端，不能被浏览器直接运行

然后再执行`npm run start`，以SSR模式运行项目。但是要注意的是，每次代码更新，在执行`npm run start`之前，一定要先执行`npm run build`。否则运行的并不是最新build的项目

现在打开**http://localhost:3000**，看到是SSR模式运行的项目。打开调试工具，看到_document.js设置的代码已生效

yarn start默认是运行在3000端口，如果想运行在其它端口，可以执行以下命令：

```bash
npm run start -p 4000
```

## css预处理及使用

### 使用scss

在Next.js中集成Sass/Scss是非常容易的事。因为Next.js本身就对Sass/Scss做了友好支持，只需要执行以下命令，安装sass：

```bash
npm install sass --dev
```

安装后，即可正常使用Sass/Scss。受益于Next.js对Sass/Scss的友好支持，强烈建议在Next.js使用Sass/Scss

### 样式命名规范

合理的样式命名规范对项目开发有很大的帮助，主要体现在以下方面：

1. 避免因样式名重复导致的污染

2. 从命名上可直观区分“组件样式”、“页面样式”（用于给在此页面的组件样式做定制调整）、“全局样式”

3. 快速定位模块，便于查找问题

> 命名规范示例：
>
> - **G-xx：** 表示全局样式，用来定义公用样式
>
> - **P-xx:** 表示页面样式，用来设置页面的背景色、尺寸、定制化调整在此页面的组件样式
>
> - **M-xx:** 表示组件样式，专注组件本身样式

### **配置全局样式**

- common/styles/reset.css

```css
    ...（略）
```

- common/styles/global.scss

```scss
$bgColor: #bae7ff;
body {    
    background: $bgColor;
}
```

- common/styles/frame.scss

```scss
@import './reset.css';
@import './global.scss';
```

- pages/_app.js

```jsx
import Head from 'next/head'
import '@/common/styles/frame.scss'
    ...（略）
```

### **配置页面（pages）样式**

> 要特别注意的一点，在Next.js项目中，组件的代码中是无法引入全局样式的。对于组件自身的样式，只能使用CSS Module来引用，即文件名为"xxx.module.scss"。否则会报错如下：`Global CSS cannot be imported from files other than your Custom <App>. Due to the Global nature of stylesheets, and to avoid conflicts, Please move all first-party global CSS imports to pages/_app.js. Or convert the import to Component-Level CSS (CSS Modules).`

- **pages/index.module.scss：**

```scss
.P-index {
    margin: 50px auto 0;
    width: 800px;
    height: 400px;
    border: solid 1px #999;
    background: #fff;
    h1 {
        margin-top: 30px;
        text-align: center;
        font-size: 30px;
        font-weight: bold;
    }
    .img-wrap, .pic{
        display: block;
        margin: 20px auto 0;
        width: 60px;
        height: 60px;
    }
}
```

- **pages/index.js**

```jsx
import pageStyles from './index.module.scss'
function Index() {
    return (
        <div className={pageStyles['P-index']}>
            <h1>This is Index Page.</h1>
        </div>
    )
}

export default Index
```

由于使用CSS Module的方式加载样式，因此在生成的HTML中，className会自动加上随机字符串后缀，对应的css也会自动添加相应的字符串。正因为CSS Module这个机制，同名样式互相污染的问题也就不存在了

### 配置webpack

```js
module.exports = {
  webpack: (
    config,
    { buildId, dev, isServer, defaultLoaders, nextRuntime, webpack }
  ) => {
    config.module.rules.push({
      test: /\.mdx/,
      use: [
        options.defaultLoaders.babel,
        {
          loader: '@mdx-js/loader',
          options: pluginOptions.options,
        },
      ],
    })
    return config
  },
}
```

### 配置代理

> rewrite选项允许我们将传入请求路径映射到不同的目标路径。rewrite可以用作URL代理，并且可以屏蔽目标路径，看起来好像没有更改url。相反的，redirect将会把路由重定向到另一个路由，并显示 URL 更改。要使用rewrite，可以使用next.config.js中的rewrite key，rewrite允许我们将传入请求路径映射到不同的目标路径
>

```js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/about',
        destination: '/',
      },
    ]
  },
}
```

## 页面路由

### **优化index路由**

> 虽然index页面和对应的样式都正常显示了，但随着页面数量的增加，pages下的页面js文件和scss样式文件将会非常多，混杂在一起看上去不美观，也不利于高效维护。如果能够把每个页面的js和scss单独放在一个目录里，就会非常清晰了
>
> 在src/pages下新建index目录，把src/pages下的index.js和index.module.scss放到index目录下。调整后的pages目录结构如下：

```txt
/pages
├─ /api
├─ /index
|  ├─ index.js
|  └─ index.module.scss
├─ _app.js
├─ _document.js
└─ 404.js
```

这样调整后，http://localhost:3000页面会变成404。这是因为新建了index这一级目录，导致页面的访问路径也多了一级。index页面的地址变为了http://localhost:3000/index。但这并不是我们想要的。我们还是希望能够通过http://localhost:3000访问index页面

**方法一：通过next.config.js配置**

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
    reactStrictMode: true,
    swcMinify: true,
    exportPathMap: async function (
        defaultPathMap,
        { dev, dir, outDir, distDir, buildId }
     ) {
        return {
            '/': { page: '/index' },
        }
    },
}

module.exports = nextConfig
```

经过这样的配置，重启项目后，在dev模式下，http://localhost:3000可以正常访问index页面了。并且，http://localhost:3000/index也可以访问index页面

> **注意：**
>
> 1. 修改next.config.js需要重启项目才能生效
>
> 2. 以上方式仅在dev和SSG模式下生效。执行npm run start的SSR模式不生效

**方法二：通过组件引入**

了同时兼容dev、SSR、SSG模式，可以不依赖于next.config.js。直接采用最朴素的方式，新建一个页面并引入这个“首页”即可

- pages/index.js

```jsx
import Home from '@/pages/home'
function Index() {
    return <Home />
}
export default Index
```

- 将pages/index目录更名为home

```txt
/pages
├─ /api
├─ /home
|  ├─ index.js
|  └─ index.module.scss
├─ _app.js
├─ _document.js
├─ 404.js
└─ index.js
```

经过以上调整后，在dev、SSR、SSG模式下访问http://localhost:3000或者http://localhost:3000/home都是home页面，完美解决

### 主页路由

- `pages/index.js` → `/`
- `pages/blog/index.js` → `/blog`

### 嵌套路由

- `pages/blog/first-post.js` → `/blog/first-post`
- `pages/dashboard/settings/username.js` → `/dashboard/settings/username`

### 动态路由

- `pages/blog/[slug].js` → `/blog/:slug` (`/blog/hello-world`)
- `pages/[username]/settings.js` → `/:username/settings` (`/foo/settings`)
- `pages/post/[...all].js` → `/post/*` (`/post/2020/id/title`)

**获取动态路由参数:**

动态路由参数由useRouer这个hook来获取

```jsx
import { useRouter } from 'next/router'

const Post = () => {
  const router = useRouter()
  const { pid } = router.query

  return <p>Post: {pid}</p>
}

export default Post
```

使用Link组件跳转到动态路由页面时，它的动态路由参数就会被捕获

```jsx
import Link from 'next/link'

function Home() {
  return (
    <ul>
      <li>
        {/* { "pid": "abc" } */}
        <Link href="/post/abc"> 
          <a>Go to pages/post/[pid].js</a>
        </Link>
      </li>
      <li>
        {/* { "foo": "bar", "pid": "abc" } */}
        <Link href="/post/abc?foo=bar"> 
          <a>Also goes to pages/post/[pid].js</a>
        </Link>
      </li>
      <li>
        {/* { "pid": "abc", "comment": "a-comment" } */}
        <Link href="/post/abc/a-comment">
          <a>Go to pages/post/[pid]/[comment].js</a>
        </Link>
      </li>
    </ul>
  )
}

export default Home
```

动态路由参数可以存在多个，如果不确定动态路由参数有多少个，则可以使用`[...params]`语法

`pages/post/[...slug].js`可以匹配`/post/a`, `/post/a/b`, `/post/a/b/c`...

```json
/post/a => { "slug": ["a"] }
```

```json
/post/a/b => { "slug": ["a", "b"] }
```

如果写成双中括号语法`[[...slug]]`，则表示参数全部是可选的。例如`pages/post/[[...slug]].js`将匹配 `/post`, `/post/a`, `/post/a/b`

```js
{ } // GET `/post` (empty object)
{ "slug": ["a"] } // `GET /post/a` (single-element array)
{ "slug": ["a", "b"] } // `GET /post/a/b` (multi-element array)
```

### 错误路由

**404页面**

> 404页面可能会经常被访问。为每次访问呈现一个错误页面会增加 Next.js 服务器的负载
>
> 为了避免上述缺陷，Next.js 默认提供了一个静态404页面，而不需要添加任何其他文件

- pages/404.js

```jsx
export default function Custom404() {
  return <h1>404 - Page Not Found</h1>
}
```

404页面，在dev和SSR模式下，如果访问一个不存在的地址，是可以自动显示404页面的

但在SSG模式下，404页面被静态化成了404.html，访问一个不存在的地址并不会自动跳转至404.html。需要结合Apache或者Nginx的配置来实现

**500页面**

> 服务器呈现每次访问的错误页面会增加响应错误的复杂性。为了帮助用户尽快得到对错误的响应，Next.js 默认提供了一个静态500页面，而不需要添加任何其他文件

- pages/500.js

```jsx
export default function Custom500() {
  return <h1>500 - Server-side error occurred</h1>
}
```

**定制错误页面**

> 在next中，错误由 Error 组件在客户端和服务器端处理。如果希望覆盖它，那么定义文件pages/_error.js并添加以下代码:

- pages/_error.js

```jsx
function Error({ statusCode }) {
  return (
    <p>
      {statusCode
        ? `An error ${statusCode} occurred on server`
        : 'An error occurred on client'}
    </p>
  )
}

Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404
  return { statusCode }
}

export default Error
```

### **next/router和next/link**

> Next.js项目内的页面跳转使用<Link>，不要使用<a>，如果是项目外部链接，则使用<a>
>
> 由于导航栏可能在多个页面中都会使用，因此定义为全局组件

- **/components/nav/index.js**

```jsx
import { useRouter } from 'next/router'
import Link from 'next/link'
import styles from './nav.module.scss'

function Nav() {
  const router = useRouter()
  // 获取当前页面的pathname
  const pathname = router.pathname
  
  const changeRoute = (newLocale: string) => {
    const { pathname, asPath, query } = router
    router.push({ pathname, query }, asPath, { locale: newLocale })
  }
  
  return (
    <div className={styles['M-nav']}>
      /* 声明式导航 */
      <Link href='/'>
        <div className={styles['tab'] + (pathname === '/' ? ' ' + styles['current'] : '')}>Home</div>
      </Link>
      <Link href='/about'>
        <div className={styles['tab'] + (pathname === '/about' ? ' ' + styles['current'] : '')}>About</div>
      </Link>
      /* 编程式导航，切换语言 */
      <button onClick={() => changeRoute("zh-cn")}>切换为中文</button>
    </div>
  )
}
export default Nav
```

- **/components/nav/nav.module.scss**

```scss
.M-nav {
  display: flex;
  background: #e6f7ff;
  .tab {
    flex: 1;
    height: 80px;
    line-height: 80px;
    font-size: 24px;
    text-align: center;
    cursor: pointer;
    &.current {
      background: #1890ff;
      color: #fff;
    }
  }
}
```

**链接到动态路径，例如：`pages/blog/[slug].js`**

```jsx
<Link href={`/blog/${encodeURIComponent(post.slug)}`}>
  <a>{post.title}</a>
</Link>

<Link
  href={{
    pathname: '/blog/[slug]',
    query: { slug: post.slug },
  }}
>
  <a>{post.title}</a>
</Link>
```

## 图片引用

### **使用原生<img>标签引入图片**

- pages/home/index.js

```jsx
import imgLogo from '@/common/images/app.png'

function Index() {
  return (
    <div>
      <img src={imgLogo.src} alt='' />
    </div>
  )
}

export default Index
```

注意：使用<img>引入项目内图片时，与Create-React-App略有不同，需要加.src：

```jsx
// Create-React-App项目使用方式
<img src={imgLogo} alt="" />
// Next.js项目使用方式
<img src={imgLogo.src} alt="" />
```

但是，使用<img>，在执行yarn build静态化的时候，会报warning：

> Warning: Do not use `<img>` element. Use `<Image />` from `next/image` instead. See: https://nextjs.org/docs/messages/no-img-element @next/next/no-img-element

### **使用next/image引用图片**

> next自带的<Image>可以认为是<img>的升级版，提供了非常方便的尺寸适配、加载等属性。但也会自动增加很多样式，会影响原生的<img>样式。所以要根据情况选用。如果只是简单引入图片，无需使用<Image>
>
> 建议为<Image>包裹一个父容器，并为父容器定义样式。<Image>会自动适配父容器的大小，因此可以不用为<Image>特意设置width和height。如果直接使用<Image>，其自动生成的<sapn>容器自带很多样式，很容易导致页面样式错乱

- pages/home/index.js

```jsx
import Image from 'next/image'
import imgLogo from '@/common/images/app.png'

function Index() {
  return (
    <div>
      <Image src={imgLogo} alt='' />
    </div>
  )
}

export default Index
```

此时，在dev和SSR模式下是可以正常运行的，但在SSG静态化的过程中会报错：

> Error: Image Optimization using Next.js' default loader is not compatible with `next export`.
>
> Possible solutions:
>
>  - Use `next start` to run a server, which includes the Image Optimization API.
>
>  - Configure `images.unoptimized = true` in `next.config.js` to disable the Image Optimization API.
>
> Read more: https://nextjs.org/docs/messages/export-image-api

这是由于<Imgae>自带的各种优化功能API是用于SSR的，是根据客户端的情况（设备类型、屏幕尺寸等）进行图片的动态优化处理，因此无法在SSG中使用。解决办法如下，修改next.config.js：

```js
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  experimental: {
    images: {
      unoptimized: true,
    },
  },
}

module.exports = nextConfig
```

## **生成静态化网站（SSG）**

> Next.js SSG的流程是：先执行build生成SSR版本（即.next目录里的文件），然后执行export生成SSG版本

修改package.json，将export命令设置在build之后执行：

```json
"scripts": {
  "dev": "next dev",
  "build": "next build && next export",
  "start": "next start",
  "lint": "next lint"
},
```

执行npm run build之后，SSG版本的静态化网站文件全部生成了，输出的位置是在项目根目录下，多出来一个out目录，结构如下：

```txt
/out
├─ /_next              <-- 静态资源目录
|  ├─ /static          <-- css、js、图片目录
|  └─ /y7j9HBq1Ymqrs76z0Budj    <-- 随机字符串，我也不知道这是啥，空的
├─ 404.html            <-- 404页面
├─ about.html          <-- about页面
├─ home.html           <-- home页面
└─ index.html          <-- index页面（同home页面）
```

需要注意的是：在本地双击打开out里面的html是无法正常运行的。因为页面引用的所有css、js、图片，都是以"/"开头的绝对路径，所以必须放到服务器的Web根目录里。可以使用Apache、Ngnix或者Node.js自行搭建一个Web服务器。把out目录里的文件全部放到Web根目录里。

### **设置静态资源的basePath**

静态网站部署方式是把“out目录内的”全部文件放到服务器Web根目录下。如果要放到二级目录里该怎么处理呢？例如这个SSG版本的网站要部署在app目录下

1. 配置Next.js项目的basePath，让HTML引入静态资源的路径都以"/app"开头

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  experimental: {
    images: {
      unoptimized: true,
    },
  },
  basePath: process.env.NODE_ENV === 'development' ? '' : '/app',
}

module.exports = nextConfig
```

2. 重新构建，npm run build

3. 把build出来的“out目录内的”全部文件放到服务器Web根目录下的app目录里。然后通过Web服务访问该SSG网站

   这是以访问http://127.0.0.1/app/为例，可以看到页面正常显示了，静态资源引用的地址也改为了以"/app"开头的绝对路径，因此可以正常加载

**注意：**进行以上设置后，SSR和SSG环境的网站访问路径都会多加了app这一级

- dev模式，home页面地址为http://localhost:3000

- SSR模式，home页面地址为http://localhost:3000/app/

- SSG模式，home页面地址为http://localhost:3000

### **设置SSG export输出的目录名称**

Next.js执行export命令默认输出的目录名称是out。当然可以自定义修改。只需为执行命令添加参数"-o 目录名称"即可。以输出目录名为app为例：

```
npm run build -o app
```

执行这个就相当于执行：

```
npx next build && npx next export -o app
```

## 接口请求

### **CSR/SSR/SSG**

在Next.js项目中，API请求有三种方式。但是根据项目的部署方式，最多可以同时有两种，即：

1. CSR+SSR

2. CSR+SSG

CSR的API请求，就是常规前端项目中的请求方式，即：由客户端浏览器发起请求，拿到数据后再渲染到页面

SSR的API请求，是由服务端（Next.js的Node.js）发起请求，拿到数据后，组装到HTML里，然后把组装好的HTML返回给客户端浏览器

SSG的API请求，与SSR的API请求类似，也是由服务器发起请求并把数据组装到HTML里，然后进行静态化输出。但由于是完全静态化的，所以当API数据发送变化的时候，必须重新静态化才能更新页面

|          | **CSR **     | **SSR**       | **SSG**       |
| -------- | ------------ | ------------- | ------------- |
| 发起请求 | 客户端浏览器 | 服务端Node.js | 服务端Node.js |
| 组装HTML | 客户端浏览器 | 服务端Node.js | 服务端Node.js |
| 接口数据 | 实时更新     | 实时更新      | 需重新静态化  |

### getStaticProps

> getStaticProps：SSR和SSG均支持，但仅在build的时候发起API请求，build后的网站不再请求API，数据不会更新
>
> 下面的代码在dev环境中一定是可以正常运行的。但是如果是在SSG环境下。在构建静态页面完成后，如果这时修改data.php里的数据，再次刷新页面，并不会显示新的数据，这时因为getStaticProps只在build时请求数据，因此不会再请求更新数据了

```tsx
import axios from 'axios'
import Link from 'next/link'

function Profile(props) {
  const { data } = props.profileData
  return (
    <div>
      <Link href='/'>
        <div className={pageStyles['top-bar']}>&lt;返回</div>
      </Link>
      <h1>This is Profile Page.</h1>
      <p>ID: {data.id}</p>
      <p>Name: {data.name}</p>
      <p>Age: {data.age}</p>
      <p>Favorite: {data.favorite}</p>
    </div>
  )
}

export const getStaticProps = async (context) => {
  const res = await axios({
    method: 'get',
    url: 'http://127.0.0.1/api/data.php',
    params: { id: 1 },
  })
  return {
    // 这里的props将会传递给组件使用
    props: {
      profileData: res.data,
    },
  }
}
export default Profile
```

由于getServerSideProps或者getStaticProps请求接口数据是在Node.js服务端进行的，所以不存在跨域的问题

|          | **getServerSideProps** | **getStaticProps** |
| -------- | ---------------------- | ------------------ |
| dev      | 支持                   | 支持               |
| SSR      | 支持                   | 支持               |
| SSG      | 不支持                 | 支持               |
| 执行时机 | 服务端收到页面请求时   | 仅在build时        |
| 数据更新 | 实时更新               | 必须重新build      |

#### context参数

- params 包含使用动态路由的页面的路由参数。例如，如果页面名称是[id]。那么params看起来就像{ id: ... }。它必须与getStaticPath一起使用

- preview 如果页处于预览模式且未定义，则预览为true

- previewData包含由setPreviewData设置的预览数据

- locale 包含活动的locale(如果启用)

- locales 包含所有受支持的locale(如果启用)

- defaultLocale 包含已配置的默认语言环境(如果启用)

#### 返回值

**props:** 

- 同getServerSideProps

**notFound:**

- 同getServerSideProps

- 注意：notfound不需要fallback: false模式，因为只有从getStaticPath返回的路径将被pre-rendered

**redirect:**

- 同getServerSideProps

- 注意：有点鸡肋，因为如果在构建时已知重定向，那么应该在next.config.js中添加它们

**revalidate:**

- 这个属性表示nextjs将要重新生成页面时要符合下面两个条件：
  - 在有新的请求进来的时候
  - 并且最多每10秒生成一次

```js
export async function getStaticProps() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  return {
    props: {
      posts,
    },
    revalidate: 10, // In seconds
  }
}
```

#### 阅读文件 process.cwd()

> 如果想要直接在getStaticProps中从文件系统读取文件，就必须得到一个文件的完整路径，而由于Next.js将代码编译到一个单独的目录中，因此不能使用__dirname，因为它返回的路径将不同于page目录。我们应该使用process.cwd()，它为我们提供执行Next.js的目录
>

```jsx
import { promises as fs } from 'fs'
import path from 'path'

function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>
          <h3>{post.filename}</h3>
          <p>{post.content}</p>
        </li>
      ))}
    </ul>
  )
}

export async function getStaticProps() {
  const postsDirectory = path.join(process.cwd(), 'posts')
  const filenames = await fs.readdir(postsDirectory)

  const posts = filenames.map(async (filename) => {
    const filePath = path.join(postsDirectory, filename)
    const fileContents = await fs.readFile(filePath, 'utf8')
    return {
      filename,
      content: fileContents,
    }
  })

  return {
    props: {
      posts: await Promise.all(posts),
    },
  }
}

export default Blog
```

### getServerSideProps

> getServerSideProps：适用于SSR，不支持SSG，服务端会重新发起请求更新数据

```jsx
import axios from 'axios'
import Link from 'next/link'

function Profile(props) {
  const { data } = props.profileData
  return (
    <div>
      <Link href='/'>
        <div className={pageStyles['top-bar']}>&lt;返回</div>
      </Link>
      <h1>This is Profile Page.</h1>
      <p>ID: {data.id}</p>
      <p>Name: {data.name}</p>
      <p>Age: {data.age}</p>
      <p>Favorite: {data.favorite}</p>
    </div>
  )
}

export const getServerSideProps = async (context) => {
  const res = await axios({
    method: 'get',
    url: 'http://127.0.0.1/api/data.php',
    params: { id: 1 },
  })
  return {
    // 这里的props将会传递给组件使用
    props: {
      profileData: res.data,
    },
  }
}
export default Profile
```

由于这里我们使用的是setServerSideProps，因此他只能用于SSR模式。这时执行npm run build就会在export环节报错。因为export是用于SSG的，而setServerSideProps不支持SSG。因此我们需要单独执行一下npx next build，然后手动启动项目npm run start即可

#### context参数

- params: 如果这个页面使用动态路由，params包含路由参数。如果页名为[id]。那么params看起来就像{ id: ... }

- req: HTTP IncomingMessage对象，带有一个附加cookie参数，它是一个具有映射到cookies的字符串值和字符串键的对象

- res: HTTP Response对象

- query: 表示查询字符串的对象，包括动态路由参数

- preview: 如果页面处于预览模式，则为true，否则为false

- previewData: 由setPreviewData设置的预览数据集

- resolvedUrl: 请求URL的规范化版本，它删除客户端转换的_next/data前缀，并包含原始查询值

- locale: 包含激活的locale(如果启用)

- locales: 包含所有受支持的locale(如果启用)

- defaultLocale: 包含已配置的默认语言环境(如果启用)

#### 返回值

**props:**

> Props对象是一个键-值对，其中每个值都由页面组件接收

```tsx
export async function getServerSideProps(context) {
  return {
    props: { message: `Next.js is awesome` }, // will be passed to the page component as props
  }
}
```

**notfound:**

> notFound布尔值允许页面返回404状态和404 Page。使用notFound: true，即使之前已经成功生成了页面，该页面也将返回404。这是为了支持像用户生成内容被作者删除这样的用例

```tsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  if (!data) {
    return {
      notFound: true,
    }
  }

  return {
    props: { data }, // will be passed to the page component as props
  }
}
```

**redirect:**

> 重定向对象允许重定向到内部和外部资源。类似于{ target: string，forever: boolean }。在某些罕见的情况下，您可能需要为较旧的HTTP客户端分配一个自定义状态代码以进行正确的重定向。在这些情况下，可以使用statusCode属性而不是permanent属性，但不能同时使用这两个属性

```tsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  if (!data) {
    return {
      redirect: {
        destination: '/',
        permanent: false,
      },
    }
  }

  return {
    props: {}, // will be passed to the page component as props
  }
}
```

### getInitialProps

> Next.js v9.3版本引入了getServerSideProps，在定义服务端渲染这边页面上的props，getServerSideProps基本上取代了getInitialProps
>
> 基本上来说，如果你要在编译阶段发出任何请求之前渲染页面，你应该使用`getStaticProps`，该方法会使页面静态化进行渲染，也意味着后续不会被重复渲染直到下一次的编译。虽然该方法对于渲染速度及SEO有显著的提高，但它不适合经常改变的动态数据渲染方式
>
> 如果你希望在请求时渲染页面，你可以使用`getServerSideProps`方法，在客户端得到请求响应时渲染页面。该方法会对每个请求都是做页面渲染，虽然该方法使得服务端会对每个请求重新构建页面，从而降低整体的访问速度，然而该方法相比React在客户端渲染的普通方式有着较大的SEO优势
>
> 关于语言和框架的升级，向后兼容性很重要，尽管老的模式或方法在后续的工程里不再使用，它们仍然被维护，以此保证不破坏遗留代码及边界情况。这就是`getInitialProps`的情况，它几乎和`getServerSideProps`有着一样的用法行为

**`getServerSideProps`和`getInitialProps`的不同点：**

遗留的getInitialProps和较新的getServerSideProps之间的主要区别在于，当用户单击一个Link访问站点的不同部分时，如何在转换期间调用该函数

使用getInitialProps，转换将在初始页面加载时在服务器端执行，然后在页面转换期间在客户机上执行。但是，如果逻辑指的是数据库之类的东西，而这些东西在客户机上可能无法访问，那么就会产生问题

例如，在下面的代码片段中，直接从数据库获取用户将在初始页面加载时工作。但是，它可能在转换时失败，因为User模型在客户机上不可用：

```jsx
// Import a User model
import User from "../models/User"
function Page({ User }) {
    return <div>Username: {User.username}</div>
}

Page.getInitialProps = async (context) => {
    // Get user id
    const User = await User.findOne(ctx.query.id)
    // return props
    return {
        User
    }
}

export default Page
```

而getServerSideProps将在初始页面加载时在服务器端执行转换。在页面转换时，getServerSideProps将对服务器进行API调用，在服务器上再次运行逻辑并将结果作为JSON返回。通过进行此更改，我们解决了getInitialProps中出现的上下文切换问题。在下面的示例中，您可以直接调用数据库，它在初始页面加载和转换时都能正常工作:

```jsx
// Import a User model
import User from "../models/User"
function Page({ User }) {
    return <div>Username: {User.username}</div>
}
export async function getServerSideProps(context) {
    // Get user id
    const User = await User.findOne(ctx.query.id)
    return {
        props: {User}, // will be passed to the page component as props
    }
} 

export default Page
```

#### context参数

- pathname: 当前路由。这是/page中页面的路径

- query: 解析为对象的URL的query字符串部分

- asPath: 浏览器中显示的实际路径(包括查询)的字符串

- req: HTTP请求对象(仅限于服务器)

- res: HTTP响应对象(仅限于服务器)

- err: 如果在呈现过程中遇到任何错误，则返回

#### 返回值

- 他返回一个对象，用于填充props

### getStaticPaths

> SSG模式，所有页面都是静态化的，怎么把所有id值的页面都自动逐个静态化？
>
> SSR模式下，想要把一部分动态路由的页面静态化，要怎么才能做到？
>
> 这时就可以用到Next.jsx给我们提供的getStaticPaths方法

#### return值: paths

paths键确定将预先呈现哪些路径。例如，假设有一个使用Dynamic Routes命名的页面/post/[id]。如果从此页导出getStaticPath并返回以下路径，那么Next.js 将在下一个构建过程中静态生成/post/1和/post/2

```js
return {
  paths: [
    { params: { id: '1' }},
    {
      params: { id: '2' },
      // with i18n configured the locale for the path can be returned as well
      locale: "en",
    },
  ],
  fallback: ...
}
```

用于逐个生成页面每个 params 对象的值必须与页面名称中使用的参数匹配:

- 如果页面名是pages/post/[postId]/[commentId]，那么参数应该包含postId和commentId

- 如果页面名称使用诸如page/[...slug]之类的捕捉所有路由，那么params应该包含slug(这是一个数组)。如果这个数组是[‘hello’，‘world’] ，那么Next.js将在/hello/world下生成静态页面

- 如果页面使用可选的catch-all路由，可以使用null、[]、undefined或false来呈现最根的路由。例如，如果为pages/[[...slug]]提供slug: false，Next.js将生成静态页面/

#### **return值: fallback**

如果fallback为false，那么getStaticPath未返回的所有路径都将导致变成404页面。当下一次构建运行时，Next.js将检查getStaticPath是否返回fallback: false，然后它将只构建由 getStaticPath返回的路径。如果要创建的路径数量较少，或者不经常添加新的页数据，则此选项非常有用

如果需要添加更多的路径，并且出现了fallback: false，则需要再次运行下一次构建，以便生成新的路径

下面的示例pre-rendered每个页面的一篇博客文章。博客文章列表将从CMS获取，并由getStaticPath返回。然后，对于每个页面，它使用getStaticProps从CMS获取发布数据

```js
// pages/posts/[id].js
function Post({ post }) {
  // Render post...
}

export async function getStaticPaths() {
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))

  // 我们将在构建时pre-render这些路径
  // { falback: false } 意味着其他路由应该为404
  return { paths, fallback: false }
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://.../posts/${params.id}`)
  const post = await res.json()

  return { props: { post } }
}

export default Post
```

如果fallback为true，则getStaticProps的行为会以下列方式发生变化:

- 从getStaticPath返回的路径将在构建时由getStaticProps render为静态页面

- 在构建时未生成的路径将不会生成404页。相反，Next.js将在第一次请求这些路径时提供页面的“fallback”版本。像谷歌这样的网络爬虫不会得到fallback服务，它访问的路径会表现为fallback: "blocking"

- 当通过next/link或next/router(客户端)导航到一个带有fallback: true的页面时，Next.js将不会提供fallback服务，该页面会表现为fallback: "blocking"

- 在后台，Next.js将静态地生成所请求的路径HTML和JSON，包括运行getStaticProps

- 完成后，浏览器接收生成路径的JSON。这将用于自动render所需props的页面。从用户的角度来看，页面将从fallback页面切换到完整页面

- 同时，Next.js将此路径添加到pre-render页面列表中。对同一路径的后续请求提供服务，就像在构建时pre-render的其他页一样

如果fallback是“blocking”，getStaticPath没有返回的新路径将等待生成与SSR相同的HTML(因此为什么阻塞) ，然后缓存以备将来的请求，这样每个路径只会发生一次。

getStaticProps 的行为如下:

- 从getStaticPath返回的路径将在构建时由getStaticProps渲染为静态页面

- 在构建时未生成的路径将不会生成404页。相反，Next.js将对第一个请求进行SSR并返回生成的HTML

- 完成后，浏览器接收生成路径的HTML。从用户的角度来看，它将从“浏览器正在请求页面”过渡到“加载完整页面”。没有loading/fallback状态

- 同时，Next.js将此路径添加到pre-render页面列表中。对同一路径的后续请求提供服务，就像在构建时pre-render的其他页一样

fallback: "blocking"默认情况下不会更新生成的页面。要更新生成的页面，使用 [Incremental Static Regeneration(静态增量再生)](https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration) 和fallback: "blocking"结合使用

> **注意:** fallback: true | “blocking”模式不支持使用next export

#### falllback页面

> 在页面的“fallback”版本中：
>
> - 该页面的props将是空的
> - 使用router可以检测到fallback是否正在渲染(router.isFallback)

```js
// pages/posts/[id].js
import { useRouter } from 'next/router'

function Post({ post }) {
  const router = useRouter()
  // 如果页面尚未生成，则将显示此信息
  // 初始化，直到 getStaticProps()完成运行
  if (router.isFallback) {
    return <div>Loading...</div>
  }
  // Render post...
}

export async function getStaticPaths() {
  return {
    // 在构建时只生成“/post/1”和“/post/2”
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
    // 启用静态生成其他页，例如: ‘/post/3’
    fallback: true,
  }
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://.../posts/${params.id}`)
  const post = await res.json()
  return {
    props: { post },
    // 如果有请求的话，每秒最多重新生成一次页面
    revalidate: 1,
  }
}

export default Post
```

### **搭建Next.js API Routers服务**

> pages目录下的api目录是Next.js指定的特殊目录，专用于搭建Next.js的自建API服务
>
> **注意：**
>
> - pages/api目录提供的API服务不可用于SSG
> - pages/api目录只运行在服务端，并不会增加客户端代码的大小
> - 由于API Routers运行在服务端，因此不能在getStaticProps方法中作为请求地址使用。因为在build的时候，API服务并没启动，所以获取不到数据

- **pages/api/test.js**

```js

export default function handler(req, res) {
    if (req.method === 'GET') {
        const { id } = req.query
        let data = { id }
        switch (id) {
            case '1':
                data.name = 'Tom'
                data.age = 21
                data.favorite = 'reading, sport'
                break
            case '2':
                data.name = 'Jerry'
                data.age = 22
                data.favorite = 'art, music'
                break
            case '3':
                data.name = 'Bill'
                data.age = 23
                data.favorite = 'comic, movie'
                break
            default:
                break
        }
        res.status(200).json({ message: 'OK', data })
    } else {
        res.status(200).json({ message: 'OK', data })
    }
}
```

以上代码相当于生成了一个test数据接口服务。在dev和SSR模式下，该API请求的地址就是http://localhost:3000/api/test

## 使用CLI命令自定义输出目录

如果我们的项目是一个比较通用的项目，希望通过CLI命令，根据需要动态生成不同的目录和basePath，那就不能把basePath写死在next.config.js里了

这种场景适合于自动化部署，即：根据传递过来的目录名参数，动态生成静态化目录

- 先安装cross-env这个插件，用来统一跨平台的环境变量写法：

```bash
npm instal cross-env --dev
```

- 如果是Windows系统，在package.json中添加customBuild脚本：

```json
"scripts": {
  "dev": "next dev",
  "build": "cross-env BASE_PATH=/test next build && next export",
  "start": "next start",
  "lint": "next lint",
  "customBuild": "cross-env BASE_PATH=%npm_config_base% next build && next export -o %npm_config_out%"
}
```

- 如果是MacOS或者Linux，刚才的命令行传参需要这种格式进行调整：

```json
{
  "customBuild": "cross-env BASE_PATH=$npm_config_base next build && next export -o $npm_config_out"
}
```

- 修改next.config.js：

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  experimental: {
    images: {
      unoptimized: true,
    },
  },
}

if (process.env.BASE_PATH) {
  nextConfig.basePath = process.env.BASE_PATH
} else {
  nextConfig.basePath = process.env.NODE_ENV === 'development' ? '' : '/app'
}

module.exports = nextConfig
```

- 然后执行下面的指令：

```bash
npm run customBuild --base=/diy --out=diy
```

完成之后就可以发现生成了diy的输出目录，basePath也变成了diy。通过这种方式，可以结合系统脚本，实现自动化部署

## redux

- **store/counter/action.js**

```js
export const counterActionTypes = {
  INCREMENT: "INCREMENT",
};

export const incrementCounter = () => {
  return { type: counterActionTypes.INCREMENT };
};
```

- **store/counter/reducer.js**

```js
import { counterActionTypes } from "./action";

const counterInitialState = {
  count: 0,
};

export default function reducer(state = counterInitialState, action) {
  switch (action.type) {
    case counterActionTypes.INCREMENT:
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
}
```

- **store/users/action.js**

```js
export const usersActionTypes = {
  ADD_USER: "ADD_USER",
};

export const addUser = (newUser) => {
  return { type: usersActionTypes.ADD_USER, user: newUser };
};
```

- **store/users/reducer.js**

```js
import { usersActionTypes } from "./action";

const usersInitialState = {
  users: ["John Doe", "Mary Jane"]
};

export default function reducer(state = usersInitialState, action) {
  switch (action.type) {
    case usersActionTypes.ADD_USER: {
        return { ...state, users: [...state.users, action.user] };
      }
    default:
      return state;
  }
}
```

- **store/store.js**

> next-redux-wrapper这个库帮我们做的事情就是，把在SSR期间(getServerSideProps)的dispatch结果合并到客户端redux存储中(通过派发一个叫HYDRATE的动作)，但不会将客户端存储状态同步到服务器redux存储
>
> 还有就是通过工厂模式(initStore)的方式来生成store，这样可以防止用户之间访问数据的冲突

```js
import { createStore, applyMiddleware, combineReducers } from 'redux'
import { HYDRATE, createWrapper } from 'next-redux-wrapper'
import { composeWithDevTools } from 'redux-devtools-extension'
import users from './users/reducer'
import counter from './counter/reducer'

const combinedReducer = combineReducers({
  counter,
  users,
})

const masterReducer = (state, action) => {
  if (action.type === HYDRATE) {
    const nextState = {
      ...state,
      counter: {
        count: state.counter.count + action.payload.counter.count
      },
      users: {
        users: [...new Set([...action.payload.users.users, ...state.users.users])]
      }
    }
    return nextState
  }
  else {
    return combinedReducer(state, action);
  }
}

const initStore = () => {
  return createStore(masterReducer, composeWithDevTools(
    applyMiddleware()
  ))
}

export const wrapper = createWrapper(initStore)
```

- **pages/_app.js**

```js
import { wrapper } from '../store/store'

const App = ({ Component, pageProps }) => {
  return <Component {...pageProps} />
}

export default wrapper.withRedux(App)
```

- **pages/index.js**

> 每当用户打开`getStaticProps`的页面时，`HYDRATE`操作将被调度。此操作的`payload`包含页面静态生成时`state`，但是客户端没有这些`state`，因此我们必须将其与现有的客户端`state`正确合并

```js
import Page from '../components/Page'
import { incrementCounter } from '../store/counter/action'
import { wrapper } from '../store/store'

const Index = (props) => {
  return <Page title="Index Page" linkTo="/other" />
}

export const getStaticProps = wrapper.getStaticProps((store) => () => {
  store.dispatch(incrementCounter())
})

export default Index
```

- **pages/other.js**

> 每当用户打开`getServerSideProps`的页面时，`HYDRATE`操作将被调度。此操作的`payload`包含ssr时的`state`，而客户端不会收到这些`state`，因此我们必须将其与现有的客户端`state`正确合并

```js
import Page from "../components/Page";
import { wrapper } from "../store/store";
import { incrementCounter } from "../store/counter/action";
import { addUser } from '../store/users/action';

const Other = (props) => {
  return <Page title="Other Page" linkTo="/" />;
};

export const getServerSideProps = wrapper.getServerSideProps((store) => async () => {
  store.dispatch(incrementCounter());
  
  const response = await fetch(`https://reqres.in/api/users/${Math.floor(Math.random() * (10) + 1)}`);
  const {data} = await response.json();
  store.dispatch(addUser(`${data.first_name} ${data.last_name}`))
});

export default Other;
```

- **components/Page.js**

```js
import Link from "next/link";
import Users from "./Users";
import AddCount from "./IncrementCounter";

const Page = (props) => {
  return (
    <div>
      <h1 style={{backgroundColor: "#ddd"}}>{props.title}</h1>
      <Users/>
      <AddCount />
      <br/>
      <nav>
        <Link href={props.linkTo}>
          <a>Navigate</a>
        </Link>
      </nav>
    </div>
  );
};

export default Page;
```

- **components/IncrementCounter.js**

```js
import React from 'react'
import { useSelector, useDispatch } from 'react-redux'

const IncrementCounter = () => {
  const dispatch = useDispatch()
  const counter = useSelector((state) => state.counter.count)
  return (
    <div>
      <h1>
        Counter: <span>{counter}</span>
      </h1>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>Add To Count</button>
    </div>
  )
}

export default IncrementCounter
```

- **components/Users.js**

```js
import React, { useState } from "react";
import { useSelector, useDispatch } from "react-redux";
import { addUser } from "../store/users/action";

export default function Clock() {
  const dispatch = useDispatch();
  const { users } = useSelector((state) => state.users);
  const [name, setName] = useState("");

  const addNewUser = () => {
    dispatch(addUser(name));
  };

  const handleChange = (event) => {
    setName(event.target.value);
  };

  return (
    <div>
      <label>New User:</label>
      <input type="text" value={name} onChange={handleChange} />
      <button onClick={addNewUser}>Add</button>
      <h4>User List:</h4>
      <ol>
        {users.map((user) => (
          <li>{user}</li>
        ))}
      </ol>
    </div>
  );
}
```



