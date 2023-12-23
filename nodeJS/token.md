# Token

## jsonwebtoken

> josnwebtoken是一个用于生成和验证token的工具

- **jwt.sign**

  ```js
  // 何时返回token  要根据自己的业务逻辑
  router.get("/login", function (req, res) {
    res.json({
      result: "ok",
      token: jwt.sign(
        {
          name: "BinMaing",
          data: "=============",
        },
        global.PrivateKey,
        {
          expiresIn: 60 * 60 * 24, // 24小时过期
        }
      ),
    });
  });
  ```

- **jwt.verify**

  ```js
  app.use(function (req, res, next) {
    // 定义 不用token 的api（unless）
    if (req.originalUrl.indexOf("/getToken") >= 0) {
      return next();
    }
    //定义 用token的api 对其验证
    var token = req.body.token || req.query.token || req.headers["x-access-token"];
    jwt.verify(token, global.PrivateKey, function (err, decoded) {
      if (err) {
        // 返回错误信息
        res.send({
          success: false,
          message:
            "Failed to authenticate token. Make sure to include the " +
            "token returned from /users call in the authorization header " +
            " as a Bearer token",
        });
        return;
      } else {
        // 解析必要的数据（相应字段为定义token时的字段）
        req.username = decoded.name;
        req.data = decoded.data;
        logger.debug(util.format("Decoded from JWT token: username - %s, orgname - %s", decoded.name, decoded.data));
        return next();
      }
    });
  });
  ```

## express-jwt

> express-jwt是nodejs的一个中间件，他来验证指定http请求的JsonWebTokens的有效性，如果有效就将JsonWebTokens的decoded设置到req.auth里面。
>
> express-jwt内部引用了jsonwebtoken，对其封装使用。jsonwebtoken是用来生成token给客户端的，express-jwt是用来验证token的，它的原理就是通过接收请求头中的authorization字段，如果验证通过就把decoded放进req.auth中，否则就抛出一个叫UnauthorizedError的全局错误，可以通过捕获错误中间件来捕获它。

> 设置需要保护的API

```js
var { expressjwt } = require("express-jwt");

global.PrivateKey = "hello  BigManing"; //加密token 校验token时要使用
app.use(
  expressjwt({
    secret: global.PrivateKey,
    algorithms: ["HS256"],
  }).unless({
    path: ["/users/login"], //除了这个地址，其他的URL都需要验证
  })
);
```

> 校验token失败时的处理

```js
app.use(function (err, req, res, next) {
  if (err.name === "UnauthorizedError") {
    //如果验证token失败，返回401
    res.status(401).send("invalid token...");
  }
});
```

>  token过期时的err值：

```js
{
    "name": "UnauthorizedError",
    "message": "jwt expired",
    "code": "invalid_token",
    "status": 401,
    "inner": {
        "name": "TokenExpiredError",
        "message": "jwt expired",
        "expiredAt": "2017-08-03T10:08:44.000Z"
    }
}
```

> token无效时的err值：

```js
{
    "name": "UnauthorizedError",
    "message": "invalid signature",
    "code": "invalid_token",
    "status": 401,
    "inner": {
        "name": "JsonWebTokenError",
        "message": "invalid signature"
    }
}
```

客户端访问被保护的API时，就需要带着token，token 要放到 authorization 这个header里，对应的值以Bearer开头然后空一格，接近着是token值。举例说明：

- **server**

  ```js
  var express = require("express");
  var router = express.Router();
  var jwt = require("jsonwebtoken");
  
  router.get("/info", function (req, res) {
    res.json({
      name: req.auth,
    });
  });
  
  req.auth示例=============>
  {
      "name": "BinMaing",
      "data": "=============",
      "iat": 1501814188,
      "exp": 1501814248
  }
  ```

- **client**

  ```js
  token示例(之所以要写成这样，是因为express-jwt是按这个格式去解析的)：
  authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQmluTWFpbmciLCJkYXRhIjoiPT09PT09PT09PT09PSIsImlhdCI6MTUwMTgxNDE4OCwiZXhwIjoxNTAxODE0MjQ4fQ.GoxGlc6E02W5VvqDNawaOrj3MPO-4UYeFdngKR4bVTE
  ```

## token续签(双token机制)

![2b52ac84196749e0a23e2b2b5e0fbc29](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/2b52ac84196749e0a23e2b2b5e0fbc29.webp)

如图所示，生成两个Token，access_token负责后端业务验证，refresh_token负责续签验证，所以从逻辑上各司其职并不是脱裤子放屁

用户登录时，派发access_token、refresh_token，其中access_token过期时间是1小时，而refresh_token过期时间是2小时，且二者负载内容一致

在后续请求中，access_token未过期则后端正常执行业务，当access_token过期时再验证refresh_token，如果refresh_token未过期则续签，即派发新的access_token、refresh_token到客户端

> 如果客户端没有对登录做防抖，在1s内发送了多个请求到后端，如果按照上面的流程执行，可能会出现客户端突然获取多个新token且层层覆盖的情况，对于后端是一个性能损耗的bug。解决方案如下：
>
> 1. 记录新token的客户端信息，如果记录里面没有发现原始token，则生成新token并记录原始token和新token，再设置过期时间比如3s
> 2. 如果发现了和请求的原始token相同的记录，则直接返回记录里的新token，再重新设置过期时间3s



























