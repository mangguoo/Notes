## cookie-session

> 用户session可以通过cookies以下面两种方式存储：存储在客户端或者服务器端。
>
> - 一个模块通过cookie将session值存储在客户端。
> - 一个模块像express-session那样仅将session identifier通过cookie存储在客户端，将session值存储在服务器端，比如数据库中。
>
> 下面几点将帮助你，选择一种去使用：
>
> - cookie-session不需要任何数据库/资源存放在服务器端，只要所有的session值不超过最大的客户端cookie数量限制。
> - cookie-session可以简化特定的负载均衡场景。
> - cookie-session可以使用轻量级的sesseion并包括一个identifier去查询二级存储，从而减少数据库的查询次数。

**安装cookie-session:**

```js
npm install cookie-session
```

### **使用Api(`cookieSession(options)`):**

> 通过提供的选项，创建一个新的cookie session中间件。这个中间件会将session属性依附依附到req上，session属性提供一个对象代表加载的session。这个session可以是一个新的session，如果request中没有提供有效的session，或者是一个在request中已经加载了的session。这个中间件将自动地向response中添加Set-Cookie头，如果req.session的内容被修改了的话。

```js
var cookieSession = require('cookie-session')
var express = require('express')

var app = express()

app.use(cookieSession({
  name: 'session',
  secret: 'secret key',
  //Cookie Options
  maxAge: 24 * 60 * 60 * 1000 // 24 hours
}))

app.get('/', function (req, res, next) {
  // Update views
  req.session.views = (req.session.views || 0) + 1
  // Write response
  res.end(req.session.views + ' views')
})

app.listen(3000)
```

### Options

> Cookie session在options对象中接受下面这些属性。

name：设置cookie的名称，默认是session。

keys：用于加密和解密cookie值的秘钥列表。cookies总是使用keys[0]去加密，而其他的keys用来解密。

secret：如果keys没有提供的话，secret是一个字符串将作为单个的秘钥。

Cookie Options：其他的options可以通过cookies.get()和cookies.set()来使用(当然也可以直接配置)，去控制secure、domain、path和其他设置项。**其中包含如下选项：**

- maxAge：一个代表从现在到过期的数字，以毫秒为单位。
- expires：一个日期对象代表cookie失效的日期。（通常expires在session的最后一个选项）
- path：一个代表cookie路径的字符串。
- domain：一个代表cookie域名的选项（默认没有）
- sameSite：一个boolean值或者字符串代表cookie是否是同一个网站的cookie（默认false）。这个选项可以被设置为’strict’,’lax’或者true（会被映射为’strict’）。
- secure：一个boolean值暗示这个cokkie是否只能在https上传输（默认为false）
- httpOnly：一个boolean值代表cookie是否只能在HTTP(S)上传输，并且不可以通过javascript传输。
- signed：一个boolean值代表这个cookie是否可以被加密。（默认是true）
- overwrite：一个boolean值代表是否可以设置同样name的cookie的值（默认为true）

### req.session

> 它代表request中的session，有如下一些属性：

- .isChanged：如果在请求期间session发生了改变，值为true

- .isNew：如果session是新的，值为true

- .isPopulated：决定这个session是否已经存在或者是空的
- ... ...详见官网

#### req.sessionOptions

> 通过它可以获取当前请求的session设置项

### 删除或修改session

> 删除一个session，只需要把它置为null，而修改一个session，也是只需要对session对象进行操作即可，cookie-session模块将自动帮我们添加set-cookie头

```
req.session = null
```

### 修改默认option

```js
var cookieSession = require('cookie-session')
var express = require('express')

var app = express()

app.use(cookieSession({
  name: 'session',
  keys: ['key1', 'key2']
}))

app.use(function (req, res, next) {
  req.sessionOptions.maxAge = req.session.maxAge || req.sessionOptions.maxAge
  next()
})
```

### 延长会话时间

> 如果会话的内容没有更改，则此模块不发送 Set-Cookie 标头。这意味着要延长用户浏览器中会话的过期时间，需要对会话进行某种修改。
>
> 使用如下配置之后，只要用户操作的频率小于十分钟，就会一直更新session expires，防止用户在使用过程中数据过期。

```js
var cookieSession = require('cookie-session')
var express = require('express')

var app = express()

app.use(cookieSession({
  name: 'session',
  keys: ['key1', 'key2']
}))

app.use(function (req, res, next) {
  req.session.nowInMinutes = Math.floor(Date.now() / 60e3)
  next()
})
```

### 使用自定义签名算法

> **安装keygrip:**
>
> ```js
> npm install keygrip
> ```
>
> **keygrip的Api:**
>
> ```js
> keys = new Keygrip([keylist], [hmacAlgorithm], [encoding])
> ```
>
> 这个类将基于提供的Keylist创建一个新的新keyhold，该键列表用于SHA1 HMAC 进行摘要计算，Keylist是必选的。hmacAlgorithm 默认为“ sha1”，encoding 默认值为“ base64”。

```js
var cookieSession = require('cookie-session')
var express = require('express')
var Keygrip = require('keygrip')

var app = express()

app.use(cookieSession({
  name: 'session',
  keys: new Keygrip(['key1', 'key2'], 'SHA384', 'base64')
}))
```
