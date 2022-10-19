### express

``` js
const express=require("express");
const app=express();
```

#### express通信

##### 路由匹配过程：

每当一个请求到达服务器之后，需要先经过路由的匹配，只有匹配成功之后，才会调用对应的处理函数。在匹配时，会按照路由的顺序进行匹配，如果请求类型和请求的 URL 同时匹配成功，则 Express 会将这次请求，转交给对应的 function 函数进行处理。

![20211008202733708](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/20211008202733708.png)

> 路由匹配的注意点：
>
> 按照定义的先后顺序进行匹配。
> 请求类型和请求的URL同时匹配成功，才会调用对应的处理函数。

##### app.get   app.post

``` js
app.get("/a",function(req,res){
    res.send("hello")
})

app.post("/a",function(req,res){
    res.send("hello")
})

app.listen(4001);
```

##### app.put   app.delete   app.all

1、GET请求会向数据库发索取数据的请求，从而来获取信息，该请求就像数据库的select操作一样，只是用来查询一下数据，不会修改、增加数据，不会影响资源的内容，即该请求不会产生副作用。无论进行多少次操作，结果都是一样的。

2、与GET不同的是，PUT请求是向服务器端发送数据的，从而改变信息，该请求就像数据库的update操作一样，用来修改数据的内容，但是不会增加数据的种类等，也就是说无论进行多少次PUT操作，其结果并没有不同。

3、POST请求同PUT请求类似，都是向服务器端发送数据的，但是该请求会改变数据的种类等资源，就像数据库的insert操作一样，会创建新的内容。几乎目前所有的提交操作都是用POST请求的。

4、DELETE请求顾名思义，就是用来删除某一个资源的，该请求就像数据库的delete操作。

> app.all是不管哪种请求都可以接收

``` js
const express = require('express')
const app = express()

app.all('*', function (req, res) { // 匹配任何方式的任何路由
  res.send('aaa')
})

app.all('/a', function (req, res) { // 匹配任何方式的/a路由
  res.send('hello')
})

app.listen(4001)
```

#### request的部分属性

##### req.query：

> 获取get通过地址传参方式传过来的参数

``` js
app.get("/a",function(req,res){
    console.log(req.query);
    res.send("hello")
})
```

##### req.body：

>  获取post通过地址传参方式传过来的参数,但是node并不会自动解析传过来的数据，而是需要使用一个express内置的中间件：

- 处理x-www-form-urlencode和json

``` js
const express = require('express')
const app = express()
// parse application/x-www-form-urlencode
app.use(express.urlencoded({ extended: true }))
// parse application/json
app.use(express.json())

app.post("/a",function(req,res){
    console.log(req.body);
    res.send("hello")
})
app.listen(4001);
```

``` js
(async function init() {
  let res = await fetch('http://localhost:4004/e', {
    headers:{"Content-Type":"application/x-www-form-urlencoded"},
    body: "a=1&b=2",
    headers:{"Content-Type":"application/json"},
    body: JSON.stringify({a:1,b:2})
    method: 'post'
  })
  res = await res.text()
  console.log(res)
})()
```

- 处理form-data

```js
# 安装multer
npm i multer -S
```

``` js
var express=require("express");
var app=express();
const multer=require("multer")();//载入并且执行函数返回对象
// parse application/formdata
app.use(multer.none());

app.post("/a",function(req,res){
    console.log(req.body);
    res.send("hello")
})
app.listen(4001);
```

```js
;(async function init() {
  const fd = new FormData()
  fd.set('name', 'xietian')
  fd.set('age', 30)
  let res = await fetch('http://localhost:4004/e', {
    method: 'post',
    body: fd
  })
  res = await res.text()
  console.log(res)
})()
```

##### req.path：获取当前路径

``` js
var express=require("express");
var app=express();

app.get("/a",function(req,res){
    console.log(req.path);//path 是/a
    res.send("hello")
})
app.listen(4001);
```

#### response的部分属性

##### res.send：

>  用于发送数据给前端

``` js
app.get("/a",function(req,res){
    res.send(Buffer.from('whoop'))//可以发送Buffer
    // res.send({ some: 'json' })//可以直接发送Json
    // res.send('<p>some html</p>')//可以发送字符
})
```

##### res.status：

>  返回状态码并且发送

``` js
app.get("/a",function(req,res){
    res.status(404).send('Sorry, we cannot find that!')//可以设置响应的状态的同事发送
})
```

##### req.set：

> 设置响应头

``` js
app.get("/a",function(req,res){
   res.set({
       'Access-Control-Allow-Origin':'*',
       'Content-Type':'text/html;charset=utf-8',
   });
   res.set("Set-Cookie","sessionId=xietian; Path=/; HttpOnly");
   res.send('aaa')
})
```

##### res.type：

>  设置响应头消息的MIME类型

``` js
app.get("/a",function(req,res){
    res.type("json");
    res.send({a:1,b:2});
})
```

##### res.json：

>  直接设置响应消息的MIME类型是json,并且将对象用json.stringfy转换直接发送

``` js
app.get("/a",function(req,res){
    res.json({a:1,b:2});
})
```

##### res.jsonp：

> 直接发送jsonp消息

``` js
app.get("/a",function(req,res){
    //返回的消息在callback函数中调用
    res.jsonp({a:1,b:2});
})
```

#### express的中间件

app.use是express“调用中间件的方法”。所谓“中间件，就是处理HTTP请求的函数，用来完成各种特定的任务，比如检查用户是否登录、分析数据、以及其他在需要最终将数据发送给用户之前完成的任务。

简而言之，app.use() 里面使用的参数，主要是函数。但这个使用，并不是函数调用，而是使能的意思。当用户在浏览器发出请求的时候，这部分函数才会启用，进行过滤、处理。

express的路由，如app.get()，app.post()，app.all()。。。但其实，它们都是app.use的别名。

注意use后面的第二个函数中有3个参数，第一个是req，第二个是res，第三个是next 我们在use中处理完内容后，必须执行next()这样才可以进入后续的内容，否则将不会向后继续执行。

> 在express中间件分为5种:
>
> 1、自定义中间件
>
> 2、路由中间件
>
> 3、静态中间件
>
> 4、第三方中间件
>
> 5、错误处理中间件

##### 自定义中间件

``` js
// 与get、post等方法不同，use的 ‘/’ 路由表示匹配所有路由
app.use("/",function(req,res,next){
    res.set({'Access-Control-Allow-Origin':'*'});
    next();
})
// 不写路由参数也表示匹配所有路径
app.use(function(req,res,next){
    res.set({'Access-Control-Allow-Origin':'*'});
    next();
})
```

> 特殊写法

``` js
var listHandler=[function(req,res,next){
    res.set({
        'Access-Control-Allow-Origin':'*',
    });
    console.log("a");
    next();
},function(req,res,next){
    console.log("b");
    next();
},function(req,res,next){
    console.log("c");
    next();
}]
app.use("/a",listHandler)

app.use("/a",function(req,res,next){
    res.set({
        'Access-Control-Allow-Origin':'*',
    });
    console.log("a");
    next();
},function(req,res,next){
    console.log("b");
    next();
},function(req,res,next){
    console.log("c");
    next();
})
```

##### 路由中间件

> main.js

```js
var express=require("express");
var Router1=require("./router1")
var Router2=require("./router2")
var app=express();

app.use("/user",Router1);
app.use("/goods",Router2);

app.listen(4005);
```

> router1.js

```js
var express=require("express");
var Router=express.Router();

Router.get("/login",function(req,res){
    res.end("111");
})
Router.get("/logout",function(req,res){
    res.end("222");
})
Router.get("/signup",function(req,res){
    res.end("333");
})

module.exports=Router;
```

> router2.js

```js
var express=require("express");
var Router=express.Router();

Router.get("/addgoods",function(req,res){
    res.end("ccc");
})
Router.get("/removegoods",function(req,res){
    res.end("ddd");
})

module.exports=Router;
```

##### 第三方中间件

- > 使用cros中间件处理跨域

  ``` js
  npm install cros
  ```

  ``` js
  var cors=require("cors");
  app.use(cors());
  ```

- > 使用multer处理form-data请求

  ``` js
  npm install multer
  ```

  ``` js
  const multer=require("multer")();
  app.use(multer.none());
  ```

##### 内置中间件

- express.static：静态处理，可以快速将静态文件返回给前端

  ```js
  app.use('/public',express.static("./public"));
  ```

- express.urlencoded：处理search请求体

  ```js
  app.use(express.urlencoded({ extended: true }))
  ```

- express.json：处理json请求体

  ```js
  app.use(express.json())
  ```

##### 错误处理中间件

错误处理中间件有 4 个参数，定义错误处理中间件时必须使用这 4 个参数。即使不需要 next 对象，也必须在签名中声明它，否则中间件会被识别为一个常规中间件，不能处理错误。

错误处理中间件和其他中间件定义类似，只是要使用 4 个参数，而不是 3 个，其签名如下： (err, req, res, next)。

``` js
app.use(function(err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

#### express操作cookie

##### 设置cookie：

>  express直接提供了api,只需要在需要使用的地方调用如下api即可

```js
app.use('/', function(req, res, next){
    res.cookie(name, value [, options]);
})
```

这样express就会将其填入Response Header中的Set-Cookie，达到在浏览器中设置cookie的作用。res.cookie需要加入如下参数，其它options为可选参数：

- name: 类型为String

- value: 类型为String和Object，**如果是Object会在cookie.serialize()之前自动调用JSON.stringify对其进行处理**

- Options: 类型为对象，可使用的属性如下：

  > - domain：cookie在什么域名下有效，类型为String，默认为网站域名
  >
  > - expires: cookie过期时间，类型为Date。如果没有设置或者设置为0，那么该cookie只在这  个这个session有效，即关闭浏览器后，这个cookie会被浏览器删除
  >
  > - httpOnly: 只能被web server访问，类型Boolean
  >
  > - maxAge: 实现expires的功能，设置cookie过期的时间，类型为String，指明从现在开始，     多少毫秒以后，cookie到期
  >
  > - path: cookie在什么路径下有效，默认为'/'，类型为String
  >
  > - secure：只能被HTTPS使用，类型Boolean，默认为false
  >
  > - signed:使用签名，类型Boolean，默认为false。如果为true，那么express会使用req.secret来完成签名，需要cookie-parser配合使用

##### 删除cookie：

>  express直接提供了api删除浏览器中的cookie

```js
res.clearCookie(name [, options]);
```

##### 获取cookie：

> cookie的推送和删除都可以使用express自带的api，不过要是想用express获取cookie就需要cookie-parser中间件帮忙了。

- 引用cookie-parser

```js
npm install cookie-parser --save
```

- 使用cookie-parser

如果没有使用签名，cookie保存在req.cookies中，而如果使用了signed签名的cookie保存在req.signedCookies中。

```js
var express = require('express');
var cookieParser = require('cookie-parser');
var app = express();

//若需要使用签名，需要指定一个secret,字符串,否者会报错
app.use(cookieParser('secret'));

app.use(function (req, res, next) {
  console.log(req.cookies.nick); // undefined
  console.log(req.signedCookies.nick); // chyingp
  console.log(req.cookies.a); // chyingp
  console.log(req.signedCookies.a); // undefined
  next();
});

app.use(function (req, res, next) {  
  // 传入第三个参数 {signed: true}，表示要对cookie进行摘要计算
  res.cookie('nick', 'chyingp', {signed: true});
  res.cookie('a', 'chyingp');
  res.end('ok');
});

app.listen(3000);
```

#### express-generator

##### 下载

``` js
npm i express-generator -g
```

##### 生成脚手架

``` js
express --view=ejs 
或者
express -e
```