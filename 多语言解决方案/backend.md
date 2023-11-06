# backend

> 这是一个i18next的中间件，它的作用就是使用XMLHttpRequest或fetch API从后端服务器中加载语言资源，可以用于Node.js、浏览器和Deno

## 安装与使用

```shell
$ npm install i18next-http-backend
```

```js
import i18next from 'i18next';
import HttpApi from 'i18next-http-backend';

i18next.use(HttpApi).init(i18nextOptions);
```

## backend选项

### 默认参数

```json
{
  // 定义请求语言资源的路径
  loadPath: '/locales/{{lng}}/{{ns}}.json',
  // 定义其它语言文件资源的路径，它将被用作去请求缺失的资源
  addPath: '/locales/add/{{lng}}/{{ns}}',
  // 是否支持多重加载
  allowMultiLoading: false,
  // 在获取数据之后解析数据
  // 它可以被理解为JSON.parse方法的第二个参数(解析函数)，这个解析函数的参数就是解析后对象的value
  parse: function(data) { return data.replace(/a/g, ''); },
  // 在向addPath发送请求之前解析数据
  parsePayload: function(namespace, key, fallbackValue) { return { key } },
    
  // 允许跨域
  crossDomain: false,
  // 允许用户凭据跨域
  withCredentials: false,
  // overrideMimeType相当于request.overrideMimeType("application/json")
  overrideMimeType: false,
  // 设置自定义头部，相当于request.setRequestHeader(key, value)
  customHeaders: {
    authorization: 'foo',
    // ...
  },
  // customHeaders也可以被指定为一个函数
  customHeaders: () => ({
    authorization: 'foo',
    // ...
  }),
  // 用于fetch API的请求参数，也可以是一个函数：function (payload) => ({ method: 'GET' })
  requestOptions: {
    mode: 'cors',
    credentials: 'same-origin',
    cache: 'default'
  }
  // 自定义请求函数
  // ‘options’就是这个完整的options对象
  // ‘url’就是‘loadPath’的值
  // ‘payload’将是一个key:value对象，用于保存缺失的翻译
  // ‘callback’是一个接受两个参数(‘err’和‘res’)的函数
  // ‘err’是一个错误
  // ‘res’是一个具有‘status’属性和‘data’属性的对象，该属性包含一个字符串化的对象实例作为请求的lng和ns，或者在出错时为null
  request: function (options, url, payload, callback) {},
  // 给url添加参数. 'example.com' -> 'example.com?v=1.3.5'
  queryStringParams: { v: '1.3.5' },
  // 可用于在特定的时间间隔内重新加载资源(在服务器环境中很有用)
  reloadInterval: false
}
```

### 自定义选项

```js
import i18next from 'i18next';
import HttpApi from 'i18next-http-backend';

i18next.use(HttpApi).init({
  backend: options,
});
```

```js
import HttpApi from 'i18next-http-backend';
const HttpApi = new HttpApi(null, options);
```

```js
import HttpApi from 'i18next-http-backend';
const HttpApi = new HttpApi();
HttpApi.init(null, options);
```
