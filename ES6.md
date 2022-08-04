#### eval()

`eval()` 函数会将传入的字符串当做 JavaScript 代码进行执行。

- 参数

  `string`：一个表示 JavaScript 表达式、语句或一系列语句的字符串。表达式可以包含变量与已存在对象的属性。

- 返回值

  返回字符串中代码的返回值。如果返回值为空，则返回 `undefined`。

``` js
const o1 = {}
const o2 = {}
const o3 = {}
for (let i = 1; i < 4; i++) {
  // 相当于传给eval的是字符串'o1'，然后解析后就变成了o1对象
  eval('o' + i).a = 10
}
console.log(o1, o2, o3)
// { a: 10 } { a: 10 } { a: 10 }
```

> eval()函数在es6中的改变：

正常模式下，JavaScript 语言有两种变量作用域（scope）：全局作用域和函数作用域。严格模式创设了第三种作用域：`eval`作用域。

正常模式下，`eval`语句的作用域，取决于它处于全局作用域，还是函数作用域。严格模式下，`eval`语句本身就是一个作用域，不再能够在其所运行的作用域创设新的变量了，也就是说，`eval`所生成的变量只能用于`eval`内部。

```js
(function () {
  'use strict';
  var x = 2;
  console.log(eval('var x = 5; x')) // 5
  console.log(x) // 2
})()
```

上面代码中，由于`eval`语句内部是一个独立作用域，所以内部的变量`x`不会泄露到外部。注意，如果希望`eval`语句也使用严格模式，有两种方式。

```js
// 方式一
function f1(str){
  'use strict';
  return eval(str);
}
f1('undeclared_variable = 1'); // 报错

// 方式二
function f2(str){
  return eval(str);
}
f2('"use strict";undeclared_variable = 1')  // 报错
```

严格模式下，使用`eval`或`arguments`作为标识名将会报错：

```js
'use strict';

var eval = 17;
var arguments = 17
// SyntaxError: Unexpected eval or arguments in strict mode
```

#### 永远不要使用 eval！
eval() 是一个危险的函数， 它使用与调用者相同的权限执行代码。如果你用 eval() 运行的字符串代码被恶意方（不怀好意的人）修改，您最终可能会在您的网页/扩展程序的权限下，在用户计算机上运行恶意代码。更重要的是，第三方代码可以看到某一个 eval() 被调用时的作用域，这也有可能导致一些不同方式的攻击。相似的 Function 就不容易被攻击。

eval() 通常比其他替代方法更慢，因为它必须调用 JS 解释器，而许多其他结构则可被现代 JS 引擎进行优化。

此外，现代 JavaScript 解释器将 JavaScript 转换为机器代码。这意味着任何变量命名的概念都会被删除。因此，任意一个 eval 的使用都会强制浏览器进行冗长的变量名称查找，以确定变量在机器代码中的位置并设置其值。另外，新内容将会通过 eval() 引进给变量，比如更改该变量的类型，因此会强制浏览器重新执行所有已经生成的机器代码以进行补偿。但是（谢天谢地）存在一个非常好的 eval 替代方法：只需使用 window.Function。这有个例子方便你了解如何将eval()的使用转变为Function()。

#### 使用Function()替代eval()：

>  Function构造函数和函数声明的区别：

使用构造函数创建的Function函数不会为其创建上下文闭包，它们总是在全局范围内创建。运行它们时，它们将只能访问自己的局部变量和全局变量，而不能访问Function创建构造函数的范围内的变量。这与使用eval()函数表达式的代码不同。

``` js
// Create a global property with `var`
var x = 10;

function createFunction1() {
  const x = 20;
  return new Function('return x;'); // this `x` refers to global `x`
}

function createFunction2() {
  const x = 20;
  function f() {
    return x; // this `x` refers to the local `x` above
  }
  return f;
}

const f1 = createFunction1();
console.log(f1());          // 10
const f2 = createFunction2();
console.log(f2());          // 20
```

虽然此代码在 Web 浏览器中可以正常工作，但将在 Node.js 中f1()会报错，提示因为找不到变量 ‘x’ (ReferenceErrorxx)。这是因为在 Node 环境中的顶级作用域不是全局作用域，而是模块的本地作用域。

> eval和Function使用对比：

使用 eval 的糟糕代码：

```js
function looseJsonParse(obj){
    return eval("(" + obj + ")");
}
console.log(looseJsonParse(
   "{a:(4-1), b:function(){}, c:new Date()}"
))
```

不用 eval 的更好的代码：

```js
function looseJsonParse(obj){
    return Function('"use strict";return (' + obj + ')')();
}
console.log(looseJsonParse(
   "{a:(4-1), b:function(){}, c:new Date()}"
))
```

#### module中自动打开严格模式：

``` js
<script type="module">
    // 自动严格模式
</script>
```

#### generator实现阻塞同步：

``` js
// 强制让异步停下来不再执行，等待异步完成后再继续向后执行
function* fn() {
  res = yield setTimeout(function () {
    g.next(10)
  }, 2000)
  console.log(res)
}

const g = fn()
console.log(g.next())
// {value: 1, done: false} 计时器id
```

``` js
// 阻塞式同步
function* loadImage() {
  var arr = []
  for (var i = 78; i < 83; i++) {
    var img = new Image()
    img.src = `./img/img_${i}.jpeg`
    yield (img.onload = function () {
      arr.push(this)
      g.next()
    })
  }
  arr.forEach((item) => console.log(item.src))
}

var g = loadImage()
g.next()
```

#### 解构面试题：

``` js
// 题目一：使用同一条解构语句，使其能够解构下面的四个对象，并且解构出来的值要符合标准
var obj = { a: 10, b: { b: { a: 20 } } }
// a=10  a1=20;
var obj = { a: 10, b: { b: {} } }
// a=10 a1=100
var obj = { b: {} }
// a=20 a1=0;
var obj = {}
// a=20  a1=-1

var { a = 20, b: { b: { a: a1 = 100 } = { a: 0 } } = { b: { a: -1 } } } = obj
console.log(a, a1)

// 题目二：写出下列函数调用时a和a1的值
function fn({ a, b: { a: a1 } = { a: 10 } } = { a: 20, b: { a: 30 } }) {
  console.log(a, a1)
}

fn() //20,30
fn({}) //undefined 10
fn({ a: 1 }) //1 10
fn({ a: 1, b: {} }) //1 undefined
fn({ a: 1, b: { a: 50 } }) //1 50
```

> 解构原生对象中的属性和方法：

``` js
// 解构出数组中的len属性
var str = 'abvcde'
var { length: len } = str
console.log(len) //6
// 解构Math对象的静态方法
var { random, max } = Math
console.log(random() * 10)
console.log(max(10, 20))
// 有的对象中使用了this，这种对象不能进行解构，如果解构后，this的指向就会发生改变
var arr = [1, 2, 3, 4]
var { push } = arr
```

#### this指向问题：

1. 全局中的this：指向全局对象

2. 函数调用时this的指向：在非严格模式下指向window，严格模式下指向undefined

3. 普通回调函数中this指向

   ``` js
   var obj = {
     a: function (f) {
       f()
     },
     b: function () {
       console.log(this) //在非严格模式下指向window，严格模式下指向undefined
     }
   }
   obj.a(obj.b)
   ```

4. 通过arguments回调的函数

   ``` js
   var o = {
     a: function (f) {
       arguments[0]()
     },
     b: function () {
       console.log(this)
     }
   }
   
   o.a(o.b)
   ```

5. 事件侦听时调用的回调函数中的this指向

   ``` js
   var obj = {
     a: function () {
       document.addEventListener('click', this.clickhandler)
     },
     clickhandler(e) {
       console.log(this) //谁侦听，this指向谁，侦听的对象
     }
   }
   obj.a()
   
   var obj = {
     a: function () {
       document.addEventListener('click', () => this.clickhandler())
     },
     clickhandler(e) {
       console.log(this) // this指向obj
     }
   }
   obj.a()
   ```

   ``` js
   const input = document.getElementById('txt')
   input.addEventListener('input', inputHandler)
   
   function inputHandler(e) {
     setTimeout(function () {
       console.log(this) // window
     })
     setTimeout(() => {
       console.log(this) // input
     })
   }
   ```

6. 对象中的this指向

   ``` js
   var b = 3
   var obj = {
     b: 2,
     a: this.b, //属性中this指向当前对象外的this指向
     c: function () {
       console.log(this) //指向当前对象自身
     },
     d() {
       console.log(this) //指向当前对象自身
     }
   }
   console.log(obj.a) // 3
   obj.c() // {b: 2, a: 3, c: ƒ, d: ƒ}
   obj.d() // {b: 2, a: 3, c: ƒ, d: ƒ}
   ```

7. 数组中的部分方法中有thisArg参数的方法，this将会被替代指向

   ``` js
   var arr = [1, 2, 3, 4]
   
   arr.forEach(
     function () {
       console.log(this)
     },
     { a: 1 }
   )
   ```

8. 箭头函数中this的指向

   ``` js
   var obj = {
     a: function () {
       var fn = () => {
         console.log(this) //obj
       }
       fn()
     }
   }
   obj.a()
   
   document.addEventListener('click', () => {
     console.log(this) //window
   })
   ```

#### 使用symbol去除魔术字符串：

> 魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。风格良好的代码，应该尽量消除魔术字符串，改由含义清晰的变量代替。

``` js
const LEFT = Symbol(),
  TOP = Symbol(),
  RIGHT = Symbol(),
  BOTTOM = Symbol()
var list = [LEFT, TOP, RIGHT, BOTTOM]
var state
init()

function init() {
  document.addEventListener('keydown', keyHandler)
  document.addEventListener('keyup', keyHandler)
  setInterval(animation, 16)
}

function keyHandler(e) {
  if (e.type === 'keyup') return (state = undefined)
  state = list[e.keyCode - 37]
  state = TOP
}

function animation() {
  if (!state) return
  switch (state) {
    case LEFT:
      console.log('left')
      break
    case TOP:
      console.log('top')
      break
    case RIGHT:
      console.log('right')
      break
    case BOTTOM:
      console.log('bottom')
      break
  }
}
```

#### set的运用：

> 使用set实现不重复点击所有按钮：

``` js
const s = new Set()
document.addEventListener('click', clickHandler)

function clickHandler(e) {
  if (e.target.nodeName !== 'BUTTON') return
  s.add(e.target)
  if (s.size === 3) {
    console.log('全部点击完成')
  }
}
```

> 使用数组与set做事件发布的对比

``` js
var arr = []
function add(fn) {
  if (arr.includes(fn)) return
  arr.push(fn)
}
function remove(fn) {
  var index = arr.indexOf(fn)
  arr.splice(index, 1)
}
function dispatch() {
  for (var i = 0; i < arr.length; i++) {
    arr[i]()
  }
}

function fn1() {
  console.log('fn1')
}
function fn2() {
  console.log('fn2')
  remove(fn2)
}
function fn3() {
  console.log('fn3')
}

add(fn1)
add(fn1)
add(fn2)
add(fn3)

dispatch()
// fn1
// fn2
```

``` js
var arr = new Set()
function add(fn) {
  arr.add(fn)
}
function remove(fn) {
  if (arr.has(fn)) arr.delete(fn)
}
function dispatch() {
  for (var fn of arr) {
    fn()
  }
}

function fn1() {
  console.log('fn1')
}
function fn2() {
  console.log('fn2')
  remove(fn2)
}
function fn3() {
  console.log('fn3')
}

add(fn1)
add(fn1)
add(fn2)
add(fn3)

dispatch()
console.log('----split----')
dispatch()

// /fn1
// fn2
// fn3
// ----split----
// fn1
// fn3
```

#### map的使用

``` js
var m = new Map([
  ['a', 1],
  ['b', 2]
])
console.log(m) // Map(2) {'a' => 1, 'b' => 2}

var o = { a: 1, b: 2, c: 3 }
console.log(Object.entries(o)) // (3) [Array(2), Array(2), Array(2)]
console.log(Object.fromEntries(m)) // {a: 1, b: 2}
```

> 遍历map

``` js
const m = new Map([
  ['a', 1],
  ['b', 2]
])

for (var [key, value] of m) {
  console.log(key, value)
}

for (var key of m.keys()) {
  console.log(key)
}

for (var value of m.values()) {
  console.log(value)
}

for (var [key, value] of m.entries()) {
  console.log(key, value)
}
```

> 用map实现用户按顺序点击按钮：

``` js
// 回调地狱
var bns = document.querySelectorAll('button')
bns[0].onclick = function () {
  bns[1].onclick = function () {
    bns[2].onclick = function () {}
  }
}

// 使用map
var m = new Map()
var bns = document.querySelectorAll('button')
m.set(bns[0], bns[1])
m.set(bns[1], bns[2])
m.set(bns[2], bns[3])
m.set(bns[3], bns[4])

bns[0].addEventListener('click', clickHandler)

function clickHandler(e) {
  this.removeEventListener('click', clickHandler)
  if (!m.has(this)) return
  m.get(this).addEventListener('click', clickHandler)
}
```



