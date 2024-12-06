# manifest

## manifest.json

> `manifest.json`经常被用在[PWA](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps)，用来告知浏览器关于PWA应用的一些信息如应用图标、启动应用的画面

```json
{
  "short_name": "IKEYIT",
  "name": "IKEYIT",
  "icons": [
    {
      "src": "favicon.ico",
      "sizes": "64x64 32x32 24x24 16x16",
      "type": "image/x-icon"
    },
    {
      "src": "logo192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "logo512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

## assets-manifest.json

> `assets-manifest.json`经常会在create-react-app这个脚手架的打包文件中看到，由`webpack-manifest-plugin`这个webpack插件产生，`assets-manifest.json`其实只是源文件和加哈希后文件的一个对比表，它不会对应用的运行产生任何影响，浏览器也不会去请求它

```json
{
  "files": {
    "runtime.js": "./static/js/runtime.6ad36f88.js",
    "runtime.js.map": "./static/js/runtime.6ad36f88.js.map",
    "static/js/1.1b841b7f.chunk.js": "./static/js/1.1b841b7f.chunk.js",
    "static/js/1.1b841b7f.chunk.js.map": "./static/js/1.1b841b7f.chunk.js.map",
    "static/js/2.a7eca0dd.chunk.js": "./static/js/2.a7eca0dd.chunk.js",
    "static/js/2.a7eca0dd.chunk.js.map": "./static/js/2.a7eca0dd.chunk.js.map",
    "account-center.css": "./static/css/account-center.bdd9253b.chunk.css",
    "account-center.js": "./static/js/account-center.3f9040d5.chunk.js",
    "account-center.js.map": "./static/js/account-center.3f9040d5.chunk.js.map",
    "login.css": "./static/css/login.b55ef5c0.chunk.css",
    "login.js": "./static/js/login.c92545ea.chunk.js",
    "login.js.map": "./static/js/login.c92545ea.chunk.js.map",
    "main.css": "./static/css/main.e4ac3e55.chunk.css",
    "main.js": "./static/js/main.0ce1db46.chunk.js",
    "main.js.map": "./static/js/main.0ce1db46.chunk.js.map",
    "seller-center.css": "./static/css/seller-center.14a25345.chunk.css",
    "seller-center.js": "./static/js/seller-center.787f704f.chunk.js",
    "seller-center.js.map": "./static/js/seller-center.787f704f.chunk.js.map",
    "shop-decorator.css": "./static/css/shop-decorator.6b05c47d.chunk.css",
    "shop-decorator.js": "./static/js/shop-decorator.7ec6a114.chunk.js",
    "shop-decorator.js.map": "./static/js/shop-decorator.7ec6a114.chunk.js.map",
    "super-center.css": "./static/css/super-center.2c647b14.chunk.css",
    "super-center.js": "./static/js/super-center.69d84907.chunk.js",
    "super-center.js.map": "./static/js/super-center.69d84907.chunk.js.map",
    "static/js/9.42bf8a31.chunk.js": "./static/js/9.42bf8a31.chunk.js",
    "static/js/9.42bf8a31.chunk.js.map": "./static/js/9.42bf8a31.chunk.js.map",
    "account-center.html": "./account-center.html",
    "index.html": "./index.html",
    "login.html": "./login.html",
    "seller-center.html": "./seller-center.html",
    "shop-decorator.html": "./shop-decorator.html",
    "static/css/account-center.bdd9253b.chunk.css.map": "./static/css/account-center.bdd9253b.chunk.css.map",
    "static/css/login.b55ef5c0.chunk.css.map": "./static/css/login.b55ef5c0.chunk.css.map",
    "static/css/main.e4ac3e55.chunk.css.map": "./static/css/main.e4ac3e55.chunk.css.map",
    "static/css/seller-center.14a25345.chunk.css.map": "./static/css/seller-center.14a25345.chunk.css.map",
    "static/css/shop-decorator.6b05c47d.chunk.css.map": "./static/css/shop-decorator.6b05c47d.chunk.css.map",
    "static/css/super-center.2c647b14.chunk.css.map": "./static/css/super-center.2c647b14.chunk.css.map",
    "static/js/1.1b841b7f.chunk.js.LICENSE.txt": "./static/js/1.1b841b7f.chunk.js.LICENSE.txt",
    "static/js/2.a7eca0dd.chunk.js.LICENSE.txt": "./static/js/2.a7eca0dd.chunk.js.LICENSE.txt",
    "static/js/seller-center.787f704f.chunk.js.LICENSE.txt": "./static/js/seller-center.787f704f.chunk.js.LICENSE.txt",
    "super-center.html": "./super-center.html"
  },
  "entrypoints": [
    "static/js/runtime.6ad36f88.js",
    "static/js/1.1b841b7f.chunk.js",
    "static/css/main.e4ac3e55.chunk.css",
    "static/js/main.0ce1db46.chunk.js"
  ]
}
```

## precache-manifest.js

> 这个文件由`workbox-webpack-plugin`插件生成，用来告诉workbox哪些静态文件可以缓存
