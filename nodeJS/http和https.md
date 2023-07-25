# http和https

## get

``` js
// const { get, request } = require('http') // 根据要请求地址的协议选用http和https
const { get, request } = require('https')
// GET 请求获取和前端的AJAX类似，但不需要考虑通信中的跨域
get('https://www.smzdm.com/homepage/headline', async function (result) {
  const data = await getData(result)
  console.log(data)
})

function getData(req) {
  return new Promise((resolve, reject) => {
    let _data = ''
    req.on('data', (_chunk) => _data += _chunk)
    req.on('end', () => resolve(_data))
  })
}
```

### 使用get代理跨域：

- **代理服务器**

``` js
const http = require('http')
const { get } = require('https')
http
  .createServer(async function (req, res) {
    const url = req.url.split('?')[1].split('=')[1]
    res.writeHead(200, {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'text/html;charset=utf-8'
    })
    const data = await httpsGet(url)
    res.end(data)
  })
  .listen(4000)

function getData(req) {
  return new Promise((resolve, reject) => {
    let data = ''
    req.on('data', (_chunk) => (data += _chunk))
    req.on('end', () => resolve(data))
  })
}

function httpsGet(url) {
  return new Promise((resolve, reject) => {
    get(url, async (req) => { // req: http.IncomingMessage
      resolve(await getData(req))
    })
  })
}
```

- **前端**

``` js
const xhr = new XMLHttpRequest()
xhr.addEventListener('load', loadHandler)
// 利用服务端没有跨域限制的特性，让服务器去代理跨域
xhr.open('GET','https://www.smzdm.com/homepage/headline')
xhr.send()
function loadHandler(e) {
  console.log(JSON.parse(xhr.response))
}
```

## request

### 使用request代理跨域

- **代理服务器**

``` js
const http = require('http')

const options = {
  protocol: 'http:',
  hostname: 'localhost',
  method: 'POST',
  port: 4001,
  path: '',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': Buffer.byteLength(postData) // 传输的数据长度
  }
}

http
  .createServer(async (req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/html;charset=utf-8',
      'Access-Control-Allow-Origin': '*'
    })
    const reqData = await getData(req)
    const data = await requestServer(reqData)
    res.end(data)
  })
  .listen(4002)

function requestServer(data) {
  return new Promise(function (resolve, reject) {
    const request = http.request(options, async function (req) { // req: http.IncomingMessage
      const data = await getData(req)
      resolve(data)
    })
    request.write(data) // 与ajax的Post请求类似，需要传入请求体
    request.end() // 即使没有请求体，也得使用end结束请求
  })
}

function getData(req) {
  return new Promise((resolve, reject) => {
    let data = ''
    req.on('data', (_chunk) => (data += _chunk))
    req.on('end', () => resolve(data))
  })
}
```

- **被跨域服务器**

``` js
const http = require('http')
http
  .createServer((req, res) => {
    let data = ''
    req.on('data', (_data) => (data += _data))
    req.on('end', () => {
      console.log(data)
      res.end('hello')
    })
  })
  .listen(4001)
```

- **前端**

``` js
const postData = JSON.stringify({
  province: '上海',
  city: '上海',
  district: '宝山区',
  address: '同济支路199号智慧七立方3号楼2-4层',
  latitude: 43.0,
  longitude: 160.0,
  message: '求购一条小鱼',
  contact: '13666666',
  type: 'sell',
  time: 1571217561
})

const xhr = new XMLHttpRequest()
xhr.addEventListener('load', loadHandler)
xhr.open('POST', 'http://localhost:4001')
xhr.send(postData)
function loadHandler(e) {
  console.log(JSON.parse(xhr.response))
}
```

## get和request的区别

- get方式不含有请求的方式(options)，直接写入地址请求就可以了
- request方式包含详细的请求方式，既可以解决get问题也可以解决post问题
- 如果要发送数据(post)，就需要给请求中设置headers属性，增加Content-Length头部，属性值就是Buffer.byteLength(data)
- request会返回一个对象，只有调用该对象的end方法后才会发送数据

