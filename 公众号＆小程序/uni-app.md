# uni-app

> uni-app 是一个使用 Vue.js 开发所有前端应用的框架，开发者编写一套代码，可发布到iOS、Android、Web（响应式）、以及各种小程序（微信/支付宝/百度/头条/飞书/QQ/快手/钉钉/淘宝）、快应用等多个平台。

## hello uni-app

> HBuilderX对uni-app的适配非常好，可以直接使用它创建uni-app的模板项目，并且语法提示非常完善，对于mainfest配置文件，HBuilderX还给出了图形界面帮助我们去配置，并且他会自动帮助我们去下载开发时所需要的依赖，比如我们使用到了sass，但是没有安装sass依赖，在我们去编译运行项目的时候，他就会自动帮我们去下载依赖

**基本目录结构：**

```txt
│  App.vue // 根组件，类似于原生小程序的根app
│  index.html // 用作跨平台，比如说网页端
│  main.js // 打包的入口文件
│  manifest.json // 项目配置文件
│  pages.json // 页面配置，类似于app.json
│  uni.scss // 一些公用样式，定义了一些变量  
│  unpackage // 打包文件
├─pages // 存储页面
│  └─index
│     └─index.vue 
└─static // 存储静态文件
```

项目创建完成后，我们要先设置小程序的appid，否则调试的时候微信开发工具会提示我们使用的是游客模式登录：

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221029121427602.png" alt="image-20221029121427602" style="zoom: 50%;" />

配置完成后，就可以启动调试项目，由于每个软件的小程序都有自己的开发调试平台，因此我们开发什么小程序，就需要使用什么开发工具去调试，而uni-app也给出了一键开启调试环境：

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221029122229327.png" style="zoom:50%;" />

但是想要使用uni-app直接打开微信小程序的调试环境，必须先在微信小程序开发工具中进行一些设置：

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221029123155789.png" alt="image-20221029123155789" style="zoom:50%;" />

> 在调试项目时，微信小程序的开发工具会报`DevTools failed to parse SourceMap`(请求不到sourceMap，无法定位到源代码)，要想解决这个问题，需要下载一个插件：`jsSourceMap`

- 安装插件后，并且uniapp项目编译完成后，右击项目，执行修改jsSourceMap地址

- 插件会修改 unpackage/dist/dev/mp-weixin 目录下的所有js文件

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221029141137572fdsa.png" style="zoom:50%;" />

## uni-app配置

```json
{
  "pages": [ // 页面配置，增加和删除页面都需要在这里进行配置 
    {
      "path": "pages/index/index", // 页面的路径
      "style": { // 页面配置项，类似于原生小程序中的配置文件(page.json)
        "navigationBarTitleText": "首页", // 页面标题，显示在头部
        "enablePullDownRefresh": true // 是否可以下拉刷新
      }
    },
    {
      "path": "pages/category/category",
      "style": {
        "enablePullDownRefresh": false
      }
    }
  ],
  "subPackages": [ // 分包配置
    {
      "root": "subpages", // 分包的根目录
      "pages": [ // 分包页面（可以有多个分包）
        {
          "path": "detail/detail", // 分包页面路径
          "style": {} // 分包页面的配置
        }
      ]
    }
  ],
  "globalStyle": { // 全局配置，类似于app.json，会被页面配置所覆盖
    "navigationBarBackgroundColor": "#1387C9", // 头部背景颜色
    "navigationBarTextStyle": "white", // 头部字体颜色
    "navigationBarTitleText": "uniApp学习使用", // 标题
    "h5": { // 编译为h5页面时，显示的配置(可以根据不同的环境进行不同的配置)
      "titleNView": {
        "titleText": "我是H5页面"
      }
    }
  },
  "tabBar": { // 底部菜单栏
    "color": "#000", // 字体颜色
    "selectedColor": "#3cc51f", // 被命中时的颜色
    "borderStyle": "black", // 边框
    "backgroundColor": "#ffffff", // 背景色
    "list": [ // 按钮列表
      {
        "pagePath": "pages/index/index", // 路由
        "iconPath": "static/center.png", // 图标
        "selectedIconPath": "static/center-active.png", // 命中后的图标
        "text": "首页" // 文字描述
      },
      {
        "pagePath": "pages/category/category",
        "iconPath": "static/home.png",
        "selectedIconPath": "static/home-active.png",
        "text": "分类"
      }
    ]
  }
}
```

## 网络请求

- **/pages/index/index.vue**

> 请求电影列表，并通过列表方式渲染出来，点击时分别通过编程式导航和声明式导航的方式跳转进详情页面

```vue
<template>
  <view class="container">
    <view class="film-item" v-for="(item, index) in films" :key="item.filmId">
      <!-- 编程式导航 -->
      <image @click="goDetail(item.filmId)" :src="item.poster"></image>
      <view>
        <!-- 声明式导航 -->
        <navigator :url="`/subpages/detail/detail?id=${item.filmId}`">{{ substr(item.name) }}</navigator>
      </view>
    </view>
    <view class="go-top" :class="{ active: showTopBtn }" @click="goTop"></view>
  </view>
</template>

<script>
import { get } from "@/utils/http.js";
export default {
  data() {
    return {
      films: [],
      page: 1,
      showTopBtn: false,
    };
  },
  // onLoad在网页端先于created生命周期执行，而在小程序中是在created后执行
  // 因此在混合式开发框架中，尽量用vue中的生命周期，除非需要使用一些小程序特有的生命周期
  // onLoad中接收一个参数options，可以用于获取路由传递过来的query
  onLoad(options) {
    console.log("onload");
  },
  async created() { // 在created中发送网络请求
    // 这里进行环境判断，因为在小程序环境中请求api不会涉及跨域，而在h5中，因为域名不相同，因此会涉及跨哉
    // 因此要为h5环境做代理跨域，但是编译为小程序环境又没有devServer，这样就导致两个环境需要单独配置
    // 这里就可以使用环境判断，为h5和小程序分别定义url，让程序直接访问，而让h5走代理服务器
    // let [err, ret] = await uni.request({
    // 	 #ifdef MP
    // 	 url: 'https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageNum=1&pageSize=10'
    // 	 #endif
    // 	 #ifdef H5
    // 	 url: '/api/v1/getNowPlayingFilmList?cityId=110100&pageNum=1&pageSize=10'
    // 	 #endif
    // })
    this.loadData(); // 对环境判断的网络请求进行封装
  },
  onPageScroll({ scrollTop }) { // 小程序特有生命周期，页面滚动时执行
    this.showTopBtn = scrollTop >= 100;
  },
  onReachBottom() { // 小程序特有生命周期，滚动到底部时触发
    this.loadData(); // 滚动到底部时加载更多数据
  },
  methods: {
    async loadData() {
      let page = this.page;
      let [err, ret] = await get("/api/v1/getNowPlayingFilmList?cityId=110100&pageSize=10&pageNum=" + page);
      if (ret.data.data.films.length > 0) {
        if (page === 1) {
          this.films = ret.data.data.films;
        } else {
          this.films.push(...ret.data.data.films);
          // 数据会一直追加,因此这里只截取20条
          this.films = this.films.slice(-21, -1);
        }
        this.page++;
      } else {
        uni.showToast({ // 提示，没有更多数据
          title: "没有更多的新数据了",
        });
      }
    },
    substr(str) {
      return str.slice(0, 6);
    },
    goDetail(id) {
      uni.navigateTo({ // 编程式导航
        url: `/subpages/detail/detail?id=${id}`, // 跳转到详情页面
      });
    },
    goTop() {
      uni.pageScrollTo({ // 使用小程序的api方法
        scrollTop: 0,
        duration: 0,
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  .go-top {
    width: 100rpx;
    height: 100rpx;
    background: red;
    position: fixed;
    bottom: 10rpx;
    right: 10rpx;
    z-index: 1;
    opacity: 0;
    &.active {
      opacity: 1;
    }
  }
}
.film-item {
  display: flex;
  margin-bottom: 20rpx;
  image {
    width: 300rpx;
    height: 300rpx;
  }
}
</style>
```

- **/utils/http.js**

> 对环境判断的网络请求进行封装

```js
const http = (url, method = "GET", data = {}) => {
  // #ifdef MP
  url = "https://api.iynn.cn/film" + url; // 如果是小程序端，则加上域名，不走代理
  // #endif
  return uni.request({
    url,
    method,
    data,
  });
};
export const get = (url) => http(url);
```

> **代理配置：**
>
> - uni-app提供了配置代理的方式，在mainfest.json中进行配置
> - 也可以直接使用vue的代理，在vue.config.js中进行配置

- **/mainfest.json**

```json
{
  // ... ...
  "h5": {
    "devServer": {
      "proxy": {
        "/api": {
          "target": "https://api.iynn.cn/film/",
          "changeOrigin": true
        }
      }
    }
  }
}
```

- **/vue.config.js**

```js
module.exports = {
  devServer: {
    proxy: {
      "/api": {
        target: "https://api.iynn.cn/film/",
        changeOrigin: true,
      },
    },
  },
};
```

## uni-app分包处理

> 由于小程序有体积和资源加载限制，各家小程序平台提供了分包方式，优化小程序的下载和启动速度。而uni-app是做跨端开发看，所以也就引入了分包机制
>
> uni-app默认是把所有page打包在一起，但是为了防止加载不需要的页面，提升首屏加载速度。这样就能在启动时只加载主包，按需要加载分包。提高进入页面时的加载速度，不至于必须等所有页面加载完
>
> - 微信小程序每个分包的大小是2M，总体积一共不能超过20M
> - 百度小程序每个分包的大小是2M，总体积一共不能超过8M
> - 支付宝小程序每个分包的大小是2M，总体积一共不能超过4M
> - QQ小程序每个分包的大小是2M，总体积一共不能超过24M
> - 字节小程序每个分包的大小是2M，总体积一共不能超过16M

### 查看分包体积

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/8b3efbaaf89a4bf89aca4749a901e359.png" alt="8b3efbaaf89a4bf89aca4749a901e359" style="zoom: 67%;" />

### 实现分包

- **/pages.json**

> 配置分包所在的路径，以及分包的一些配置

```json
{
  // ... ...
  "subPackages": [
    {
      "root": "subpages",
      "pages": [
        {
          "path": "detail/detail",
          "style": {}
        }
      ]
    }
  ]
}
```

- **/subpages/detail/detail.vue**

> **引入第三方组件(以u-parse为例)：**
>
> - 首先去DCloud插件市场找到想要的组件
> - 点击“使用HBuilderX"导入插件
> - HBuilderX会自动打开，并开始下载
> - 组件下载完成后会被放进根目录下的components目录下
> - 这时就可以直接引入并使用即可
>
> **注意：**由于detail页面进行了分包，因此在小程序环境下可以引入不到/componets目录下的组件(h5环境下无异常)，解决方法就是直接移动这个组件到分包的页面中去
>
> <img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221029230228447.png" alt="image-20221029230228447" style="zoom: 67%;" />

```vue
<template>
  <view>
    <block v-if="film === null">
      <!--  请求完成之前显示Loading组件 -->
      <loading></loading>
    </block>
    <block v-else>
      <view>{{ film.name }}</view>
      <!--  小程序自带的富文本功能 -->
      <rich-text :nodes="film.body"></rich-text>
      <!--  第三方富文件功能组件 -->
      <u-parse :content="film.body"></u-parse>
    </block>
  </view>
</template>

<script>
import { get } from "@/utils/http.js";
import loading from "@/components/loading.vue";
// 引入第三方富文件功能组件
import uParse from "./u-parse/u-parse.vue";

export default {
  components: {
    loading,
    uParse,
  },
  data() {
    return {
      film: null,
    };
  },
  async mounted() {
    // 这里没有直接通过onLoad生命周期中的options参数来获取query，是因为低版本的uni-app它是不能用onLoad，因此建议用vue的生命周期方法来获取
    let id = this.$mp.query.id; // this.$mp表示组件实例
    let [err, ret] = await get("/api/v1/getFilmInfo?filmId=" + id);
    uni.setNavigationBarTitle({ // 动态修改当前标题
      title: this.film.name,
    });
    let data = ret.data.data.film;
    data.body = `<p style="box-sizing: border-box; margin-top: 20px; margin-bottom: 20px; padding: 0px; border: 0px; color: rgb(34, 34, 34); font-family: &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, &quot;Helvetica Neue&quot;, Arial, sans-serif; font-size: 18px; text-align: justify; white-space: normal; background-color: rgb(255, 255, 255);">一场秋雨过后，气温骤降，<span style="box-sizing: border-box; margin: 0px; padding: 0px; border: 0px; color: rgb(51, 51, 51); --tt-darkmode-color: #A3A3A3;">这就是所谓的“一场秋雨，一场寒”。天气变冷，</span>除了及时添加衣服之外，还要多吃一些滋补的食物，给身体储蓄能量。<span style="box-sizing: border-box; margin: 0px; padding: 0px; border: 0px; color: rgb(51, 51, 51); --tt-darkmode-color: #A3A3A3;">秋季的天气比较干燥，一碗暖心暖胃的汤菜，更适合这个季节。</span></p><p><img src="https://p3-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/c914c9c326c44693b1166e9484e4b356~noop.image?_iz=58558&from=article.pc_detail&x-expires=1667550729&x-signature=ADEyxanBu%2BSJX%2FfYqIPb%2Fw3WnA0%3D" class="syl-page-img"/></p><p class="pgc-img-caption" style="box-sizing: border-box; margin-top: 20px; padding: 0px; border: 0px; margin-bottom: 0px !important;"><br/></p><p style="box-sizing: border-box; margin-top: 20px; margin-bottom: 20px; padding: 0px; border: 0px; color: rgb(34, 34, 34); font-family: &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, &quot;Helvetica Neue&quot;, Arial, sans-serif; font-size: 18px; text-align: justify; white-space: normal; background-color: rgb(255, 255, 255);">冬瓜排骨汤，是我家秋冬季节经常食用的一道汤菜。冬瓜是一种高钾低钠的蔬菜，食用价值非常高。它含有丰富的蛋白质、碳水化合物、维生素B、C以及钙、铁、磷等矿物质。排骨富含蛋白质和多种微量元素。</p>`;
    this.film = data;
  },
};
</script>

<style lang="scss">
/* 引入第三方富文件功能组件的样式 */
@import url("./u-parse/u-parse.css");
</style>
```

## uni-app环境判断

### 配置文件中使用环境判断

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        // #ifdef H5
        "navigationBarTitleText": "H5 - 首页",
        // #endif
        // #ifdef MP
        "navigationBarTitleText": "MP - 首页",
        // #endif
        // #ifdef APP
        "navigationBarTitleText": "APP - 首页",
        // #endif
        "enablePullDownRefresh": true
      }
    }
  ],
  // ... ...
}
```

### js代码中使用环境判断

```vue
<script>
export default {
  // ... ...
  // 分享给朋友，几乎所有的小程序都支持
  onShareAppMessage() {
    return {
      path: `/subpages/detail/detail?id=${this.id}`, // 可以自定义分享的页面
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事",
    };
  },
  // 分享到朋友圈和添加到收藏夹，目前只有微信小程序支持，因此这里需要做环境判断
  // #ifdef MP-WEIXIN
  onShareTimeline() {
    return {
      query: `?id=${this.id}`, // 分享到朋友圈不支持自定义路径，只能自定义query
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事 -- 朋友圈",
    };
  },
  onAddToFavorites() {
    return {
      query: `?id=${this.id}`, // 添加到收藏夹不支持自定义路径，只能自定义query
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事 -- 收藏",
    };
  },
  // #endif
};
</script>
```

### 模板和样式中使用环境判断

```vue
<template>
  <view class="container">
    <view class="title">我是一个标题</view>
    <!-- 在vue模板中来环境判断 -->
    <!-- #ifdef MP -->
    <view>我是小程序</view>
    <!-- #endif -->
  </view>
</template>

<style lang="scss" scoped>
.title {
  // #ifdef MP
  color: red;
  // #endif
  // #ifndef MP
  color: green;
  // #endif
}
</style>
```

## uni-app中使用vuex

- **/src/main.js**

```js
import App from './App'

// #ifndef VUE3
import Vue from 'vue'
import store from './store'

Vue.prototype.$store = store // 给Vue实例添加一个成员属性$store
Vue.config.productionTip = false
App.mpType = 'app'

const app = new Vue({
  store, // vue2中需要在创建vue实例时传入store
  ...App
})
app.$mount()
// #endif

// 暂时不考虑vue3的情况
// #ifdef VUE3
import { createSSRApp } from 'vue'
export function createApp() {
  const app = createSSRApp(App)
  return {
    app
  }
}
// #endif
```

- **/store/index.js**

```js
import Vue from "vue";
import Vuex from "vuex";
import count from "@/store/modules/count.js";

Vue.use(Vuex);
const store = new Vuex.Store({
  modules: {
    count,
  },
});

export default store;
```

- **/store/modules/count.js**

```js
const countModule = {
  namespaced: true, // 开启命名空间
  state: { // 状态
    num: 1,
  },
  mutations: { // 同步修改数据
    addNum(state, payload) {
      state.num += payload;
    },
  },
  actions: { // 异步请求
    asyncAddNum({ commit }, payload) {},
  },
};

export default countModule;
```

- **/store/pages/category/category.vue**

```vue
<template>
  <view>
    {{ num }}
    <button type="primary" @click="addNum">加一</button>
  </view>
</template>

<script>
export default {
  computed: {
    num() {
      return this.$store.state.count.num;
    },
  },
  methods: {
    addNum() {
      this.$store.commit("count/addNum", 1);
    },
  },
};
</script>
```

## 表单数据收集

```vue
<template>
  <view class="form">
    <label> 姓名：<input v-model="truename" /> </label>
    <label> 手机：<input v-model="phone" /> </label>
    <button @click="saveUser">保存用户信息</button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      truename: "",
      phone: "",
    };
  },
  methods: {
    saveUser() {
      console.log(this.truename, this.phone);
    },
  },
};
</script>
```

## uni-app打包

### 打包为h5

> 1. manifest.json中修改h5配置
>
> 2. hbuilderx上方按钮，点击发行，选择“网站-pc web 或手机h5”，按流程操作，打包后的h5目录会显示在控制台中
>
> 3. uniapp项目必须经过发行打包才可以部署在Nginx上
> 4. nginx配置需要配置项目根目录以及url，如果是未知域名，则需要在hosts文件中将其映射为127.0.0.1

### 发布为小程序

> 1. 在微信公众平台注册小程序
> 2. mainfest.json中修改微信小程序的配置(一定要填写appId)
> 3. 配置完成后，点击发行，发行为“小程序-微信”，按流程操作，打包完成后HBuilderX会提示在小程序开发工具中点击上传，预览 - 生成二维码 - 扫描二维码可以看到效果
> 4. 最后就可以在微信小程序的【后台-开发管理-开发版本】看到提交后的代码，点击提交审核发布即可
> 5. 发布完小程序后，在设置中可以下载小程序二维码进行扫描登陆，或者名称搜索都可以
