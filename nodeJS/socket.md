# socket

## 轮询

> 客户端和服务器之间会一直进行连接，每隔一段时间客户就会主动发送请求给服务器端询问一次。这种方式连接数会很多，而且每次发送请求都会有Http的Header头，会很耗流量，需要服务器有很快的处理速度和资源

- **server**

```js
let express = require('express');
let app = express();
app.use(express.static('www'));
app.get('/', (req,res) => {
    res.send('' + Date.now());
});
app.listen(3000);
```

- **client**

```html
<div id="root"></div>
<script>
setInterval(function () {
	let xhr = new XMLHttpRequest();
	xhr.open('GET', 'http://localhost:3000/', true);
	xhr.onreadystatechange = function () {
		if (xhr.readyState == 4 && xhr.status == 200) {
			document.querySelector('#root').innerHTML = xhr.responseText;
		}
	}
	xhr.send();
}, 3000);
</script>
```

## 长轮询

> 长轮询是对轮询的改进版，客户端发送HTTP给服务器之后，看有没有新消息，如果没有新消息，就一直等待,当有新消息的时候，才会返回给客户端。在某种程度上减小了网络带宽问题。但是他需要服务器有很高的并发，也就是同时接待客户的能力

- **server**

```js
let express = require('express');
let app = express();
app.use(express.static('www'));
app.get('/', (req, res) => {
  let timer = setInterval(() => {
    let seconds = new Date().getSeconds()
    if (seconds == 50) {
      res.send('' + Date.now())
      clearInterval(timer)
    }
  }, 1000)
})
app.listen(3000);
```

- **client**

```html
<div id="root"></div>
<script>
	(function send() {
		let xhr = new XMLHttpRequest();
		xhr.open('GET', 'http://localhost:3000', true);
		xhr.onreadystatechange = function () {
			if (xhr.readyState == 4 && xhr.status == 200) {
				document.querySelector('#root').innerHTML = xhr.responseText;
				send();
			}
		}
		xhr.send();
	})();
</script>
```

## socket

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

## websocket

> 从上面可以看出其实这两种方式，都是在不断地建立HTTP连接，然后等待服务端处理，可以体现HTTP协议的另外一个特点，被动性
>
> HTTP 还是一个无状态协议。通俗的说就是，服务器因为每天要接待太多客户了，是个健忘鬼，你一挂电话，他就把你的东西全忘光了，把你的东西全丢掉了。你第二次还得再告诉服务器一遍
>
> 所以在这种情况下出现了 WebSocket 。他解决了 HTTP 的这几个难题。首先，被动性，当服务器完成协议升级后（HTTP->Websocket），服务端就可以主动推送信息给客户端了

### http与websocket的主要区别

![aHR0cHM6Ly93d3cuamF2YXpoaXlpbi5jb20vd3AtY29udGVudC91cGxvYWRzLzIwMjAvMDcvamF2YTctMTU5NTE0MjA3Ni5wbmc](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/aHR0cHM6Ly93d3cuamF2YXpoaXlpbi5jb20vd3AtY29udGVudC91cGxvYWRzLzIwMjAvMDcvamF2YTctMTU5NTE0MjA3Ni5wbmc.png)

### websocket的特点

下面是一个典型的 WebSocket 握手

```bash
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

可以发现这段类似 HTTP 协议的握手请求中，多了这么几个东西

```vbnet
Upgrade: websocket
Connection: Upgrade
```

这个就是 WebSocket 的核心了，告诉 Apache 、 Nginx 等服务器：注意啦，我发起的请求要用 WebSocket 协议，快点帮我找到对应的助理处理

```vbnet
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

首先， Sec-WebSocket-Key 是一个 Base64 encode 的值，这个是浏览器随机生成的，用于验证服务器是不是真的是 WebSocket 助理

然后， Sec_WebSocket-Protocol 是一个用户定义的字符串，用来区分同 URL 下，不同的服务所需要的协议

最后， Sec-WebSocket-Version 是告诉服务器所使用的 WebSocket Draft （协议版本），在最初的时候，WebSocket 协议还在 Draft 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么 Firefox 和 Chrome 用的不是一个版本之类的，当初 WebSocket 协议太多可是一个大难题。。不过现在还好，已经定下来了，大家都使用同一个版本13。然后服务器会返回下列东西，表示已经接受到请求， 成功建立 WebSocket连接了

```vbnet
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

这里开始就是 HTTP 最后负责的区域了，告诉客户，我已经成功切换协议了

```vbnet
Upgrade: websocket
Connection: Upgrade
```

依然是固定的，告诉客户端即将升级的是 WebSocket 协议，而不是 mozillasocket，lurnarsocket 或者 shitsocket

然后， Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后的 Sec-WebSocket-Key 。这是服务器为了告诉客户端它是websocket助理

后面的， Sec-WebSocket-Protocol 则是表示最终使用的协议

至此，HTTP 已经完成它所有工作了，接下来就是完全按照 WebSocket 协议进行了

### 使用websocket

```bash
npm i ws -S
```

```bash
const Websocket=require("ws");
var ws=new Websocket.Server({port:4001});
```

### 基于websocket的点名系统

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
  // 只要收到用户消息则改变其登录状态
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

## socket.io

> Socket.IO是一个WebSocket库，可以同时适用于浏览器环境和nodejs环境，它会自动根据浏览器从WebSocket、AJAX长轮询、Iframe流等等各种方式中选择最佳的方式来实现网络实时应用，非常方便和人性化，而且支持的**浏览器最低达IE5.5**

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

### 基于socket.io的小游戏

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

## WebSocket心跳重连

> WebSocket 最大的优势就是能够保持前后端消息的长连接，但是在某些情况下，长连接失效并不会得到及时的反馈，前端并不知道连接已断开。例如用户网络断开，并不会触发 websocket 的任何事件函数，这个时候如果发送消息，消息便无法发送出去，浏览器会立刻或者一定短时间后（不同浏览器或者浏览器版本可能表现不同）触发 onclose 函数
>
> 为了避免这种情况，保证连接的稳定性，前端需要进行一定的优化处理，一般采用的方案就是心跳重连。前后端约定，前端按一定间隔发送一个心跳包，后端接收到心跳包后返回一个响应包，告知前端连接正常。如果一定时间内未接收到消息，则认为连接断开，前端进行重连
>
> **心跳重连：**通过以上分析，可以得到实现心跳重连的关键是按时发送心跳消息和检测响应消息并判断是否进行重连，所以首先设置 4 个小目标：
>
> - 可以按一定间隔发送心跳包
> - 连接错误或者关闭时能够自动重连
> - 若在一定时间间隔内未接收消息，则视为断连，自动进行重连
> - 可以自定义心跳消息并设置最大重连次数

将 WebSocket 管理封装为一个工具类 WebsocketHB，通过传入配置对象来自定义心跳重连机制

```js
class WebsocketHB {
  constructor({
    url, // 连接客户端地址
    pingTimeout = 8000, // 发送心跳包间隔，默认 8000 毫秒
    pongTimeout = 15000, // 最长未接收消息的间隔，默认 15000 毫秒
    reconnectTimeout = 4000, // 每次重连间隔
    reconnectLimit = 15, // 最大重连次数
    pingMsg // 心跳包的消息内容
  }) {
    // 初始化配置
    this.url = url
    this.pingTimeout = pingTimeout
    this.pongTimeout = pongTimeout
    this.reconnectTimeout = reconnectTimeout
    this.reconnectLimit = reconnectLimit
    this.pingMsg = pingMsg
    // 实例变量
    this.ws = null
    this.pingTimer = null // 心跳包定时器
    this.pongTimer = null // 接收消息定时器
    this.reconnectTimer = null // 重连定时器
    this.reconnectCount = 0 // 当前的重连次数
    this.lockReconnect = false // 锁定重连
    this.lockReconnectTask = false // 锁定重连任务队列
	// 创建webSocket连接
    this.createWebSocket()
  }

  createWebSocket() {
    try {
      this.ws = new WebSocket(this.url)
      this.ws.onclose = () => {
        this.onclose()
        this.readyReconnect()
      }
      this.ws.onerror = () => {
        this.onerror()
        this.readyReconnect()
      }
      this.ws.onopen = () => {
        this.onopen()
        this.clearAllTimer()
        this.heartBeat()
        this.reconnectCount = 0
        // 解锁，可以重连
        this.lockReconnect = false
      }
      this.ws.onmessage = event => {
        this.onmessage(event)
        // 每次接收消息时，清除最长间隔消息重连定时器，能接收消息说明连接正常，不需要重连
        clearTimeout(this.pongTimer)
        this.pongTimer = setTimeout(() => {
          this.readyReconnect()
        }, this.pongTimeout)
      }
    } catch (error) {
      console.error('websocket 连接失败!', error)
      throw error
    }
  }

  // 发送心跳包
  heartBeat() {
    // 这里使用 setTimeout 模拟 setInterval 定时发送心跳包，避免定时器队列阻塞
    this.pingTimer = setTimeout(() => {
      this.send(this.pingMsg)
      this.heartBeat()
    }, this.pingTimeout)
  }

  // 准备重连
  readyReconnect() {
    // 避免循环重连，当一个重连任务进行时，不进行重连
    if (this.lockReconnectTask) return
    this.lockReconnectTask = true
    this.clearAllTimer()
    this.reconnect()
  }

  // 重连
  reconnect() {
    if (this.forbidReconnect) return
    // 需要注意的是每次进行重连时加锁，避免进行无效重连
    if (this.lockReconnect) return
    // 并且限制最大重连次数
    if (this.reconnectCount > this.reconnectLimit) return
    // 加锁，禁止重连
    this.lockReconnect = true
    this.reconnectCount += 1
    this.createWebSocket()
    this.reconnectTimer = setTimeout(() => {
      // 解锁，可以重连
      this.lockReconnect = false
      this.reconnect()
    }, this.reconnectTimeout)
  }}

  // 清空所有定时器
  clearAllTimer() {
    clearTimeout(this.pingTimer)
    clearTimeout(this.pongTimer)
    clearTimeout(this.reconnectTimer)
  }

  // 销毁 ws
  destroyed() {
    // 在实例销毁的时候设置一个禁止重连锁，避免销毁的时候还在尝试重连，并且清空所有定时器，关闭长连接
    this.forbidReconnect = true
    this.clearAllTimer()
    this.ws && this.ws.close()
  }
}
```

### 引入与使用

| 属性             | 必填  | 类型   | 默认值      | 描述                                  |
| :--------------- | :---- | :----- | :---------- | :------------------------------------ |
| url              | true  | string | none        | websocket 服务端接口地址              |
| pingTimeout      | false | number | 8000        | 心跳包发送间隔                        |
| pongTimeout      | false | number | 15000       | 15 秒内没收到后端消息便会认为连接断开 |
| reconnectTimeout | false | number | 4000        | 尝试重连的间隔时间                    |
| reconnectLimit   | false | number | 15          | 重连尝试次数                          |
| pingMsg          | false | string | "heartbeat" | 心跳包消息                            |

```javascript
const opts = {
  url: 'ws://xxx',
  pingTimeout: 8000, // 发送心跳包间隔，默认 8000 毫秒
  pongTimeout: 15000, // 最长未接收消息的间隔，默认 15000 毫秒
  reconnectTimeout: 4000, // 每次重连间隔
  reconnectLimit: 15, // 最大重连次数
  pingMsg: 'heartbeat' // 心跳包的消息内容
}

const ws = new WebsocketHB(opts)

ws.onopen = () => {
  console.log('connect success')
}
ws.onmessage = e => {
  console.log(`onmessage: ${e.data}`)
}
ws.onerror = () => {
  console.log('connect onerror')
}
ws.onclose = () => {
  console.log('connect onclose')
}

// 发送消息
ws.send('Hello World')

// 销毁实例
ws.destroyed()
```
