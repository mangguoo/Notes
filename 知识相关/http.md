# http缓存

> 在浏览器向服务器请求资源的时候，许多静态的资源是长时间不会更改的。如果重复的发起请求，不断**从服务器获取**长时间不会改变的资源，不仅会浪费宽带，增加服务器的压力，甚至影响到用户的体验。通过http缓存就能设置资源缓存。缓存后能减少资源的请求、减少响应的时间

- 客户端向服务器发起请求时，会根据HTTP响应头的字段加载缓存的资源
- **http 缓存从第二次开始**。第一次请求从服务器加载所有资源，第二次请求浏览器根据响应头的字段加载缓存资源
- http 缓存分为 **强缓存** 与 **协商缓存**。**无论是命中哪个缓存资源都是从客户端本地加载，不同的是协商缓存会向服务器发起请求，强缓存不会向服务器发起请求**

## **强缓存(Cache-Control)**

**强制缓存在客户端本地，请求直接从本地缓存中加载，不去请求服务器，返回的状态码是200**。强缓存需要`Cache-Control`字段，`Cache-Control`是HTTP响应头中的字段。当我们向服务器请求资源时，服务器认为需要被缓存的资源比如（css，js文件 、图片），这些资源在响应头中都会默认携带**`Cache-Control: public, max-age=0`** 。public, max-age=0是默认值（相当于没有对资源缓存进行处理）

**`Cache-Control`属性值**

1. public：这些资源可以被任意对象缓存（如：客户端，代理服务器，等等）
2. private：这些资源只能在客户端缓存
3. max-age=xx：设置资源缓存的最大时间，超过时间表示缓存过期，请求会发到服务器（单位：秒）
4. s-maxage=xx：只用于共享缓存，比如：CDN缓存（s -> share）(`s-maxage` 用于代理缓存)
5. no-cache：加载缓存资源前，强制发送请求到服务器进行“协商缓存”
6. no-store：不被做任何缓存
7. must-revalidate：使用缓存之前必须验证缓存资源的状态(进行协商)

### Expires

- 与cache-control字段相似的还有一个字段：expires

- 他已经被cache-control所取代，而为了兼容所以被保留了下来

- 他表示一个网页或url地址不再被浏览器缓存的时间，一旦超过了这个时间，浏览器就会从服务器去重新请求资源

## **协商缓存(Etag)**

**请求的资源通过资源标识与服务器协商比对，协商成功，服务器返回304状态码，浏览器从本地加载**。协商缓存需要用到`Etag`字段与`if-none-match`，**Etag是HTTP响应头中的字段，Etag的值是根据资源内容编码生成的一段字符串（资源标识），内容不同就会生成不同的Etag**

再次发起请求时，请求头会带有`if-none-match`字段，值为上一次响应的Etag（没有则不会携带）。服务器会将请求的资源生成资源标识与发送过来的值进行比对，比如果比对成功则返回304状态码，浏览器从本地加载该资源

如果协商失败服务器会返回新的资源+新的Etag（资源标识）

> 补充：如果不想命中协商缓存与强缓存，win系统可以直接`ctrl+f5`或`ctrl+shift+r`

## 协商缓存(Last-Modified)

- HTTP1.0中通过Pragma字段控制页面缓存，通常设置为`no-cache`并加上`expires: 0`（立即过期，下次再用时去服务端拿）
- HTTP1.1中启用Cache-Control来控制页面是否缓存
- 而我们在使用webpack打包后的项目，js和css文件的文件名通常都有文件hash，这样只要资源改变了，再打包时文件名就会发生变化，而HtmlWebpackPlugin会在生成index.html时帮我们把新的js与css引入
- 这样我们就只需要保证用户请求到的html文件是最新的，那么他拿到的js和css就也是最新的

```nginx
{
  location / {
    index index.html index.htm;
    try_files $uri /index.html;
    if ($request_filename ~* .*\.(?:htm|html)$) {
      add_header Cache-Control "no-cache, must-revalidate";
	  	add_header "Pragma" "no-cache";
	 		add_header "Expires" "0";
    }
    if ($request_filename ~* .*\.(?:js|css)$) {
        expires 7d;
    }
    if ($request_filename ~* .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$) {
        expires 7d;
    }
  }
}
```

客户端第一次请求一个URL，服务器返回状态是200，同时有一个`Last-Modified`报头的属性标记

```txt
Last-Modified:Tue, 24 Feb 2019 08:01:04 GMT
```

客户端第二次请求此URL，浏览器会向服务器传送`If-Modified-Since`报头，询问该时间是否被修改过。如果服务器资源没有变化，自动返回304，内容为空，客户端直接从缓存中取内容即可；如果资源有变化，则向客户端发送最新资源，并更新`Last-Modified`

```txt
If-Modified-Since:Tue, 24 Feb 2019 08:01:04 GMT
```

## must-revalidate

在HTTP客户端（浏览器或者缓存服务器）上，如果某个URL对应的缓存过期了，客户端会再次向该URL发送一个条件请求（带有`If-Modified-Since`/`If-None-Match`请求头），如果服务端（缓存服务器或者源站）返回的状态码是`304`（没有响应体），则客户端会根据该`304`响应所包含的一些响应头（`Date`、`Last-Modified`、`Cache-Control`等）重新计算出这条缓存的过期时间，比如：

```text
HTTP/2 304
Cache-Control: max-age=86400
```

这样的过程可以归结为强缓存过期了，对服务器发起协商缓存，然后协商缓存成功(`304`)，就能让这条缓存重新续命一天

如果返回的状态码是`200`，则整条缓存会被新返回的响应体替换掉

无论是哪种情况，这条缓存都重新变的有效了，HTTP规范里把这一“让过期的缓存重新变的有效”过程，叫做revalidate。简单来说就是再次校验看看缓存是不是真的过期了，真过期了的话返回`200`，假过期（客户端判断为过期了--强缓存过期，但服务端说并没有--资源未更新，命中协商缓存）的话返回`304`

revalidate是个常见的动作，缓存过期(强缓存)就会revalidate，那`must-revalidate`是用来做什么的？

```txt
Cache-Control: max-age=86400, must-revalidate
```

写这个配置可能是想表达：该缓存有效期为一天，在这一天内，每次使用缓存前要先校验一遍才能使用。可试试就知道了，这里的`must-revalidate`并不会生效，这条缓存仍然是直接读取了本地。这是因为`must-revalidate`生效有个前提，前提就是这个缓存必须已经过期，也就是说，必须一天以后，这个`must-revalidate`才可能发挥作用

之所以需要`must-revalidate`，是因为HTTP规范是允许客户端在某些特殊情况下直接使用过期缓存的，比如校验请求发送失败的时候，还比如有配置一些特殊指令（`stale-while-revalidate`、`stale-if-error`等）的时候

## Keep Alive

> 这个技术可以帮助我们对TCP链接进行复用，也就是说当我们和一台服务器进行TCP建立连接之后，接下来的请求就都不需要重复建立链接。Nginx默认开启`keep-alive`

Initial connection为 TCP链接的建立，后续资源加载就没有Initial connection

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/461c1dc07e23da2dbec0c83d0f488438.png)

可以在Request Headers中看到`keep-alive`参数，表示他是复用已创建的tcp连接：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/0b936e48c27e60c766a4dc5bcba1f37a.png)

```nginx
http {
  keepalive_timeout  65; // 超时时间，65s内没使用TCP链接就会断掉
  keepalive_requests 100; // 客户端和服务端进行TCP链接后，会开始计数，第101个请求就需要重新建立TCP链接
}
```

