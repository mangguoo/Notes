# multiparty

> Multiparty用于接收客户端发过来的form-data数据请求

``` js
const form = new multiparty.Form(options)
```

## options参数

- encoding：formdata的数据设置编码，默认是utf-8
- maxFieldsSize:限制字段，按字节分配的内存量，默认是2M，超出则会产生错误。
- maxFields：限制被解析字段的数量，默认为1000
- maxFilesSize：此属性只有在autoFiles为true的时候生效，设置上传文件接收字节的最大数量。也就是限制最大能上传多大的文件
- autoFields：启用field事件，并禁用part事件。如果监听field事件，该属性自动为true
- autoFiles：启用file事件，并禁用part事件，如果监听file事件，则默认为true
- uploadDir：放置文件的目录，只有autoFiels为true是有用

## parse方法

实例化完构造函数后，利用**parse()**方法来解析FormData数据。该方法接收两个参数，无返回值

1. request对象，把创建服务时，回掉函数中的第一个参数传进去就可以

2. callback，一个回调函数，通过该回调函数，可以获取到解析后的数据

如果是上传文件，使用这个回调函数的话。就不需要再执行写入文件的工作了，因为插件已经完成了。只需要设置好uploadDir参数，然后做些后续操作就可以了。因为回调函数会默认开启autoFields和autoFlies。个人感觉应该是内部监听field和file事件

>  该回调函数有三个参数：
>
>  1. err是发生错误时，返回的异常信息
>  2. fields是一个对象，存储着FormData里的字段信息
>  3. files存储的是文件信息。如果你把整个file对象直接放进formData内则有值，否则为空对象。如果要自己写文件的话，这个回调函数完全可以忽略掉

``` js
http
  .createServer(async function (req, res) {
    res.writeHead(200, {
      'Content-Type': 'text/plain;charset=utf-8',
      'Access-Control-Allow-Origin': '*'
    })
    if (req.url === '/upload') {
      const form = new multiparty.Form()
      form.parse(req, async (err, fields, files) => {
        if (err) return res.end('上传失败')
        const file = files.file[0]
        const name = file.originalFilename
        const buff = await fsp.readFile(file.path)
        await fsp.writeFile(`./img/${name}`, buff)
      })
      return res.end('上传成功')
    }
    res.end('404')
  })
  .listen(4003)
```

> files和fields对象示例

``` js
// 可以看到在没有指定uploadDir的情况下，它把文件默认上传到了缓存目录下
{
  file: [
    {
      fieldName: 'file',
      originalFilename: '07-38-31-38dbb6fd5266d0161034125e9c2bd40735fa3592.jpg',
      path: 'D:\\ilmango\\AppData\\Local\\Temp\\OKgsXWTPGPcVzLoGd-C33fb9.jpg',
      headers: [Object],
      size: 26342
    }
  ]
}
{ name: [ '张三' ], age: [ '18' ] }
```

如果要自己写文件就要靠事件，因为nodeJS是通过事件来实现异步编程，来达到其它后台语言多线程的效果。而mutiparty提供了一些事件，开僻了自己写文件的通道。

## part事件

该事件是实现自己写文件的关键。它会在请求中遇到文件数据时触发，它的回调函数是一个实现可读流的实例对象

注意使用part事件时，不要再去监听fields和files事件。如果监听了的话，那在part事件中，你将得不到你想要的数据

> server

``` js
http
  .createServer(async function (req, res) {
    res.writeHead(200, {
      'Content-Type': 'text/plain;charset=utf-8',
      'Access-Control-Allow-Origin': '*'
    })
    if (req.url === '/upload') {
      const form = new multiparty.Form()
      form.parse(req)
      form.on('part', function (part) {
        if (!part.filename) return
        const writeStream = fs.createWriteStream(path.resolve(part.filename))
        part.pipe(writeStream)
      })
      return res.end('上传成功')
    }
    res.end('404')
  })
  .listen(4003)
```

> part对象示例

``` js
PassThrough {
  _readableState: ReadableState { // 可读流
	......
  },
  ........
  allowHalfOpen: true, // 能否断点续传
  headers: { // 请求的头部信息
    'content-disposition': 'form-data; name="file"; filename="IMG_20190417_210716.jpg"',
    'content-type': 'image/jpeg'
  },
  name: 'file', // 字段名称
  filename: 'IMG_20190417_210716.jpg', // 文件名称
  byteOffset: 151, // 表示这部分数据，在主体数据中的字节偏移量
  byteCount: 88156 // 数据总的字节长度
}
```

> client

``` js
var form = document.querySelector('form')
form.addEventListener('submit', sendFiles)

function sendFiles(e) {
  e.preventDefault()
  var data = new FormData(form)
  ajax(data)
}

function ajax(data) {
  var xhr = new XMLHttpRequest()
  xhr.addEventListener('load', loadHandler)
  xhr.open('POST', 'http://localhost:4003/upload')
  // xhr.setRequestHeader('content-type', 'multipart/form-data')
  xhr.send(data)
}

function loadHandler(e) {
  console.log(e.target.response)
}
```

注意：boundary就是在Content-Type为**multipart/form-data**的情况下，作为一个随机数分隔符，来实现既可以上传多个文件，也可以上传键值对

而这个例子中不能设置content-type请求头，这是因为FormData会我们生成了boundary，并用这个boundary分割请求内容，并且默认context-type为 multipart/form-data

而当自己设置content-type请求头时，Multipart会认为我们自己设置了boundary，就不会自己生成，因而就会报错：`missing content-type boundary`

## aborted事件：

>  会在请求中止时触发。

## close事件：

>  会在请求结束之后触发。

## file事件：

> 如果发送的是文件，则可以监听该事件。监听此事件，插件会把文件写到磁盘上，在利用回调返回相关信息。回调函数参数如下：
>
> 1. name：字段名称
>
> 2. file：存储着文件信息的对象

``` js
http
  .createServer(async function (req, res) {
    res.writeHead(200, {
      'Content-Type': 'text/plain;charset=utf-8',
      'Access-Control-Allow-Origin': '*'
    })
    if (req.url === '/upload') {
      const form = new multiparty.Form()
      form.parse(req)
      form.on('file', async function (name, file) {
        const buff = await fsp.readFile(file.path)
        await fsp.writeFile(`./img/${file.originalFilename}`, buff)
        res.end('上传成功')
      })
      return res.end('上传成功')
    }
    res.end('404')
  })
  .listen(4003)
```

> file对象示例

``` js
file {
  fieldName: 'file',
  originalFilename: '07-38-31-38dbb6fd5266d0161034125e9c2bd40735fa3592.jpg',
  path: 'D:\\ilmango\\AppData\\Local\\Temp\\nvzCWoU7jyGY1j0l4cg5TI3k.jpg',
  headers: {
    'content-disposition': 'form-data; name="file"; filename="07-38-31-38dbb6fd5266d0161034125e9c2bd40735fa3592.jpg"',
    'content-type': 'image/jpeg'
  },
  size: 26342
}
```

## field事件：

>  监听此事件，可以获取到请求中的具体数据。回调函数两个参数。name：字段名。value：字段值。

