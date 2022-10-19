#### axios.create(config)

> axios.create(config) 对axios请求进行二次封装

这个方法可以根据指定配置创建一个新的 axios ,也就是每个axios 都有自己的配置，新的 axios 只是没有 取消请求 和 批量请求 的方法，其它所有语法都是一致的，那为什么要这种语法？

- 需求，项目中有部分接口需要的配置与另一部分接口的配置不太一样；
- 解决：创建2个新的 axios ，每个都有自己的配置，分别对应不同要求的接口请求中；

```js
const instance = axios.create({
   baseURL:"http://localhost:3000"
})

// 使用instance发请求
instance({
    url:"/posts"
})
instance.get("/posts")
```

