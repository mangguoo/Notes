## 什么是洋葱模型

> 在 Express 里，中间件（middleware） 是一个函数，可以处理请求 req、响应 res，并决定是否将请求传递给下一个中间件。

这些中间件像一层层洋葱一样包裹在请求处理的流程上：

请求从外层进入 → 依次经过每一层中间件 → 到达最内层（路由处理器）。

响应再从最内层出来 → 依次经过中间件 → 返回给客户端。

所以，它既像一个调用栈，又像一颗洋葱，请求和响应都要穿过这些层。

## 洋葱模型执行过程

假设有 3 个中间件：

```js
app.use((req, res, next) => {
  console.log("中间件 A - 进入");
  next();
  console.log("中间件 A - 返回");
});

app.use((req, res, next) => {
  console.log("中间件 B - 进入");
  next();
  console.log("中间件 B - 返回");
});

app.get('/', (req, res) => {
  console.log("路由处理器");
  res.send("Hello Express");
});
```

执行顺序：

```txt
请求进来 →
中间件 A - 进入
  中间件 B - 进入
    路由处理器
  中间件 B - 返回
中间件 A - 返回
→ 响应返回
```

## 洋葱模型的关键点

1. next() 控制流程

- 调用 next()：进入下一层中间件。
- 不调用：请求会停留在当前中间件，不会继续向下。
- 调用 next(err)：跳到错误处理中间件。

2. 顺序很重要

- Express 按照 app.use 或 app.METHOD 的顺序注册和执行。
