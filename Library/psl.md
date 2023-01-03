# psl

> Public Suffix List是一个跨厂商的倡议，以提供一个准确的域名后缀列表。
>
> Public Suffix List是Mozilla项目的一个倡议，但是作为社区资源进行维护。它可以在任何软件中使用，但最初是为了满足浏览器制造商的需要而创建的。
>
> “Public Suffix”是指互联网用户可以直接注册的后缀名。比如说“.com”和“pvt.k12.wy.us”。Public Suffix List是所有已知公共后缀的列表。

## 安装

### Node.js

```bash
npm install --save psl
```

### Browser

```js
<script src="psl.min.js"></script>
```

## API

### psl.parse(domain)

> 基于公共后缀列表解析域名。返回具有以下属性的对象:
>
> - tld: 顶级域(这是公共后缀)
>
> - sld: 二级域(域名的第一个私有部分)
>
> - domain: 域名是 sld + tld
>
> - subdomain: 可选的域的左边部分

```js
var psl = require('psl');

// Parse domain without subdomain
var parsed = psl.parse('google.com');
console.log(parsed.tld); // 'com'
console.log(parsed.sld); // 'google'
console.log(parsed.domain); // 'google.com'
console.log(parsed.subdomain); // null

// Parse domain with subdomain
var parsed = psl.parse('www.google.com');
console.log(parsed.tld); // 'com'
console.log(parsed.sld); // 'google'
console.log(parsed.domain); // 'google.com'
console.log(parsed.subdomain); // 'www'

// Parse domain with nested subdomains
var parsed = psl.parse('a.b.c.d.foo.com');
console.log(parsed.tld); // 'com'
console.log(parsed.sld); // 'foo'
console.log(parsed.domain); // 'foo.com'
console.log(parsed.subdomain); // 'a.b.c.d'
```

### psl.get(domain)

> 获取域名sld + tld。如果无效，返回 null

```js
var psl = require('psl');

// null input.
psl.get(null); // null

// Mixed case.
psl.get('COM'); // null
psl.get('example.COM'); // 'example.com'
psl.get('WwW.example.COM'); // 'example.com'

// Unlisted TLD.
psl.get('example'); // null
psl.get('example.example'); // 'example.example'
psl.get('b.example.example'); // 'example.example'
psl.get('a.b.example.example'); // 'example.example'

// TLD with only 1 rule.
psl.get('biz'); // null
psl.get('domain.biz'); // 'domain.biz'
psl.get('b.domain.biz'); // 'domain.biz'
psl.get('a.b.domain.biz'); // 'domain.biz'

// TLD with some 2-level rules.
psl.get('uk.com'); // null);
psl.get('example.uk.com'); // 'example.uk.com');
psl.get('b.example.uk.com'); // 'example.uk.com');

// More complex TLD.
psl.get('c.kobe.jp'); // null
psl.get('b.c.kobe.jp'); // 'b.c.kobe.jp'
psl.get('a.b.c.kobe.jp'); // 'b.c.kobe.jp'
psl.get('city.kobe.jp'); // 'city.kobe.jp'
psl.get('www.city.kobe.jp'); // 'city.kobe.jp'

// IDN labels.
psl.get('食狮.com.cn'); // '食狮.com.cn'
psl.get('食狮.公司.cn'); // '食狮.公司.cn'
psl.get('www.食狮.公司.cn'); // '食狮.公司.cn'

// Same as above, but punycoded.
psl.get('xn--85x722f.com.cn'); // 'xn--85x722f.com.cn'
psl.get('xn--85x722f.xn--55qx5d.cn'); // 'xn--85x722f.xn--55qx5d.cn'
psl.get('www.xn--85x722f.xn--55qx5d.cn'); // 'xn--85x722f.xn--55qx5d.cn'
```

### psl.isValid(domain)

> 检查域是否具有有效的公共后缀。返回一个布尔值，指示域是否具有有效的公共后缀

```js
var psl = require('psl');

psl.isValid('google.com'); // true
psl.isValid('www.google.com'); // true
psl.isValid('x.yz'); // false
```

