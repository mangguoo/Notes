##### mime.define

mime自定义类型，当mime模块自带的mime-db库不存在或不能满足我们所需的MIME类型时，还可以自定义MIME类型

``` js
mime.define({
  'text/mytext': ['t-txt', 't-ext', 't-xt']
})
const type = mime.getExtension('text/mytext')
console.log(type)
```

