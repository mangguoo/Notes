#### cheerio

> 可以把请求到的html文件格式化为一个jquery对象

``` js
const http = require('http')
const cheerio = require('cheerio')
const { URL } = require('url')
const fs = require('fs/promises')
const path = require('path')
init()
async function init() {
  const data = await getCheerio('http://nodejs.cn/api/')
  const $ = cheerio.load(data)
  $('a').each(async (index, item) => { // 遍历html文件中的所有a标签
    const name = $(item).attr('href') // 获取他们的href
    if (!/.html$/.test(name)) return // 筛选其中的指向html文件的链接
    const url = 'http://nodejs.cn/api/' + name // 拼接链接，继续请求
    const d = await getCheerio(url)
    fs.writeFile(path.join(__dirname, name), d) // 请求成功后写入文件中
  })
}

function getCheerio(href) {
  return new Promise(function (resolve, reject) {
    const result = http.request(new URL(href), async function (req) {
      const data = await getData(req)
      resolve(data)
    })
    result.end()
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

