#### URL

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/url.jpg)

```bash
const { URL } = require('url')
const urlString = 'https://www.baidu.com:443/ad/index.html?id=8&name=mouse#tag=110'
const parsedStr = new URL(urlString)
console.log(parsedStr)

URL {
  href: 'https://www.baidu.com/ad/index.html?id=8&name=mouse#tag=110',
  origin: 'https://www.baidu.com',
  protocol: 'https:',
  username: '',
  password: '',
  host: 'www.baidu.com',
  hostname: 'www.baidu.com',
  port: '',
  pathname: '/ad/index.html',
  search: '?id=8&name=mouse',
  searchParams: URLSearchParams { 'id' => '8', 'name' => 'mouse' },
  hash: '#tag=110'
}
```

