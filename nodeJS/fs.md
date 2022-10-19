#### file

##### 引入

``` js
const fs=require("fs");
const fsp=require("fs/promise");
```

##### fsp.mkdir

``` js
fsp.mkdir(path.join(__dirname, 'test/anlw/ppp'), { recursive: true }).catch((err) => {
  console.log(err)
})
// recursive选项表示递归创建
```

##### fsp.rm

删除文件和删除文件夹都可以使用它

``` js
fsp.rm(path.join(__dirname, 'aa'), { recursive: true }).catch((err) => {
  console.log(err)
})
// recursive选项表示是否递归删除（类似于rm -rf）
```

##### fs.existsSync

这是一个同步方法，只在fs中存在

``` js
console.log(fs.existsSync(path.resolve('input.txt')))
```

##### fsp.readFile   fs.readFileSync

``` js
fsp.readFile(path.resolve('input.txt'), { encoding: 'utf-8' }).then((data) => {
  console.log(data)
})
```

> readFileSync 异步没有回调函数，但是可以使用try catch来捕获错误

``` js
const buffer = fs.readFileSync(path.resolve('input.txt'), 'utf8')
console.log(buffer.toString())
```

##### fsp.readdir   fs.readdirSync

``` js
fsp.readdir(path.resolve('.'), { encoding: 'utf-8' }).then((data) => {
  console.log(data)
})
```

> readdirSync

``` js
const files = fs.readdirSync(path.resolve('.'))
console.log(files)
```

##### fsp.stat   fs.statSync

``` js
fsp.stat(path.resolve('input.txt')).then((data) => {
  console.log(data)
})
```

> stat.isDirectory()   stat.isFile()
>
> 这两个方法都返回一个布尔值

``` js
async function readPath(dir) {
  const list = await fsp.readdir(dir)
  list.forEach(async (item) => {
    const t = await fsp.stat(path.join(dir, item))
    if (t.isDirectory()) {
      readPath(path.join(dir, item)) // 递归
    } else {
      const res = await fsp.readFile(path.join(dir, item), 'utf-8')
      console.log(res)
    }
  })
}
readPath('.')
```

> statSync()

``` js
const stats = fs.statSync(path.resolve('input.txt'))
console.log(stats.isFile())
```

##### tsnode 静态文件服务器

``` ts
import path from 'path'
import fsp from 'fs/promises'
import http from 'http'
import mime from 'mime'

http
  .createServer(async function (req, res) {
    let data, type
    const fileUrl = path.join(path.resolve(), req.url?.split('?')[0]!)
    type = mime.getType(path.join(path.resolve(), fileUrl))

    const fileStat = await fsp.stat(fileUrl)
    if (fileStat.isDirectory()) {
      data = await fsp.readFile(path.join(fileUrl, 'index.html'))
      type = 'text/html'
    } else {
      if (/text/.test(type!)) data = await fsp.readFile(fileUrl, 'utf8')
      else data = await fsp.readFile(fileUrl)
    }

    res.writeHead(200, {
      'Content-Type': type + ';charset=utf-8'
    })
    res.end(data)
  })
  .listen(4000)
```

##### fsp.writeFile   fs.writeFileSync

> fs.writeFile(file, data[, options], callback)
>
> - file：文件名（可以是相对路径，也可以是绝对路径）
>
> - data：想写入文件的数据
>
> - options里面有 encoding,mode,flag。注意他们都有默认值，所以一般不用传
>
> - callback是回调函数
>
> flag一共有三个（w：写，r：读，a：追加）。默认的是w，即写，如果有既存文件也会覆盖。如果使用a则会追加，而不会覆盖。

```js
const path = require('path')
const fsp = require('fs/promises')

fsp.writeFile(path.join(__dirname, './a.txt'), 'aaabbbccc', 'utf8').catch((err) => {
  console.log(err)
})
```

> writeFileSync

``` js
const path = require('path')
const fs = require('fs')

try {
  fs.writeFileSync(path.join(path.resolve(), 'input.txt'), 'aaa', 'utf-8')
} catch (error) {
  console.log(error)
}
```

##### writeFile实现上传文件

> 方法一：server

``` js
const http = require('http')
const fsp = require('fs/promises')
const path = require('path')

http
  .createServer(async function (req, res) {
    const buff = await getData(req)
    const name = req.headers['x-file-name'] ?? ''
    res.writeHead(200, {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'text/html;charset=utf-8',
      'Access-Control-Allow-Headers': '*'
    })
    if (buff.length === 0) return res.end()
    const file = await fsp.writeFile(path.join(path.resolve(), 'img', name), buff)
    if (file) res.end('保存成功')
    else res.end('上传失败')
  })
  .listen(4001)

function getData(req) {
  return new Promise((resolve, reject) => {
    let buff = Buffer.alloc(0) // 创建一个长度为0的二进制流数组
    req.on('data', (_chunk) => {
      // 合并二进制流数组
      buff = Buffer.concat([buff, _chunk], buff.length + _chunk.length)
    })
    req.on('end', () => resolve(buff))
  })
}
```

> 方法一：client

``` js
<form>
  <input type="file" name="file" />
  <button type="submit">提交</button>
</form>
<script>
  var form = document.querySelector('form')
  form.addEventListener('submit', submitHandler)

  async function submitHandler(e) {
    e.preventDefault()
    var file = document.querySelector('[type=file]').files[0]
    var fr = await loadFile(file)
    var result = await ajax(file.name, fr.result)
    console.log(result)
  }

  function ajax(name, buff) {
    return new Promise((resolve, reject) => {
      var xhr = new XMLHttpRequest()
      xhr.open('POST', 'http://localhost:4001')
      xhr.setRequestHeader('X-File-Name', name)
      xhr.send(buff)
      xhr.onload = function () {
        resolve(xhr.response)
      }
    })
  }

  function loadFile(file) {
    return new Promise((resolve, reject) => {
      var fr = new FileReader()
      fr.readAsArrayBuffer(file)
      fr.onload = function () {
        resolve(fr)
      }
    })
  }
</script>
```

> 方法二：server

``` js
const http = require('http')
const fs = require('fs')

http
  .createServer((req, res) => {
    res.writeHead(200, {
      'content-type': 'text/html;charset=utf-8',
      'Access-Control-Allow-Origin': '*'
    })
    req.pipe(fs.createWriteStream('./' + req.url, 'base64'))
    res.end('完成了')
  })
  .listen(4001)
```

> 方法二：client

``` js
const file = document.querySelector('[type=file]')
file.addEventListener('change', sendFiles)

function sendFiles() {
  const reader = new FileReader()
  reader.readAsArrayBuffer(file.files[0])
  reader.addEventListener('load', (e) => ajax(reader.result, file.files[0].name))
}

function ajax(data, name) {
  const xhr = new XMLHttpRequest()
  xhr.addEventListener('load', loadHandler)
  xhr.open('POST', 'http://localhost:4001/' + name)
  xhr.send(data)
}

function loadHandler(e) {}

```

##### fsp.appendFile   fs.appendFileSync

``` js
const path = require('path')
const fsp = require('fs/promises')

fsp.appendFile(path.resolve('input.txt'), 'aaaaa\n', 'utf8').catch((err) => {
  console.log(err)
})
```

> appendFileSync

``` js
fs.appendFileSync(path.resolve('input.txt'), 'aaaaa\n', 'utf8')
```

##### fsp.rename

> 重命名文件，可以实现文件移动

``` js
const path = require('path')
const fsp = require('fs/promises')

fsp.rename(path.join(path.resolve(), 'img', '1.jpg'), path.join(path.resolve(), 'img', '2.jpg')).catch((err) => {
  console.log(err)
})
```

##### fsp.unlink

> 如果路径是指符号链接，则删除链接而不会影响链接所指的文件或目录。如果路径是指非符号链接的文件路径，则将删除文件。

``` js
const path = require('path')
const fsp = require('fs/promises')

fsp.unlink(path.join(path.resolve(), 'input.txt')).catch((err) => {
  console.log(err)
})
```

##### fsp.link

``` js
fsp.link(path.resolve('index.html'), path.resolve('index.html.lnk')).catch((err) => {
  console.log(err)
})
```

##### fs.watch  fsp.watch

> 侦听文件的变化，recursive选项表示会侦听后代目录下的文件
>
> eventType表示文件所做的操作，比如change、rename等等
>
> filename表示哪个文件发生了变化

``` js
const path = require('path')
const fs = require('fs')

fs.watch(path.resolve('txt'), { recursive: true }, (eventType, filename) => {
  console.log(eventType, filename)
})
```

> fsp.watch：返回一个异步迭代器，该迭代器每次迭代都会侦听文件的更改

``` js
const path = require('path')
const fsp = require('fs/promises')

;(async function () {
  const watcher = fsp.watch(path.resolve('txt'), { recursive: true })
  for await (const info of watcher) {
    console.log(info)
  }
})()
```

##### fs.createReadStream

``` js
const path = require('path')
const fs = require('fs')

const readStream = fs.createReadStream(path.resolve('index.html'))
let data = ''
readStream.on('data', (chunk) => {
  data += chunk
})
readStream.on('end', () => {
  console.log(data)
})
```

> 使用管道连接可读流和可写流

``` js
readStream.pipe(fs.createWriteStream(path.resolve('index_back.html')))
```

##### fs.createWriteStream

``` js
const path = require('path')
const fs = require('fs')

const streamWriter = fs.createWriteStream(path.resolve('input.txt'))
setInterval(function () {
  streamWriter.write(`${new Date()}\n`, (error) => {
    console.log(error)
  })
}, 1000)
```

#### fs.access

##### 参数：

> `path` <strin> | <Buffer> | <URL>
>
> `mode`<inteer> **默认值:** `fs.constants.F_OK`。
>
> `callback` <Function>
>
> - `err` <Error>

##### 作用：

> **测试用户对 `path` 指定的文件或目录的权限。**
> `mode` 参数是一个可选的整数，指定要执行的可访问性检查。文件可访问性常量：

| 常量 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| F_OK | 表明文件对调用进程可见。这对于判断文件是否存在很有用，但对rwx权限没有任何说明。如果未指定模式，则默认值为该值。 |
| R_OK | 表明调用进程可以读取文件。                                   |
| W-OK | 表明调用进程可以写入文件。                                   |
| X_OK | 表明调用进程可以执行文件。在Windows上无效（表现得像fs.constants.F_OK） |

##### 检查文件是否存在：

```js
const fs = require('fs')
const path = require('path')

//检查文件是否存在于当前目录中
fs.access('package.json', fs.constants.F_OK, err => {
    if(err) {
        console.log('package.json不存在于当前目录中')
        return
    }
    console.log('package.json存在于当前目录中')
})
```

##### 检查文件是否可读:

```js
const fs = require('fs')
const path = require('path')

fs.access('index.js', fs.constants.R_OK, err => {
    console.log(`index.js${err? '不可读':'可读'}`)
})
```

##### 检查文件是否可写：

```js
const fs = require('fs')
const path = require('path')

fs.access('index.js', fs.constants.W_OK, err => {
    console.log(`index.js${err? '不可写':'可写'}`)
})
```

