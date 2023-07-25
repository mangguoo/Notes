# readlie

> readline是Node.js里实现标准输入输出的封装好的模块，通过这个模块我们可以以逐行的方式读取数据流。readline 模块提供了用于从可读流（例如 process.stdin）每次一行地读取数据的接口。

## close事件：

>  发生以下情况之一时会触发close事件：

（1）rl.close() 方法被调用，readline.Interface 实例放弃了对 input 和 output 流的控制；
（2）input 流接收到它的 ‘end’ 事件；
（3）input 流接收 Ctrl+D 以发出传输结束（EOT）的信号；
（4）input 流接收 Ctrl+C 以发出 SIGINT 信号，并且在 readline.Interface 实例上没有注册 ‘SIGINT’ 事件监听器。
调用监听器函数时不传入任何参数。一旦触发 ‘close’ 事件，则 readline.Interface 实例就完成了。

## readline.createInterface(options)：

input <stream.Readable> 要监听的可读流。
output <stream.Writable> 要将逐行读取的数据写入的可写流。

## 使用示例：

> 例子：读取文件中的数据，并格式化为sql语句，如下：
>
> `2	17	0	1	37	a99104	.1`    <===>   `INSERT INTO tb_param(protocol, slave, number, ptype, pid, name, format) VALUES(2, 24, 0, 1, 1, 'a04005', '.3');`

```js
const fs = require('fs')
const readline = require('readline')

const strInputFileName = './input.txt'
const strOutputFileName = 'output.txt'

const fRead = fs.createReadStream(strInputFileName)
const fWrite = fs.createWriteStream(strOutputFileName)

const objReadline = readline.createInterface({
  input: fRead,
  output: fWrite,
  terminal: false
})

objReadline.on('line', (strLine) => {
  const [protocol, slave, number, ptype, pid, name, format] = strLine.split(',')
  let strTemp = `INSERT INTO tb_param(protocol, slave, number, ptype, pid, name, format)\n `
  strTemp += `VALUES(${protocol}, ${slave}, ${number}, ${ptype}, ${pid}, '${name}', '${format}');\n`
  fWrite.write(strTemp)
})

objReadline.on('close', () => {
  console.log('readline close...')
})

fRead.on('end', () => {
  console.log('end')
})
```

> readline除了可以按行读取文件，还可以接收标准输入：

```js
const readline = require('readline')

const r1 = readline.createInterface({
  input: process.stdin,
  output: process.stdout
})
// 使用question方法，可以实现命令行的交互
r1.question('你想吃什么？', function (answer) {
  console.log(`我想吃${answer}`)
  r1.close() // 添加close事件，不然不会结束
})
// close事件监听
r1.on('close', function () {
  process.exit(0) // 结束程序
})
```

