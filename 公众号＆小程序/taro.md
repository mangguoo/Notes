# taro

> Taro 是一个开放式跨端跨框架解决方案，支持使用 React/Vue/Nerv 等框架来开发 微信 / 京东 / 百度 / 支付宝 / 字节跳动 / QQ / 飞书 小程序 / H5 / RN 等应用

## **创建项目**

```js
# 安装 CLI
$ npm install -g @tarojs/cli

# 创建项目
taro init myApp
```

## 脚手架目录结构

```txt
│  .editorconfig
│  .eslintrc
│  .gitignore
│  babel.config.js
│  package-lock.json
│  package.json
│  project.config.json // 项目配置文件，类似于uni-app的mainfest.json(可以在此处配置appId)
│  project.private.config.json //项目私有配置文件。此文件中的内容将覆盖 project.config.json 中的相同字段
│  project.tt.json
├─config // 配置文件
├─dist // 打包后的文件目录
├─src
│  │  app.config.js // 全局页面配置
│  │  app.css // 全局样式
│  │  app.js // 全局入口文件
│  │  index.html // h5端的html模板
│  └─pages
│     └─index
│          index.config.js // 页面配置文件
│          index.css // 页面样式
│          index.jsx // 页面文件
└─types // 类型文件存放处
```

**入口文件(app.js):**

```js
import { Component } from 'react'
import './app.css'

class App extends Component {
  // this.props.children 表示将要会渲染的页面
  render () {
    return this.props.children
  }
}

export default App
```



## 启动项目并开启调试

> taro脚手架提供了多个平台的dev环境,通过指令开启开发环境后，它并不会帮我们自动开启相应平台的开发工具进行调试，需要手动开启开发工具进行导入调试：

```json
"scripts": {
  "build:weapp": "taro build --type weapp",
  "build:swan": "taro build --type swan",
  "build:alipay": "taro build --type alipay",
  "build:tt": "taro build --type tt",
  "build:h5": "taro build --type h5",
  "build:rn": "taro build --type rn",
  "build:qq": "taro build --type qq",
  "build:jd": "taro build --type jd",
  "build:quickapp": "taro build --type quickapp",
  "dev:weapp": "npm run build:weapp -- --watch",
  "dev:swan": "npm run build:swan -- --watch",
  "dev:alipay": "npm run build:alipay -- --watch",
  "dev:tt": "npm run build:tt -- --watch",
  "dev:h5": "npm run build:h5 -- --watch",
  "dev:rn": "npm run build:rn -- --watch",
  "dev:qq": "npm run build:qq -- --watch",
  "dev:jd": "npm run build:jd -- --watch",
  "dev:quickapp": "npm run build:quickapp -- --watch"
}
```

**开启调试(目录为tora项目的根目录)：**

![image-20221031231121154](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/image-20221031231121154.png)

## 新建页面

> taro提供了命令行来帮助我们直接生成页面，但是它不会帮我们把新建的页面添加到配置文件中(不添加到配置文件中不能使用导航进行跳转)

**用命令来创建新页面：**

```js
taro create --name detail
```

- **/src/app.config.js**

```js
// 此文件相当于原生小程序中的app.json文件
export default defineAppConfig({
  pages: [
    'pages/index/index',
    'pages/detail/index'
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'taro学习',
    navigationBarTextStyle: 'black'
  }
})
```

## 网络请求

- **/src/utils/http.js**

```js
import Taro from '@tarojs/taro'

const request = (url, data = null, method = 'GET') => {
  return Taro.request({
    url,
    method,
    data
  })
}

export const get = url => request(url)
export const post = (url, data) => request(url, data, 'POST')
```

- /src/pages/index/index.jsx

```jsx
import { Component } from "react";
// 在taro中，小程序中的所有的组件它都是以 自定义组件的方式来导入使用
// 自定义组件它的首字母要大写
import { View, Image, Navigator } from "@tarojs/components";
import { get } from "@/utils/http.js";
import Taro from '@tarojs/taro'
import "./index.css";

export default class Index extends Component {
  state = {
    films: [],
  };

  async componentDidMount() {
    let ret = await get("https://api.iynn.cn/film/api/v1/getNowPlayingFilmList?cors=T&cityId=110100&pageSize=10&pageNum=1");
    if (ret.data.data.films.length > 0) {
      this.setState({
        films: ret.data.data.films,
      });
    }
  }

  render() {
    return (
      <View>
        {this.state.films.map((item) => (
          <View className="film-item" key={item.filmId}>
            {/* 编程式导航 */}
            <Image
              onClick={() => { 
                Taro.navigateTo({ url: `/pages/detail/index?id=${item.filmId}` });
              }}
              src={item.poster}
            />
            {/* 声明式导航 */}
            <Navigator url={`/pages/detail/index?id=${item.filmId}`}>
              {item.name}
            </Navigator>
          </View>
        ))}
      </View>
    );
  }
}
```

## 环境判断

> taro是通过环境变量来判断环境

```js
if (process.env.TARO_ENV === 'weapp') {
  this.setState({ title: '小程序' })
} else {
  this.setState({ title: 'h5' })
}
```

## 小程序特有生命周期：

```jsx
import { Component } from "react";
import { View, Text } from "@tarojs/components";
import Taro from "@tarojs/taro";
import { get } from '@/utils/http'
import "./index.css";

export default class Detail extends Component {
  state = {
    title: "平台",
    film: null,
  };
  async componentDidMount() {
    // 获取路由参数，也可以直接在onLoad生命周期中，通过options参数获取
    let id = Taro.getCurrentInstance().router.params.id;
    let ret = await get(`https://api.iynn.cn/film/api/v1/getFilmInfo?cors=T&filmId=${id}`);
    this.setState({ // 设置电影数据
      film: ret.data.data.film,
    });
    Taro.setNavigationBarTitle({ // 动态设置标题
      title: ret.data.data.film.name,
    });
  }
  onShareAppMessage() { // 分享给朋友
    return {
      path: `/pages/detail/index?id=${this.id}`,
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事",
    };
  }
  onShareTimeline() { // 朋友圈
    return {
      query: `?id=${this.id}`,
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事 -- 朋友圈",
    };
  }
  onAddToFavorites() { // 收藏
    return {
      query: `?id=${this.id}`,
      imageUrl: "http://img.1314000.cn/face.png",
      title: "一个关于冬瓜的故事 -- 收藏",
    };
  }
  render() {
    return (
      <View className="detail">
        {!this.state.film ? (
          <View>加载中...</View>
        ) : (
          <View>{this.state.film.name}</View>
        )}
        <View>{this.state.title}</View>
      </View>
    );
  }
}
```

## hooks写法

- **/src/pages/detail/index.jsx**

```jsx
import { useState } from "react";
import { View } from "@tarojs/components";
import Taro, {
  usePageScroll,
  useShareAppMessage,
  useShareTimeline,
  useAddToFavorites,
} from "@tarojs/taro";
import "./index.css";
import Loading from "../../components/Loading";
import ShowDetail from "../../components/ShowDetail";
import useFilmInfoHook from "../../hooks/useFilmInfoHook";

const Detail = () => {
  const [title, setTitle] = useState("");
  const [id, film] = useFilmInfoHook();

  useShareAppMessage(() => ({ // 分享
    path: `/pages/detail/index?id=${id}`,
    imageUrl: "http://img.1314000.cn/face.png",
    title: "一个关于冬瓜的故事",
  }));
  useShareTimeline(() => ({ // 朋友圈 
    query: `?id=${this.id}`,
    imageUrl: "http://img.1314000.cn/face.png",
    title: "一个关于冬瓜的故事 -- 朋友圈",
  }));
  useAddToFavorites(() => ({ // 收藏
    query: `?id=${this.id}`,
    imageUrl: "http://img.1314000.cn/face.png",
    title: "一个关于冬瓜的故事 -- 收藏",
  }));
  usePageScroll(({ scrollTop }) => { // 页面滚动
    console.log(scrollTop);
  });

  return (
    <View className="detail">
      {!film ? (
        <Loading />
      ) : (
        <ShowDetail poster={film.poster} title={film.name} />
      )}
      <View>{title}</View>
    </View>
  );
};

export default Detail;
```

- **/src/components/Loading/index.jsx**

```jsx
import React from 'react'
import { View } from '@tarojs/components'

const Loading = () => {
  return <View>加载中...!</View>
}

export default Loading
```

- **/src/componenets/ShowDetail/index.jsx**

```jsx
import React from 'react'
import { View, Image } from '@tarojs/components'

const ShowDetail = ({ poster, title }) => {
  return (
    <View>
      <Image src={poster} />
      <View>{title}</View>
    </View>
  )
}

export default ShowDetail
```

- **/src/hooks/useFilmInfoHook.js**

```js
import { useState, useEffect } from 'react'
import { get } from '../utils/http'
import Taro, { useRouter } from '@tarojs/taro'

const useFilmInfoHook = () => {
  const [data, setData] = useState(null)
  const [id, setId] = useState(null)
  const router = useRouter()
  useEffect(() => {
    let id = router.params.id
    get(`https://api.iynn.cn/film/api/v1/getFilmInfo?cors=T&filmId=${id}`).then(ret => {
      setData(ret.data.data.film) // 保存电影数据
      setId(id) // 保存id
      Taro.setNavigationBarTitle({ // 动态设置标题
        title: ret.data.data.film.name,
      });
    })
  }, [])
  return [id, data]
}

export default useFilmInfoHook
```

