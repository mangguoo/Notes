### 概述

> 正则表达式（regular expression）是一种表达文本模式（即字符串结构）的方法，有点像字符串的模板，常常用来按照“给定模式”匹配文本。比如，正则表达式给出一个 Email 地址的模式，然后用它来确定一个字符串是否为 Email 地址。

新建正则表达式有两种方式：

``` js
var regex = /xyz/;

var regex = new RegExp('xyz', "i");
```

它们的主要区别是，第一种方法在引擎编译代码时，就会新建正则表达式，第二种方法在运行时新建正则表达式，所以前者的效率较高。而且，前者比较便利和直观，所以实际应用中，基本上都采用字面量定义正则表达式。

### RegExp实例属性

正则实例属性分两类：

- 修饰符相关(全部是只读属性)：
  - `RegExp.prototype.ignoreCase`：返回一个布尔值，表示是否设置了`i`修饰符。
  - `RegExp.prototype.global`：返回一个布尔值，表示是否设置了`g`修饰符。
  - `RegExp.prototype.multiline`：返回一个布尔值，表示是否设置了`m`修饰符。
  - `RegExp.prototype.sticky`： 返回一个布尔值，表示是否设置了`y`修饰符。
  - `RegExp.prototype.dotAll`：返回一个布尔值，表示是否设置了`s`修饰符。
  - `RegExp.prototype.hasIndices`：返回一个布尔值，表示是否设置了`d`修饰符。
  - `RegExp.prototype.flags`：返回一个字符串，包含了已经设置的所有修饰符，按字母排序。
- 修饰符无关的属性(全部是只读属性)：
  - `RegExp.prototype.lastIndex`：返回一个整数，表示下一次开始搜索的位置。**该属性可读写**，但是只在进行连续搜索时有意义。
  - `RegExp.prototype.source`：返回正则表达式的字符串形式（不包括反斜杠），该属性只读。

### RegExp实例方法

#### 1. RegExp.prototype.test()

正则实例对象的`test`方法返回一个布尔值，表示当前模式是否能匹配参数字符串。如果正则表达式带有`g`修饰符，则每一次`test`方法都从上一次结束的位置开始向后匹配，并且可以通过正则对象的`lastIndex`属性指定开始搜索的位置。

``` js
var r = /x/g;
var s = '_x_x';

r.lastIndex // 0
r.test(s) // true

r.lastIndex // 2
r.test(s) // true

r.lastIndex // 4
r.test(s) // false
```

``` js
var r = /x/g;
var s = '_x_x';

r.lastIndex = 4;
r.test(s) // false

r.lastIndex // 0
r.test(s) // true
```

> 坑1：带有`g`修饰符时，正则表达式内部会记住上一次的`lastIndex`属性，这时不应该更换所要匹配的字符串，否则会有一些难以察觉的错误。

``` js
var r = /bb/g;
r.test('bb') // true
r.test('-bb-') // false
```

> 坑2：下面代码会导致无限循环，因为`while`循环的每次匹配条件都是一个新的正则表达式，导致`lastIndex`属性总是等于0。

``` js
var count = 0;
while (/a/g.test('babaa')) count++;
```

#### 2. RegExp.prototype.exec()

正则实例对象的`exec()`方法，用来返回匹配结果。如果发现匹配，就返回一个数组，成员是匹配成功的子字符串，否则返回`null`。如果正则表示式包含圆括号（即含有“组匹配”），则返回的数组会包括多个成员。第一个成员是整个匹配成功的结果，后面的成员就是圆括号对应的匹配成功的组。

``` js
var s = '_x_x';
var r1 = /y/;
var r2 = /_(x)/;

r1.exec(s) // null
r2.exec(s) // ["_x", "x"]
```

`exec()`方法的返回数组还包含以下四个属性：

- `input`：整个原字符串。
- `index`：模式匹配成功的开始位置（从0开始计数）。
- `groups` ：具名组对象。
- `indices`：表示匹配结果和所有组匹配结果的开始和结束位置（需要开启`d`修饰符）。

如果正则表达式加上`g`修饰符，则可以使用多次`exec()`方法，下一次搜索的位置从上一次匹配成功结束的位置开始。

``` js
var reg = /a/g;
var str = 'abc_abc_abc'

var r1 = reg.exec(str);
r1 // ["a"]
r1.index // 0
reg.lastIndex // 1

var r2 = reg.exec(str);
r2 // ["a"]
r2.index // 4
reg.lastIndex // 5

var r3 = reg.exec(str);
r3 // ["a"]
r3.index // 8
reg.lastIndex // 9

var r4 = reg.exec(str);
r4 // null
reg.lastIndex // 0
```

上面代码连续用了四次`exec()`方法，前三次都是从上一次匹配结束的位置向后匹配。当第三次匹配结束以后，整个字符串已经到达尾部，匹配结果返回`null`，正则实例对象的`lastIndex`属性也重置为`0`，意味着第四次匹配将从头开始。

``` js
const reg = /a/g
const str = 'abc_abc_abc'
let match

while ((match = reg.exec(str))) {
  console.log('#' + match.index + ':' + match[0])
}
```

正则实例对象的`lastIndex`属性不仅可读，还可写。设置了`g`修饰符的时候，只要手动设置了`lastIndex`的值，就会从指定位置开始匹配。

### 字符串的实例方法

字符串的实例方法之中，有4种与正则表达式有关。

- `String.prototype.match()`：返回一个数组，成员是所有匹配的子字符串。
- `String.prototype.matchAll()`：返回一个遍历器，可遍历全部查询到的结果。
- `String.prototype.search()`：按照给定的正则表达式进行搜索，返回一个整数，表示匹配开始的位置。
- `String.prototype.replace()`：按照给定的正则表达式进行替换，返回替换后的字符串。
- `String.prototype.split()`：按照给定规则进行字符串分割，返回一个数组，包含分割后的各个成员。

ES6 将这 五个方法，在语言内部全部调用`RegExp`的实例方法，从而做到所有与正则相关的方法，全都定义在`RegExp`对象上。

- `String.prototype.match` 调用 `RegExp.prototype[Symbol.match]`
- `String.prototype.matchAll` 调用 `RegExp.prototype[Symbol.matchAll]`
- `String.prototype.replace` 调用 `RegExp.prototype[Symbol.replace]`
- `String.prototype.search` 调用 `RegExp.prototype[Symbol.search]`
- `String.prototype.split` 调用 `RegExp.prototype[Symbol.split]`

#### 1. String.prototype.match()

字符串实例对象的`match`方法对字符串进行正则匹配，返回匹配结果。字符串的`match`方法与正则对象的`exec`方法非常类似：匹配成功返回一个数组，匹配失败返回`null`。在没有`g`修饰符时，可以把`match`看成只能调用一次的`exec`方法。

但是如果正则表达式带有`g`修饰符，则该方法与正则对象的`exec`方法行为不同，会一次性返回所有匹配成功的结果，但是不会理会组匹配，所以想要获得组匹配的话，就不要加`g`修饰符。

``` js
const s = 'abba'
const r = /a/g

console.log(s.match(r)) 
// ["a", "a"]
console.log(r.exec(s)) 
// [ 'a', index: 0, input: 'abba', groups: undefined ]
```

``` js
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/

console.log(RE_DATE.exec('1999-12-31'))
console.log('1999-12-31'.match(RE_DATE))

[
  '1999-12-31',
  '1999',
  '12',
  '31',
  index: 0,
  input: '1999-12-31',
  groups: { year: '1999', month: '12', day: '31' }
]
[
  '1999-12-31',
  '1999',
  '12',
  '31',
  index: 0,
  input: '1999-12-31',
  groups: { year: '1999', month: '12', day: '31' }
]
```

使用正则表达式实现addClass和removeClass


``` js
function addClass(elem, className) {
  elem.className = className
    .match(/\S+/g)
    .reduce(function (v, t) {
      if (!v.includes(t)) v.push(t)
      return v
    }, elem.className.match(/\S+/g))
    .join(' ')
}

function removeClass(elem, className) {
  elem.className = className
    .match(/\S+/g)
    .reduce(function (v, t, i) {
      if (v.includes(t)) v.splice(i, 1)
      return v
    }, elem.className.match(/\S+/g))
    .join(' ')
}

const div = document.querySelector('div')
addClass(div, '  div2   div4   div5 ')
removeClass(div, '  div2   div4   div5 ')
```

> 设置正则表达式的`lastIndex`属性，对`match`方法无效，匹配总是从字符串的第一个字符开始。

``` js
var r = /a|b/g;
r.lastIndex = 7;
'xaxb'.match(r) // ['a', 'b']
r.lastIndex // 0
```

#### 2. String.prototype.matchAll()

如果一个正则表达式在字符串里面有多个匹配，现在一般使用`g`修饰符或`y`修饰符，在循环里面逐一取出。有了matchAll方法后就可以一次性取出所有匹配。

不过，它返回的是一个遍历器（Iterator），而不是数组。返回遍历器的好处在于，如果匹配结果是一个很大的数组，那么遍历器比较节省资源。

``` js
const string = 'test1test2test3';
const regex = /t(e)(st(\d?))/g;

for (const match of string.matchAll(regex)) {
  console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```

#### 3. String.prototype.search()

字符串对象的`search`方法，返回第一个满足条件的匹配结果在整个字符串中的位置。如果没有任何匹配，则返回`-1`。

``` js
'_x_x'.search(/x/)
// 1
```

#### 4. String.prototype.replace()

字符串对象的`replace`方法可以替换匹配的值。它接受两个参数，第一个是正则表达式，表示搜索模式，第二个是替换的内容。正则表达式如果不加`g`修饰符，就替换第一个匹配成功的值，否则替换所有匹配成功的值。

``` js
var str = '  #id div.class  ';

str.replace(/^\s+|\s+$/g, '')
// "#id div.class"
```

`replace`方法的第二个参数可以使用美元符号`$`，用来指代所替换的内容。

- `$&`：匹配的子字符串。
- $\`：匹配结果前面的文本。
- `$'`：匹配结果后面的文本。
- `$n`：匹配成功的第`n`组内容，`n`是从1开始的自然数。
- `$<name>`：匹配具名组，name是具名组的组名。
- `$$`：指代美元符号`$`。

``` js
'hello world'.replace(/(\w+)\s(\w+)/, '$2 $1')
// "world hello"

'abc'.replace('b', '[$`-$&-$\']')
// "a[a-b-c]c"
```

`replace`方法的第二个参数还可以是一个函数，将每一个匹配内容替换为函数返回值。作为`replace`方法第二个参数的替换函数，可以接受多个参数。其中，第一个参数是捕捉到的内容，第二个参数是捕捉到的组匹配（有多少个组匹配，就有多少个对应的参数）。此外，最后还可以添加三个参数，倒数第三个参数是捕捉到的内容在整个字符串中的位置（比如从第五个位置开始），倒数第二个参数是原字符串，最后一个是具名组构成的对象。

``` js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
'2015-01-02'.replace(re, (
   matched, // 整个匹配结果 2015-01-02
   capture1, // 第一个组匹配 2015
   capture2, // 第二个组匹配 01
   capture3, // 第三个组匹配 02
   position, // 匹配开始的位置 0
   S, // 原字符串 2015-01-02
   groups // 具名组构成的一个对象 {year, month, day}
 ) => {
 let {day, month, year} = groups;
 return `${day}/${month}/${year}`;
})
```

``` JS
var prices = {
  'p1': '$1.99',
  'p2': '$9.99',
  'p3': '$5.00'
};

var template = '<span id="p1"></span>'
  + '<span id="p2"></span>'
  + '<span id="p3"></span>';

template.replace(
  /(<span id=")(.*?)(">)(<\/span>)/g,
  function(match, $1, $2, $3, $4, index, str){
    return $1 + $2 + $3 + prices[$2] + $4;
  }
);
// "<span id="p1">$1.99</span><span id="p2">$9.99</span><span id="p3">$5.00</span>"
```

> replace的结果不受正则对象lastIndex属性的影响！

**`replace`使用实例：**

``` js
// 编译格式字符串
const str = '3[ab]12[cd]'
const res = str.replace(/(\d+)\[([a-zA-Z]+)\]/g, function (item, a, b) {
  return b.repeat(a)
})
// abababcdcdcdcdcdcdcdcdcdcdcdcd

// 递归编译嵌套格式字符串
let str = '3[2[a]3[bc]]2[ab]'
function stringify(str) {
  if (!/\d+\[\w+\]/.test(str)) return str
  return stringify(
    str.replace(/(\d+)\[(\w+)\]/g, function (item, a, b) {
      return b.repeat(a)
    })
  )
}
str = stringify(str)
// aabcbcbcaabcbcbcaabcbcbcabab

// 解析query字符串
const str = 'a=3&b=a&c=5&d=6'
const o = {}
str.replace(/(\w+)=([^&]+)/g, function (item, a, b) {
  o[a] = isNaN(b) ? b : Number(b)
})
// { a: 3, b: 'a', c: 5, d: 6 }

// 对手机号进行隐私处理
const str = '18617890567'
const res = str.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2')
// 186****0567

// 对身份证号进行隐私处理
const str = '11012219980524401X'
const res = str.replace(/(\d{4})\d{11}(\d{3}|\d{2}X)/, '$1***********$2')
// 1101***********01X

// 使用正则计算字符串加法
let str = '1+2+3='
// 这边的后行断言匹配到的是等号后面的空
str = str.replace(/(?<==)/, function (item, index, str) {
  // 倒数第一个参数是原字符串
  // 倒数第二个参数是开始匹配的位置
  return str
    .match(/(\d+)\+(\d+)\+(\d+)/)
    .slice(1)
    .reduce(function (v, t) {
      return Number(v) + Number(t)
    })
})
console.log(str)
```

#### 5. String.prototype.split()

字符串对象的`split`方法按照正则规则分割字符串，返回一个由分割后的各个部分组成的数组。该方法接受两个参数，第一个参数是正则表达式，表示分隔规则，第二个参数是返回数组的最大成员数。

``` js
'aaa*a*'.split(/a*/)
// [ '', '*', '*' ]

'aaa**a*'.split(/a*/)
// ["", "*", "*", "*"]
```

上面代码的分割规则是0次或多次的`a`，由于正则默认是贪婪匹配，所以例一的第一个分隔符是`aaa`，第二个分割符是`a`，将字符串分成三个部分，包含开始处的空字符串。例二的第一个分隔符是`aaa`，第二个分隔符是0个`a`（即空字符），第三个分隔符是`a`，所以将字符串分成四个部分。

如果正则表达式带有括号，则括号匹配的部分也会作为数组成员返回：

``` js
'aaa*a*'.split(/(a*)/)
// [ '', 'aaa', '*', 'a', '*' ]
```

### 正则匹配规则

#### 1. 字面量字符和元字符

- 字面量字符：比如`/a/`匹配`a`，`/b/`匹配`b`；
- 元字符：一些有特殊含义的字符；

1. 点字符（`.`）：匹配除回车（`\r`）、换行(`\n`) 、行分隔符（`\u2028`）和段分隔符（`\u2029`）以外的所有字符；

2. 位置字符用来提示字符所处的位置，主要有两个字符：

- `^` 表示字符串的开始位置

- `$` 表示字符串的结束位置

```js
const getCookie = (name) => {
  let arrd = null;
  // ^和$可以被当做一个特殊字符来匹配
  // 在cookie字符串中，开头有可能是空格，但是也有可能没有空格，因为它可能是开头
  // 而结尾可能是分号，但是也有可能没有分号，因为结尾的可以没有分号
  const reg = new RegExp(`(^| )${name}=([^;]*)(;|$)`);
  if (document.cookie.match(reg)) {
    arrd = document.cookie.match(reg);
    return unescape(arrd[2]);
  }
  return null;
};
```

3. 选择符（`|`）：在正则表达式中表示“或关系”（OR）；

   选择符会包括它前后的多个字符，比如`/ab|cd/`指的是匹配`ab`或者`cd`，而不是指匹配`b`或者`c`。如果想修改这个行为，可以使用圆括号。

   ``` js
   /a( |\t)b/.test('a\tb') 
   // true
   // 上面代码指的是，a和b之间有一个空格或者一个制表符
   ```

> 其他的元字符还包括`\`、`*`、`+`、`?`、`()`、`[]`、`{}`等

#### 2. 转义符

正则表达式中那些有特殊含义的元字符，如果要匹配它们本身，就需要在它们前面要加上反斜杠。比如要匹配`+`，就要写成`\+`。

``` js
/1+1/.test('1+1')
// false

/1\+1/.test('1+1')
// true
```

正则表达式中，需要反斜杠转义的，一共有12个字符：`^`、`.`、`[`、`$`、`(`、`)`、`|`、`*`、`+`、`?`、`{`和`\`。需要特别注意的是，如果使用`RegExp`方法生成正则对象，转义需要使用两个斜杠，因为在字符串内部，反斜杠也是转义字符，所以它会先被反斜杠转义一次，然后再被正则表达式转义一次，因此需要两个反斜杠转义。

``` js
(new RegExp('1\+1')).test('1+1')
// false

(new RegExp('1\\+1')).test('1+1')
// true
```

#### 3. 特殊字符

正则表达式对一些不能打印的特殊字符，提供了表达方法。

- `\cX` 表示`Ctrl-[X]`，其中的`X`是A-Z之中任一个英文字母，用来匹配控制字符。
- `[\b]` 匹配退格键(U+0008)，不要与`\b`混淆。
- `\n` 匹配换行键。
- `\r` 匹配回车键。
- `\t` 匹配制表符 tab（U+0009）。
- `\v` 匹配垂直制表符（U+000B）。
- `\f` 匹配换页符（U+000C）。
- `\0` 匹配`null`字符（U+0000）。
- `\xhh` 匹配一个以两位十六进制数（`\x00`-`\xFF`）表示的字符。
- `\uhhhh` 匹配一个以四位十六进制数（`\u0000`-`\uFFFF`）表示的 Unicode 字符。

#### 4. 字符类

字符类（class）表示有一系列字符可供选择，只要匹配其中一个就可以了。所有可供选择的字符都放在方括号内，比如`[xyz]` 表示`x`、`y`、`z`之中任选一个匹配。有两个字符在字符类中有特殊含义(其它字符，包括元字符，在字符类中都没有特殊含义。如果要匹配\，由于它自己本身表示转义符，所以应该写成\\\\)。

1. 如果方括号内的第一个字符是`[^]`，则表示除了字符类之中的字符，其他字符都可以匹配。比如，`[^xyz]`表示除了`x`、`y`、`z`之外都可以匹配。如果方括号内没有其他字符，即只有`[^]`，就表示匹配一切字符，其中包括换行符。相比之下，点号作为元字符（`.`）是不包括换行符的。

   ``` js
   var s = 'Please yes\nmake my day!';
   
   s.match(/yes.*day/) // null
   s.match(/yes[^]*day/) // [ 'yes\nmake my day']
   ```

   > 注意，脱字符只有在字符类的第一个位置才有特殊含义，否则就是字面含义。
   >
   > [^]匹配一切字符，包括换行符

2. 对于连续序列的字符，连字符（`-`）用来提供简写形式，表示字符的连续范围。比如，`[abc]`可以写成`[a-c]`，`[0123456789]`可以写成`[0-9]`，同理`[A-Z]`表示26个大写字母。如果连字号（dash）不出现在方括号之中，就不具备简写的作用，只代表字面的含义。

   ``` js
   [0-9.,]
   [0-9a-fA-F]
   [a-zA-Z0-9-]
   [1-31]  // 只代表1-3
   ```

   不要过分使用连字符，设定一个很大的范围，否则很可能选中意料之外的字符。最典型的例子就是`[A-z]`，表面上它是选中从大写的`A`到小写的`z`之间52个字母，但是由于在 ASCII 编码之中，大写字母与小写字母之间还有其他字符，结果就会出现意料之外的结果。

   ``` js
   /[A-z]/.test('\\') 
   // true
   // 由于反斜杠（'\'）的ASCII码在大写字母与小写字母之间，结果会被选中
   ```

**字符类使用实例：**

``` js
// 匹配/1-31/
/^[1-9]$|^[12]\d$|^3[01]$/

//匹配0-255
/^\d$|^[1-9]\d$|^1\d{2}$|^2[0-4]\d$|^25[0-5]$/  

// 0.0.0.0-255.255.255.255 匹配IP地址
/^(\d|[1-9]\d|1\d{2}|2[0-4]\d|25[0-5])(\.(\d|[1-9]\d|1\d{2}|2[0-4]\d|25[0-5])){3}$/

// 匹配中文
/[\u4e00-\u9fd5]{2,4}/
```

#### 5. 预定义模式

预定义模式指的是某些常见模式的简写方式。

- `\d` 匹配0-9之间的任一数字，相当于`[0-9]`。
- `\D` 匹配所有0-9以外的字符，相当于`[^0-9]`。
- `\w` 匹配任意的字母、数字和下划线，相当于`[A-Za-z0-9_]`。
- `\W` 除所有字母、数字和下划线以外的字符，相当于`[^A-Za-z0-9_]`。
- `\s` 匹配空格（包括换行符、制表符、空格符等），相等于`[ \t\r\n\v\f]`。
- `\S` 匹配非空格的字符，相当于`[^ \t\r\n\v\f]`。
- `\b` 匹配词的边界。
- `\B` 匹配非词边界，即在词的内部。

``` js
// \s 的例子
// \s表示空格，所以匹配结果会包括空格
/\s\w*/.exec('hello world') // [" world"]

// \b 的例子
// \b表示词的边界，所以world的词首必须独立才会匹配
/\bworld/.test('hello world') // true
/\bworld/.test('hello-world') // true
/\bworld/.test('helloworld') // false

// \B 的例子
// \B表示非词的边界，只有world的词首不独立才会匹配
/\Bworld/.test('hello-world') // false
/\Bworld/.test('helloworld') // true
```

通常，正则表达式遇到换行符（`\n`）就会停止匹配，但是可以使用\s来匹配换行：

``` js
var html = "<b>Hello</b>\n<i>world!</i>";

/.*/.exec(html)[0]
// "<b>Hello</b>"

/[\S\s]*/.exec(html)[0]    
// "<b>Hello</b>\n<i>world!</i>"
```

> [\S\s]指代一切字符，包括换行符

#### 6. 重复类

模式的精确匹配次数，使用大括号（`{}`）表示。`{n}`表示恰好重复`n`次，`{n,}`表示至少重复`n`次，`{n,m}`表示重复不少于`n`次，不多于`m`次。

``` js
/lo{2}k/.test('look') // true
/lo{2,5}k/.test('looook') // true
```

#### 7. 量词符

量词符用来设定某个模式出现的次数。

- `?` 问号表示某个模式出现0次或1次，等同于`{0, 1}`。
- `*` 星号表示某个模式出现0次或多次，等同于`{0,}`。
- `+` 加号表示某个模式出现1次或多次，等同于`{1,}`。

``` js
// t 出现0次或1次
/t?est/.test('test') // true
/t?est/.test('est') // true
/^t?est/.test('ttest') // false

// t 出现1次或多次
/t+est/.test('test') // true
/t+est/.test('ttest') // true
/t+est/.test('est') // false

// t 出现0次或多次
/t*est/.test('ttest') // true
/t*est/.test('tttest') // true
/t*est/.test('est') // true
```

#### 8. 贪婪模式

正则表达式中的量词符，默认情况下都是最大可能匹配，即匹配到下一个字符不满足匹配规则为止，这被称为贪婪模式。

``` js
var s = 'aaa';
s.match(/a+/) // ["aaa"]
```

除了贪婪模式，还有非贪婪模式，即最小可能匹配。只要一发现匹配，就返回结果，不要往下检查。如果想将贪婪模式改为非贪婪模式，可以在量词符后面加一个问号。

``` js
var s = 'aaa';
s.match(/a+?/) // ["a"]
```

- `+?`：表示某个模式出现1次或多次，匹配时采用非贪婪模式。
- `*?`：表示某个模式出现0次或多次，匹配时采用非贪婪模式。
- `??`：表格某个模式出现0次或1次，匹配时采用非贪婪模式。

``` js
'abb'.match(/ab*/) // ["abb"]
'abb'.match(/ab*?/) // ["a"]

'abb'.match(/ab?/) // ["ab"]
'abb'.match(/ab??/) // ["a"]
```

#### 9. 修饰符

1. `g` 修饰符：

   默认情况下，第一次匹配成功后，正则对象就停止向下匹配了。`g`修饰符表示全局匹配（global），加上它以后，正则对象将匹配全部符合条件的结果，主要用于搜索和替换。

2. `i` 修饰符：

   默认情况下，正则对象区分字母的大小写，加上`i`修饰符以后表示忽略大小写（ignoreCase）。

3. `m` 修饰符：

   `m`修饰符表示多行模式（multiline），会修改`^`和`$`的行为。默认情况下（即不加`m`修饰符时），`^`和`$`匹配字符串的开始处和结尾处，加上`m`修饰符以后，`^`和`$`还会匹配行首和行尾，即`^`和`$`会识别换行符（`\n`）。

   ``` js
   /world$/.test('hello world\n') // false
   /world$/m.test('hello world\n') // true
   /^b/m.test('a\nb') // true
   ```

4. `y`修饰符：

   `y`修饰符的作用与`g`修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，`g`修饰符只要剩余位置中存在匹配就可，而`y`修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的涵义。

   ``` js
   var s = 'aaa_aa_a';
   var r1 = /a+/g;
   var r2 = /a+/y;
   
   r1.exec(s) // ["aaa"]
   r2.exec(s) // ["aaa"]
   
   r1.exec(s) // ["aa"]
   r2.exec(s) // null
   ```

   `lastIndex`属性指定每次搜索的开始位置，`g`修饰符从这个位置开始向后搜索，直到发现匹配为止。`y`修饰符同样遵守`lastIndex`属性，但是要求必须在`lastIndex`指定的位置发现匹配。

   实际上，`y`修饰符号隐含了头部匹配的标志`^`。`y`修饰符的设计本意，就是让头部匹配的标志`^`在全局匹配中都有效。

   ``` js
   const REGEX = /a/gy;
   'aaxa'.replace(REGEX, '-') // '--xa'
   
   'a1a2a3'.match(/a\d/y) // ["a1"]
   'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
   // 单单一个y修饰符对match方法，只能返回第一个匹配
   // 必须与g修饰符联用，才能返回所有匹配
   ```

   `y`修饰符的一个应用，是从字符串提取 token（词元），`y`修饰符确保了匹配之间不会有漏掉的字符。

   ``` js
   const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
   const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;
   
   tokenize(TOKEN_Y, '3 + 4')
   // [ '3', '+', '4' ]
   tokenize(TOKEN_G, '3 + 4')
   // [ '3', '+', '4' ]
   tokenize(TOKEN_Y, '3x + 4')
   // [ '3' ]
   tokenize(TOKEN_G, '3x + 4')
   // [ '3', '+', '4' ]
   
   function tokenize(TOKEN_REGEX, str) {
     let result = [];
     let match;
     while (match = TOKEN_REGEX.exec(str)) {
       result.push(match[1]);
     }
     return result;
   }
   ```

   上面代码中，如果字符串里面没有非法字符，`y`修饰符与`g`修饰符的提取结果是一样的。但是，一旦出现非法字符，两者的行为就不一样了。`g`修饰符会忽略非法字符，而`y`修饰符不会，这样就很容易发现错误。

5. `s`修饰符：dotALL模式

   正则表达式中，点（`.`）是一个特殊字符，代表任意的单个字符，但是不会匹配终止符：

   - U+000A 换行符（`\n`）
   - U+000D 回车符（`\r`）
   - U+2028 行分隔符（line separator）
   - U+2029 段分隔符（paragraph separator）

   如果希望匹配的是任意单个字符，可以使用[\S\s]或[^]，但是这种方案看起来不太直观，因此ES2018 引入`s`修饰符，使得`.`可以匹配任意单个字符。

   > `/s`修饰符和多行修饰符`/m`不冲突，两者一起使用的情况下，`.`匹配所有字符，而`^`和`$`匹配每一行的行首和行尾。

6. `d`修饰符：

   这个修饰符可以让`exec()`、`match()`的返回结果添加`indices`属性，在该属性上面可以拿到匹配结果和所有组匹配的开始位置和结束位置。

   ``` js
   const text = 'zabbcdef';
   const re = /ab+(cd(ef))/d;
   const result = re.exec(text);
   
   result.indices // [ [1, 8], [4, 8], [6, 8] ]
   ```

   如果正则表达式包含具名组匹配，`indices`属性数组还会有一个`groups`属性。该属性是一个对象，可以从该对象获取具名组匹配的开始位置和结束位置。

   如果获取组匹配不成功，`indices`属性数组的对应成员则为`undefined`，`indices.groups`属性对象的对应成员也是`undefined`。

   ``` js
   const text = 'zabbcdef';
   const re = /ab+(?<Z>cd)/d;
   const result = re.exec(text);
   
   result.indices.groups // { Z: [ 4, 6 ] }
   ```

#### 10. 组匹配

正则表达式的括号表示分组匹配，括号中的模式可以用来匹配分组的内容。

``` js
/fred+/.test('fredd') // true
/(fred)+/.test('fredfred') // true

// 第一个模式没有括号，结果+只表示重复字母d
// 第二个模式有括号，结果+就表示匹配fred这个词
```

使用组匹配时，不宜同时使用`g`修饰符，否则`match`方法不会捕获分组的内容。

``` js
var m = 'abcabc'.match(/(.)b(.)/);
// ['abc', 'a', 'c']

var m = 'abcabc'.match(/(.)b(.)/g);
// ['abc', 'abc']
```

正则表达式内部，还可以用`\n`引用括号匹配的内容，`n`是从1开始的自然数，表示对应顺序的括号。

**`\n`使用实例：**

``` js
// 分割字符串
var str = 'aabbbbbbdddddeeeeeef'
str.match(/(\w)\1*/g)

// 统计第一个字符的个数
var str = 'hjhjds jksjsdkj alkjd jksdj kswdjli qwdo qijd aksijdais  asdiuaiusd asdiuasdkjh'
const o = str
  .split('')
  .sort()
  .join('')
  .match(/(.)\1*/g)
  .reduce(function (v, t) {
    v[t[0]] = t.length
    return v
  }, {})
```

``` js
/(.)b(.)\1b\2/.test("abcabc")
// true
// \1表示第一个括号匹配的内容（即a），\2表示第二个括号匹配的内容（即c）

/y(..)(.)\2\1/.test('yabccab') 
// true

/y((..)\2)\1/.test('yabababab') 
// true
// \1指向外层括号，\2指向内层括号
```

``` js
// 匹配标签
var tagName = /<([^>]+)>[^<]*<\/\1>/;
tagName.exec("<b>bold</b>")[1]  // 'b'

// 匹配带样式的标签
var html = '<b class="hello">Hello</b><i>world</i>';
var tag = /<(\w+)([^>]*)>(.*?)<\/\1>/g;

var match = tag.exec(html);

match[1] // "b"
match[2] // " class="hello""
match[3] // "Hello"

match = tag.exec(html);

match[1] // "i"
match[2] // ""
match[3] // "world"
```

如果要在正则表达式内部引用某个“具名组匹配”，可以使用`\k<组名>`的写法。数字引用（`\1`）依然有效。这两种引用语法还可以同时使用。

``` js
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
RE_TWICE.test('abc!abc!abc') // true
RE_TWICE.test('abc!abc!ab') // false
```

- **具名组匹配**

  具名组匹配允许为每一个组匹配指定一个名字，既便于阅读代码，又便于引用。

  ```js
  const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
  
  const matchObj = RE_DATE.exec('1999-12-31');
  const year = matchObj.groups.year; // "1999"
  const month = matchObj.groups.month; // "12"
  const day = matchObj.groups.day; // "31"
  
  // 使用解构语法
  let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
  one  // foo
  two  // bar
  ```

  可以在`exec`方法返回结果的`groups`属性上引用该组名。同时，数字序号（`matchObj[1]`）依然有效。具名组匹配等于为每一组匹配加上了 ID，便于描述匹配的目的。如果组的顺序变了，也不用改变匹配后的处理代码。如果具名组没有匹配，那么对应的`groups`对象属性会是`undefined`。

- **非捕获组**

  `(?:x)`称为非捕获组（Non-capturing group），表示不返回该组匹配的内容，即匹配的结果中不计入这个括号。

  非捕获组的作用请考虑这样一个场景，假定需要匹配`foo`或者`foofoo`，正则表达式就应该写成`/(foo){1, 2}/`，但是这样会占用一个组匹配。这时，就可以使用非捕获组，将正则表达式改为`/(?:foo){1, 2}/`，它的作用与前一个正则是一样的，但是不会单独输出括号内部的内容。

  ``` js
  // 正常匹配
  var url = /(http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/;
  
  url.exec('http://google.com/');
  // ["http://google.com/", "http", "google.com", "/"]
  
  // 非捕获组匹配
  var url = /(?:http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/;
  
  url.exec('http://google.com/');
  // ["http://google.com/", "google.com", "/"]
  ```

- **先行断言**

  `x(?=y)`称为先行断言（Positive look-ahead），`x`只有在`y`前面才匹配，`y`不会被计入返回结果。比如，要匹配后面跟着百分号的数字，可以写成`/\d+(?=%)/`。“先行断言”中，括号里的部分是不会返回的。

  ``` js
  var m = 'abc'.match(/b(?=c)/);  // ["b"]
  ```

  **先行断言的妙用**

  先行断言放在正则表达式的最前方，由于先行断言前方没有内容，则会在整个字符串中进行匹配，表示必须有内容在先行断言内容的前方，即可以被用作表示该字符串中必须包含先行断言内容

  ``` js
  // (?=\D+\d) 起始字符开始首字母不能是数字，但是在整个字符串中必须包含数字
  // (?=.*[a-z]) 在任意位置包含a-z的小写字母
  // (?=.*[A-Z]) 在任意位置包含A-Z的大写字母
  // [a-zA-Z0-9_-$&!]{8,16} 密码实际包含的字符，要求最少8位，最大16位
  
  // 高级密码
  ;/^(?=\D+\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9_\-$&!]{8,16}$/.test('xietianabc')
  
  // 中级密码
  ;/^(?=\D+\d)(?=.*[a-z])[a-zA-Z0-9_\-$&!]{8,16}$|^(?=\D+\d)(?=.*[A-Z])[a-zA-Z0-9_\-$&!]{8,16}$|^(?=.*[A-Z])(?=.*[a-z])[a-zA-Z0-9_\-$&!]{8,16}$/.test(
    'xietian12'
  )
  // 初级密码
  ;/^\d{8,16}$|^[a-z]{8,16}$|^[A-Z]{8,16}$/.test('xietianabc')
  ```

- **先行否定断言**

  `x(?!y)`称为先行否定断言（Negative look-ahead），`x`只有不在`y`前面才匹配，`y`不会被计入返回结果。比如，要匹配后面跟的不是百分号的数字，就要写成`/\d+(?!%)/`。“先行否定断言”中，括号里的部分是不会返回的。

  ``` js
  /\d+(?!\.)/.exec('3.14')  // ["14"]
  ```

  **先行否定断言的妙用**

  先行否定断言放在正则表达式的最前方，由于先行否定断言前方没有内容，则会在整个字符串中进行匹配，表示必须有内容不在先行否定断言的前方，即可以被用作表示该字符串中不用全部是先行否定断言的内容

  ```js
  // ^(?![A-Za-z]+$)表示从头到位不能全是字母
  // ^(?![0-9]+$)表示从头到位不能全是数字
  // ^(?![`-=\]\[\\‘;/.,~!@#$%^&()_+|}{“:?><]+$)表示从头到位不能全是符号@#$
  // ^[0-9A-Za-z`-=\]\[\\‘;/.,~!@#$%^&()_+|}{“:?><]{8,16}$表示从头到位只能是字母数字符号@#$的集合
  
  // 需要注意的是，开始符“^”和预搜索“(?!)”都是零宽的，表示位置，所以开始符“^”只需要在整个正则表达式的开始处写一个即可
  
  // 校验密码，至少包含字母数字和特殊字符中的两种
  /^(?![0-9]+$)(?![a-zA-Z]+$)(?![`\-=\]\[‘;/.,~!@#$%^&()_+|}{“:?><]+$)[0-9A-Za-z`\-=\]\[‘;/.,~!@#$%^&()_+|}{“:?><]{8,16}$/
  ```

- **后行断言**

  “后行断言”正好与“先行断言”相反，`x`只有在`y`后面才匹配，必须写成`/(?<=y)x/`。比如，只匹配美元符号之后的数字，要写成`/(?<=\$)\d+/`。“后行断言”中，括号里的部分是不会返回的。

  ``` js
  /(?<=\$)\d+/.exec('Benjamin Franklin is on the $100 bill')  // ["100"]
  ```

  “后行断言”的实现，需要先匹配`/(?<=y)x/`里的`x`，然后再回到左边，匹配`y`的部分。这种“先右后左”的执行顺序，与所有其他正则操作相反，导致了一些不符合预期的行为。

   首先，后行断言的组匹配，与正常情况下结果是不一样的。

  ```
  /(?<=(\d+)(\d+))$/.exec('1053') // ["", "1", "053"]
  /^(\d+)(\d+)$/.exec('1053') // ["1053", "105", "3"]
  ```

  上面代码中，需要捕捉两个组匹配。没有“后行断言”时，第一个括号是贪婪模式，第二个括号只能捕获一个字符，所以结果是`105`和`3`。而“后行断言”时，由于执行顺序是从右到左，第二个括号是贪婪模式，第一个括号只能捕获一个字符，所以结果是`1`和`053`。

  其次，“后行断言”的反斜杠引用，也与通常的顺序相反，必须放在对应的那个括号之前。

  ```
  /(?<=(o)d\1)r/.exec('hodor')  // null
  /(?<=\1d(o))r/.exec('hodor')  // ["r", "o"]
  ```

  上面代码中，如果后行断言的反斜杠引用（`\1`）放在括号的后面，就不会得到匹配结果，必须放在前面才可以。因为后行断言是先从左到右扫描，发现匹配以后再回过头，从右到左完成反斜杠引用。

- **后行否定断言**

  “后行否定断言”则与“先行否定断言”相反，`x`只有不在`y`后面才匹配，必须写成`/(?<!y)x/`。比如，只匹配不在美元符号后面的数字，要写成`/(?<!\$)\d+/`。“后行否定断言”中，括号里的部分是不会返回的。

  ``` js
  /(?<!\$)\d+/.exec('it’s is worth about €90')  // ["90"]
  ```

  

