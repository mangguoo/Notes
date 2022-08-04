#### CSR

> 客户端渲染页面

![1c4b6f901c61a8cc517b11fe4b169a5e](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/fsdafsdfasdf.jpg)

- ##### 客户端：

  ``` js
  <form target="_self">
    <input type="text" name="user">
    <button type="submit">提交</button>
  </form>
  <div></div>
  
  <script>
    let form, input, div;
    init();
    function init() {
      form = document.querySelector("form")
      input = document.querySelector("input")
      div = document.querySelector("div")
      form.addEventListener("submit", submitHandler)
    }
  
    function submitHandler(e) {
      e.preventDefault()
      ajax("user=" + input.value)
    }
  
    function ajax(param) {
      const xhr = new XMLHttpRequest()
      xhr.addEventListener("load", loadHandler)
      xhr.open("GET", "http://10.9.46.180:4000?" + param)
      xhr.send()
    }
  
    function loadHandler(e) {
      console.log(this);
      div.innerHTML = this.response;
    }
  </script>
  ```

- ##### 服务端：

  ``` js
  const http = require('http')
  http
    .createServer((request, response) => {
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write('<div>你好</div>')
      response.write(`
          <style>
              ul{
                  list-style:none;
                  margin:0;
                  padding:0;
              }
          </style>
      `)
      response.write('<ul>')
      for (let i = 0; i < 10; i++) {
        response.write(`<li><a href='#${i}'>${i}</a></li>`)
      }
      response.write('</ul>')
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  ```

#### SSR

> 服务端渲染页面

![40b8d2d56ddd96f0b58ce286f7f8fa8c](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/40b8d2d56ddd96f0b58ce286f7f8fa8c.jpg)

- ##### 客户端

  ``` js
  <form action="http://10.9.46.180:4000" target="_self">
    <input type="text" name="user">
    <button type="submit">提交</button>
  </form>
  ```

- ##### 服务端

  ``` js
  const http = require('http')
  http
    .createServer((request, response) => {
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write(`<!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Document</title>
            <style>
                div{
                    width: 600px;
                    height: 400px;
                    background-color: red;
                    color: white;
                }
                ul{
                    list-style: none;
                    margin: 0;
                    padding: 0;
                }
                li{
                    float: left;
                }
            </style>
        </head>
        <body>
            <div>
                <ul>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                    <li>1</li>
                </ul>
            </div>
        </body>
        </html>`)
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  ```

#### GET

- ##### 服务端：

  ``` js
  const http = require('http')
  const parseString = require('querystring')
  http
    .createServer((request, response) => {
      const queryStr = request.url.split('?')[1]
      const structured = parseString.parse(queryStr)
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write(`hello, ${structured.user}`)
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  ```

- ##### 客户端：

  ``` js
  const xhr = new XMLHttpRequest()
  xhr.addEventListener("load", loadHandler)
  xhr.open("GET", "http://10.9.46.180:4000?user=ilmango")
  xhr.send()
  
  function loadHandler(event) {
      console.log(this.response)
  }
  ```

#### POST

- ##### 服务端：

  ``` js
  const http = require('http')
  const parseString = require('querystring')
  http
    .createServer(async function (request, response) {
      const structured = await getData(request)
      response.writeHead(200, {
        'Content-Type': 'text/html;charset=utf-8',
        'Access-Control-Allow-Origin': 'http://10.9.46.180:8888'
      })
      response.write(`hello, ${structured.user}`)
      response.end()
    })
    .listen(4000, '10.9.46.180', () => {
      console.log('success')
    })
  
  function getData(request) {
    return new Promise((resolve) => {
      let data = ''
      request.on('data', (_chunk) => {
        data += _chunk
      })
      request.on('end', () => {
        const structured = parseString.parse(data)
        resolve(structured)
      })
    })
  }
  ```

- ##### 客户端：

  ``` js
  const str = "user=ilmango"
  
  const xhr = new XMLHttpRequest()
  xhr.addEventListener("load", loadHandler)
  xhr.open("POST", "http://10.9.46.180:4000")
  xhr.send(str)
  
  function loadHandler(event) {
    console.log(this.response)
  }
  ```

  