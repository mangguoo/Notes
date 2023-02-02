> 如果它位于反向代理(如Nginx)之后，可以尝试使用来自 ngx_http_core_module 的嵌入式变量

**Nginx 配置文件：**

```txt
....
location / {
   proxy_set_header X-Real-IP $remote_addr;
....
```

**Node.js**

```js
var ip = req.ip || req.headers['x-forwarded-for'] || req.headers["x-real-ip"] ||
req.connection.remoteAddress || 
req.socket.remoteAddress ||
(req.connection.socket ? req.connection.socket.remoteAddress : null)
```