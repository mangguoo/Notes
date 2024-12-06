# service worker

> Service worker在客户端和服务端建立一个中间层，做了存储，加速了重复访问的速度

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/5e83fbb973429aa1be19e1ba9ddf39a0.png" alt="5e83fbb973429aa1be19e1ba9ddf39a0" style="zoom:50%;" />

## 使用service worker

### 安装

```shell
$ yarn add workbox-webpack-plugin
```

### 配置

- **webpack.config.js**

```js
const WorkboxPlugin = require('workbox-webpack-plugin');

module.exports = {  
  plugins: [    
    new WorkboxWebpackPlugin.GenerateSW({
      clientsClaim: true,
      exclude: [/\.map$/, /asset-manifest\.json$/],
      importWorkboxFrom: 'cdn',
      navigateFallback: paths.publicUrlOrPath + 'index.html',
      navigateFallbackBlacklist: [
        new RegExp('^/_'),
        new RegExp('/[^/?]+\\.[^/]+$'),
      ],
    }),
    // Other plugins...
  ],
  // Other webpack config...
};
```

### 打包

``` shell
$ yarn run build
```

此时在dist目录下会自动生成`precache-manifest.<revision>.js`和`service-worker.js`文件(register("/service-worker.js")

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/wd_6371355953582703382951896.png" alt="ab.png" style="zoom:67%;" />

在`precache-manifest.<revision>.js`文件中，将预缓存列表通过全局变量`self.__precacheManifest`公开，以便在service-worker.js中调用，默认会预缓存一切资源

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/wd_6371355951844995637464144.png" alt="1897634392-5ce769b856f64_articlex.png" style="zoom:67%;" />

在service-worker.js中，则自动加载`precache-manifest.<revision>.js`文件

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/wd_6371355949300561058069691.png" alt="ccc.png" style="zoom:67%;" />

### 注册

最后在入口文件中注册即可：

```js
import * as serviceWorker from './serviceWorker'

serviceWorker.register("/service-worker.js")
```

