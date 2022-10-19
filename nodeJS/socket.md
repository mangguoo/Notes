### socket

#### socket

- **client**

```js
const net = require('net')
const readline = require('readline')
const client = new net.Socket()
const order = readline.Interface({
  input: process.stdin,
  output: process.stdout
})
client.connect(4003, 'localhost', connectFinishHandler)
client.on('data', dataHandler)

function connectFinishHandler() {
  order.question('', questionHandler)
}
function questionHandler(msg) {
  client.write(msg)
  order.question('', questionHandler)
}
function dataHandler(data) {
  console.log(data + '')
}
```

- **server**

```js
const net = require('net')
const server = new net.createServer()
const list = {}
let id = 0

server.on('connection', connectionHandler).listen(4003)
function connectionHandler(client) {
  client.id = ++id
  list[id] = client
  client.on('data', (data) => dataHandler(data, client))
  client.on('error', (error) => errorHandler(error, client))
}
function dataHandler(data, client) {
  for (const key in list) {
    // 不给发消息的那个人进行广播
    if (list[key] === client) continue
    list[key].write(data + '')
  }
}
function errorHandler(error, client) {
  console.log(error, client)
}
```

#### websocket

与客户端版的socket不同，nodejs原生不支持websocket，因此需要第三方插件：

```bash
npm i ws -S
```

```bash
const Websocket=require("ws");
var ws=new Websocket.Server({port:4001});
```

基于websocket的点名系统

- **server**

```js
const WebSocket = require('ws')
const server = new WebSocket.Server({ port: 4004 })
const list = new Set()
let id = 0
server.on('connection', connectionHandler)

function connectionHandler(client) {
  client.id = ++id
  client.on('message', (msg) => messageHandler(msg, client))
  client.on('error', (error) => errorHandler(error, client))
  client.on('close', () => closeHandler(client))
}

function messageHandler(msg, client) {
  // 保存client，但是要判断client是否已经被保存，不会重复保存
  if (!client.name && !Array.from(list).find((item) => item.name === msg + '')) {
    client.name = msg + ''
    list.add({ id: client.id, name: client.name, ons: true, client })
  }、
  // 只要用户消息则改变其登录状态
  client.ons = true
  sendClients()
}
function errorHandler(error, client) {
  console.log(error)
}
function closeHandler(client) {
  // 用户离线时改变其登录状态为false
  const item = Array.from(list).find((item) => item.id === client.id)
  if (!item) return
  item.ons = false
  sendClients()
}

function sendClients() {
  for (const item of list) {
    item.client.send(JSON.stringify(Array.from(list).map((item) => ({ id: item.id, name: item.name, ons: item.ons }))))
  }
}
```

- **client**

```js
<ul class="clear"></ul>
<form action="">
  <label for="">姓名</label> <input type="text" name="name" id="" />
  <button>提交</button>
</form>
<script>
  const form = document.querySelector('form')
  const ul = document.querySelector('ul')
  const client = new WebSocket('ws://localhost:4004')
  form.addEventListener('submit', submitHandler)
  client.addEventListener('open', openHandler)
  client.addEventListener('message', messageHandler)

  function submitHandler(e) {
    e.preventDefault()
    client.send(document.querySelector('input').value)
    form.style.display = 'none'
  }

  function openHandler(e) {}

  function messageHandler(e) {
    var arr = JSON.parse(e.data)
    ul.innerHTML = arr.reduce(
      // 通过颜色来区分登录状态
      (v, t) => v + `<li style='background-color:${t.ons ? 'green' : 'red'}'>${t.name}</li>`,
      ''
    )
  }
</script>
```

#### socket.io

- 下载socket.io：socket.io这个插件，既适用于后端，也适用于前端

  ```js
  npm i socket.io
  ```

- 创建服务端socket服务

  ```js
  const socket=require("socket.io")(4001)
  ```

- socket事件：socket.io发送消息是完全自定义的事件

  ```js
  const socket = require('socket.io')(4001)
  socket.on('connection', connectHandler)
  
  function connectHandler(socket) { // 这个socket就是客户端的socket连接
    // 当客户端emit一个message事件时，messageHandler回调就会触发
    socket.on('message', (data) => messageHandler())
  }
  ```

- 客户端的socket.io引入

  ```js
  <script src="../node_modules/socket.io/client-dist/socket.io.js"></script>
  ```

- 创建客户端与服务端的socket连接

  ```js
  const socket = io.connect('localhost:4001')
  ```

- 前端也是根据抛发事件来解决前后端数据的传递，注意前后端的抛发事件和接收事件必须一致

  ```js
  socket.emit("message",handler);
  ```

#### 基于socket.io的小游戏

- **server**

```js
const server = require('socket.io')(4001)
const clients = {}
let id = 0
let list
;(function init() {
  list = Array(100)
    .fill(1)
    .map((item, i) => {
      return {
        id: i,
        x: ~~(Math.random() * (1200 - 50)),
        y: ~~(Math.random() * (700 - 50)),
        speedX: ~~(Math.random() * 5 - 3) || 1,
        speedY: ~~(Math.random() * 5 - 3) || 1
      }
    })
  setInterval(animation, 16)
  server.on('connection', connectionHandler)
})()
function connectionHandler(client) {
  client.id = ++id
  clients[id] = client
  client.on('toserver', serverHandler)
}
function serverHandler(id) {
  list = list.filter((item) => item.id !== id)
}
function animation() {
  for (let i = 0; i < list.length; i++) {
    if (list[i].x < 0 || list[i].x > 1200 - 50) list[i].speedX = -list[i].speedX
    if (list[i].y < 0 || list[i].y > 700 - 50) list[i].speedY = -list[i].speedY
    list[i].x += list[i].speedX
    list[i].y += list[i].speedY
  }
  for (const key in clients) {
    clients[key].emit('toclient', JSON.stringify(list))
  }
}
```

- **client**

```js
<canvas width="1200" height="700"></canvas>
<script src="./node_modules/socket.io/client-dist/socket.io.js"></script>
<script>
  let canvas,
    list = [],
    ctx,
    client
  ;(function init() {
    canvas = document.querySelector('canvas')
    ctx = canvas.getContext('2d')
    client = io.connect('localhost:4001')
    client.on('toclient', messageHandler)
    canvas.addEventListener('click', clickHandler)
  })()
  function messageHandler(_list) {
    list = JSON.parse(_list)
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    for (var i = 0; i < list.length; i++) {
      ctx.save()
      ctx.fillStyle = 'red'
      ctx.translate(list[i].x, list[i].y)
      ctx.beginPath()
      ctx.arc(0, 0, 25, 0, (Math.PI / 180) * 360)
      ctx.fill()
      ctx.restore()
    }
  }
  function clickHandler(e) {
    var id = hitTest(e.offsetX, e.offsetY)
    if (id) client.emit('toserver', id)
  }
  function hitTest(x, y) {
    for (var i = 0; i < list.length; i++) {
      if (x > list[i].x && x < list[i].x + 50 && y > list[i].y && y < list[i].y + 50) return list[i].id
    }
  }
</script>
```
