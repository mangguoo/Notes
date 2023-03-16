# js-cookie

## 安装

### ESModule

```bash
$ npm i js-cookie
```

### 直接下载

> 并非所有浏览器都支持 ESModule。由于这个原因，同时提供 ESModule 和 UMD 版本：

```html
<script type="module" src="/path/to/js.cookie.mjs"></script>
<script type="module">
  import Cookies from '/path/to/js.cookie.mjs'

  Cookies.set('foo', 'bar')
</script>
```

```html
<script nomodule defer src="/path/to/js.cookie.js"></script>
```

## TS声明

```bash
$ npm i @types/js-cookie
```

## 基础使用

### 设置cookie

```js
import Cookies from 'js-cookie'

Cookies.set('name', 'value', { expires: 7, path: '' })
```

### 获取cookie

```js
// 不传参数表示获取所有cookie
Cookies.get() // => { name: 'value' }
```

```js
// 获取指定域名下的foo cookie
Cookies.get('foo', { domain: 'sub.example.com' }) 
```

### 删除cookie

```js
Cookies.set('name', 'value', { path: '' })
Cookies.remove('name') // fail!
Cookies.remove('name', { path: '' }) // removed!
```

删除 cookie 时，如果不依赖默认属性，则必须传递与设置 cookie 时完全相同的路径和域属性

```js
Cookies.remove('name', { path: '', domain: '.yourdomain.com' })
```

## Cookie参数：

> Cookie属性默认值可以通过 withAttritribute()进行全局设置，它会返回一个新的Cookie实例
>
> 或者通过Cookies.set(...)的调用单独设置(优先级更高，会覆盖全局默认设置)
>
> ```js
> const api = Cookies.withAttributes({ path: '/', domain: '.example.com' })
> ```

### expires

> 定义 Cookie 何时将被删除。值必须是一个数字，该数字将被解释为从创建时开始cookie被保存的天数。如果省略，Cookie 将成为Session Cookie

```js
Cookies.set('name', 'value', { expires: 365 })
Cookies.get('name') // => 'value'
Cookies.remove('name')
```

### path

> 指定 Cookie 的可见路径，默认为 /

```js
Cookies.set('name', 'value', { path: '' })
Cookies.get('name') // => 'value'
Cookies.remove('name', { path: '' })
```

### domain

> 一个字符串，用于指定 Cookie 应该可见的有效域。Cookie 对于所有子域也是可见的。默认值: Cookie 只对创建 Cookie 的页面的域或子域可见

```js
Cookies.set('name', 'value', { domain: 'subdomain.site.com' })
Cookies.get('name') // => undefined (need to read at 'subdomain.site.com')
```

### secure

> true或false，指定 Cookie 传输是否需要安全协议(https)。默认值: false

```js
Cookies.set('name', 'value', { secure: true })
Cookies.get('name') // => 'value'
Cookies.remove('name')
```

### sameSite

> 字符串，用于控制浏览器是否正在发送 Cookie 以及跨站点请求。默认值: 未设置

```js
Cookies.set('name', 'value', { sameSite: 'strict' })
Cookies.get('name') // => 'value'
Cookies.remove('name')
```

## 设置命名空间

> 如果存在与`namespace Cookies`发生冲突的危险，可以使用noConflict 方法创建新的命名空间并保留原来的名称空间。这在第三方站点上运行脚本时尤其有用，例如作为小部件或 SDK 的一部分

```js
var Cookies2 = Cookies.noConflict()
Cookies2.set('name', 'value')
```

## 编码

> 这个项目是兼容RFC 6265。所有不允许出现在cookie-name或cookie-value中的特殊字符都使用百分号编码编码，使用每个字符的UTF-8十六进制等效值。
>
> Cookie-name或Cookie-value中允许并仍然编码的唯一字符是百分比% 字符，为了将百分比输入解释为文本，它进行了转义。
>
> 请注意，默认的编码/解码策略意味着，只有在由js-cookie进行读/写操作的 cookie之间，才能进行互操作。要覆盖默认的编码/解码策略，需要使用转换器。

## 转换器

> 覆盖原有的编码/解码器

```js
document.cookie = 'escaped=%u5317'
document.cookie = 'default=%E5%8C%97'
var cookies = Cookies.withConverter({
  read: function (value, name) {
    if (name === 'escaped') {
      return unescape(value)
    }
    // 所有其他 Cookie 使用默认方案
    return Cookies.converter.read(value, name)
  }
})
cookies.get('escaped') // 北
cookies.get('default') // 北
cookies.get() // { escaped: '北', default: '北' }
```

```js
Cookies.withConverter({
  write: function (value, name) {
    return value.toUpperCase()
  }
})
```

