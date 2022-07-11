## ES5中的继承（引出问题）

首先写一个 Creature 类表示生物，生物有类型 type

```js
function Creature(type) {
    this.type = type
} 
```

现在我们实现宠物类 Pet，继承自生物 Creature 宠物比生物多出一个 name 属性

```js
function Pet(type, name) {
    // 这么写的目的是：
    // 既然 Pet 是 Creature 的子类，
    // 那么我们先用父类的构造函数将一个全新的对象“构造”为一个“生物”
    // 再对其进行“加工”将其变成一个“宠物”
    Creature.call(this, type)

    //“加工”
    this.name = name
}

// 设置原型链的继承关系以继承方法
// 不过我们讨论的问题与原型上的方法没有什么关系
// 这里就不给原型上添加方法了，只是象征性的这么设置一下
Object.setPrototypeOf(Pet.prototype, Creature.prototype)

var myCat = new Pet('cat', '欧拉白猫')
var myDog = new Pet('dog', '哈弗大狗')
```

在这种写法下，我们的继承将会如期工作：Pet 构造函数会借用父类的构造函数将 this 初始化为一个 Creature，在此之前 this 为一个空对象，经过 Creature.call 以后，this 即变为了一个“生物”，即 this 上将会有表示“生物”所需要的属性。同时 this 的原型为 Pet.prototype，而结合后一行的 setPrototypeOf，则原型的原型为 Creature.prototype，所以this上也可以正常调用从Creature继承来的方法。

这种写法下，Creature.call 那一行之前是完全可以使用this的，毕竟当我们 new Pet 的时候，Pet 函数里的 this 已经指向了一个全新的空对象。直觉上我们会觉得 Creature.call(this) 就相当于 class 语法中的 super()，很多时候这么认为也不会有什么问题。

那么如果这么认为的话，会觉得在 class 里 super() 之前不能用 this 的设计很奇怪，因为明明Creature.call(this) 之前就可以，为什么相同的 super() （实际上不同）前不能呢。然而这个设计是必然的，现在考虑我们想继承一个原生的类型，比如正则表达式吧：

```js
function MyRegExp(pattern, flags) {
  RegExp.call(this, pattern, flags)

  this.type = 'MyRegExp'
}

MyRegExp.prototype.exec = RegExp.prototype.exec
```

上面的代码看似一切正常，但当你真正执行 var myre = new MyRegExp('foo','igm') 以后，this 还是原来那个 this，上面并没有增加任何正则该有的属性，比如 lastIndex, global 等，**也就是说 RegExp.call 根本没对 this 做任何事情**，而当你执行 myre.exec('foobar') 时，则会收到一个报错，意为 exec 在错误的对象上调用（Chrome 中是 Method RegExp.prototype.exec called on incompatible receiver），此时 exec 里的 this 对象是我们的 myre 指向的对象，但其实它并不是一个正则表达式对象，所以才报了 incompatible receiver，即exec需要一个正则对象，可它不是。

>  <u>发生了什么呢？</u>

停下来回头看看上面那句话，执行之后如果你去看 myre 对象，会发现它上面只有一个我们自己加的 type 属性，而并没有正则表达式的其它属性，**可是我们的 RegExp.call(this, pattern, flags) 这个调用不是希望把这个this“构造”为一个正则表达式对象吗？再仔细想想，这句话真的应该能把空的this对象“构造”为一个正则表达式对象吗？**

首先，这个 this 是我们通过 new MyRegExp 调用而创建出来的，它是一个以 MyRegExp.prototype 为原型的普通对象，MyRegExp 刚开始运行时它还是空的全新对象，一个属性都没有。我们这里是想要把一个普通的 this 对象变为一个正则表达式对象。

## 外来对象与朴素对象

> 可是我们真的可以这么做吗？

我们先来思考另一个问题，函数在 JS 中也是对象，我们能把一个普通对象(如var a = {})变为一个函数吗？我的意思是说，在不改变 a 的指向前提下，将a指向的这个对象变成一个函数。

这个问题稍微写过 JS 的人都知道是不能的。 我们甚至不能把一个对象转换为一个数组，反过来也不行（注意转的像是不行的，要完全转成数组）。我们**更**不能把一个**普通对象**转换为一个**正则表达式对象**，为什么说“更”呢？

JS是一门典型的宿主语言，语言中的很多基本类型是由宿主环境去实现的，但是通过JS对象或函数的形式暴露出来。比如说在浏览器中，Date，Array，Document，HTMLElement，window，等对象都是浏览器内部实现的，在标准中专门有一个术语来描述这种对象，叫外来对象（Exotic Object），而我们把一个 {} 称为朴素对象。而正则表达式正是这种外来对象，我们能够在 JS 中使用它，只是因为浏览器将它内部的正则通过 JS 暴露给了我们。回到我们之前的代码：

```js
function MyRegExp(pattern, flags) {
  RegExp.call(this, pattern, flags)

  this.type = "MyRegExp"
}
```

RegExp.call(this, pattern, flags) 这一行代码企图把 this 构造为一个正则对象，然而这个 this 是在我们 new MyRegExp() 时创建出来的，它就是一个最最普通不过的空的朴素对象（就是一个{}）。

实际上引擎考虑到了这个问题，由于这种转换行不通，当我们这样写的时候，这些原生的构造函数都会忽略我们传给它的对象，即 RegExp.call(this, pattern, flags) 运行时，RegExp 构造函数根本不会去考虑这个 this，**而是会返回一个新构造出来的对象**。

几乎所有外来对象的构造函数都是这种行为，类似地，你也不能通过 Array.call({}) 将传入的空对象转换为一个数组，实际上它返回了一个新的数组。所以如果我们真想要实现对一些自带类型（即外来对象）的继承，我们需要这么写：

```js
function MyRegExp(pattern, flags) {
  var obj = RegExp.call(this, pattern, flags)//先拿到一个父类的对象

  obj.type = "MyRegExp"//“加工”以变成我们想要的样式，即变为了子类的样子
  return obj//将其返回，否则构造函数返回返回this，而这里面我们并没有碰过this，它还是个空对象
}
```

毕竟，在面向对象的继承中，一般逻辑是我们需要先得到一个父类的实例，再对这个实例进行“加工”以得到子类的实例。

这里既然 this 无法被转换为一个正则的实例，我们只好通过父类的构造函数构造一个出来，然后在它上面进行“加工”。

在这种写法下，绑定到 this 的空对象我们是不使用的，以所我们在子类的构造函数中返回了 obj 对象。**现在你明白为什么构造函数可以返回另一个对象而不是 this 了吧**。如果不返回，我们的构造函数就会返回 this 绑定的对象，而这个对象无法被父类的构造函数转换成父类的实例。

同时由于父类构造函数根本就不会管我们传入的 this，所以这一行也可以直接写为：

```js
  var obj = new RegExp(pattern, flags)
```

并在最后把 obj 返回；并且因为这个 obj 是直接由父类的构造函数返回的，其并不继承自我们的构造函数原型，所以为了它能够访问到我们原型上的方法，还需要重置一下它的原型：

```js
function MyRegExp(pattern, flags) {
  var obj = new RegExp.call(pattern, flags)
  obj.__proto__ = MyRegExp.prototype//重置父类实例的原型，让其为子类构造函数的prototype

  obj.type = "MyRegExp"

  return obj
}

Object.setPrototypeOf(MyRegExp.prototype, RegExp.prototype)
```

这样一来，MyRegExp 实际上返回了一个 RegExp 的实例，但它以MyRegExp.prototype 为原型，而 MyRegExp.prototype 又以RegExp.prototype 为原型，它就可以访问到我们设置在MyRegExp.prototype 上的方法，也可以访问到继承自 RegExp.prototype 的方法。

其实构造函数第一句的 obj 就是原来的 this，而在这一行之前由于 obj 将要指向的对象还没返回，所以当然就不能使用它了。对应到 class 语法中，就是 super() 调用之后，this 才会绑定到父类的实例（可以认为这个实例由 super() 返回）上，而在调用之前，this 还无对象可绑，自然也就不能使用了。