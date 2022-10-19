## Class

### class源码解析：

```ts
class Father {
  public name: string
  public age: number
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }
  public say() {
    console.log('I am father')
  }
}

class Son extends Father {
  public height: number
  constructor(name: string, age: number, height: number) {
    super(name, age)
    this.height = height
  }
  public say() {
    console.log('I am son')
  }
}

const child = new Son('zhangsan', 18, 180)
child.say()
```

```js
'use strict'
// 继承方法
var __extends =
  (this && this.__extends) ||
  (function () {
    // 定义继承静态方法的函数
    var extendStatics = function (d, b) {
      // 由于函数也是对象，因此函数也有原型对象，因此可以通过函数的原型对象来实现继承
      // 也可以通过传统的方式来手动把父类上的静态属性和方法复制到子类上
      extendStatics =
        Object.setPrototypeOf /* 判断有没有setPrototypeOf方法 */ ||
        ({ __proto__: [] } instanceof Array &&
          function (d, b) {
            d.__proto__ = b /* 如果没有则使用—__proto__定义 */
          }) ||
        function (d, b) {
          /* 如果还没有则直接使用通过循环手动赋值 */
          for (var p in b)
            if (Object.prototype.hasOwnProperty.call(b, p)) d[p] = b[p]
        }
      return extendStatics(d, b)
    }
    // 返回继承方法
    return function (d, b) {
      // 如果父类不是一个函数则抛出错误
      if (typeof b !== 'function' && b !== null)
        throw new TypeError(
          'Class extends value ' + String(b) + ' is not a constructor or null'
        )
      // 继承静态属性和方法
      extendStatics(d, b)
      function __() {
        this.constructor = d
      }
      // 继承原型属性和方法
      d.prototype =
        b === null ? Object.create(b) : ((__.prototype = b.prototype), new __())
    }
  })()

var Father = /** @class */ (function () {
  function Father(name, age) {
    this.name = name
    this.age = age
  }
  Father.prototype.say = function () {
    console.log('I am father')
  }
  return Father
})()

var Son = /** @class */ (function (_super) {
  __extends(Son, _super)
  function Son(name, age, height) {
    var _this = _super.call(this, name, age) || this
    _this.height = height
    return _this
  }
  Son.prototype.say = function () {
    console.log('I am son')
  }
  return Son
})(Father)

var child = new Son('zhangsan', 18, 180)
child.say()
```

### new操作符的原理：

> new 一个实例对象的底层3步

```js
// 1.创建一个 Object 对象
var obj = {}

// 2.让新创建的对象的 __proto__ 变量指向 Person 原型对象空间
obj.__proto__ = Person.prototype;
// Object.setPrototypeOf(obj, Person.prototype)

// 3.借用Person类的构造函数来给对象赋值
Person.apply(obj, ["12344", 23]);
```

### 类的引用属性：

```js
import OrderDetail from './OrderDetail'

class Order {
  constructor(
    public orderId: number, 
    public date: Date,
    public custname: string,
    public phone: string,
    public orderDetailArray: Array<OrderDetail> // 引用属性
    ) {
    return this;
  }
    
  doEat() {
    const x = 3;
    x = 10;
  }
}

let orderDetailOne = new OrderDetail(10, "电视机", 5000, 3);
let orderDetailTwo = new OrderDetail(11, "桌子", 2000, 2);
var orderDate = new Date(2023, 10, 17, 5, 20, 0);
let order = new Order(1, orderDate, "李武", "33333",
  [orderDetailOne, orderDetailTwo]);

console.log(order);
```

```js
export default class OrderDetail {
  public orderDetailId: number
  public productname: string 
  public price: number
  public count: number

  constructor(
      orderDetailId_: number, 
      productname_: string,
      count_: number, 
      price_: number
  ) {
    this.orderDetailId = orderDetailId_;
    this.productname = productname_;
    this.price = price_;
    this.count = count_
  }

  public getTotal(): number {
    return this.price * this.count
  }
}
```

### 方法重写：

> **条件**：一定发生在继承的子类中
>
> **位置**： 子类中重写父类的方法
>
> **应用场景**：当父类中方法的实现不能满足子类功能需要或不能完全满足子类功能需要时，就需要在子类中进行重写
>
> **方法重写给继承带来的好处**: 让所有的子类共用父类中方法已经实现了一部分功能的代码【父类方法代码在各个子类中得到了复用】 
>
> **定义规则**：
>
> 1. 和父类方法同名  
> 2. 参数和父类相同，如果是引用类型的参数，需要依据具体类型来定义
>
> 3. 父类方法的访问范围【访问修饰符】必须小于子类中方法重写的访问范围【访问修饰符】，同时父类方法不能是private 

```js
class Lady {
    content='hi,body'
    sayHi(){
        return this.content
    }
}

class beautifulLady extends Lady{
    sayHi(){
        return '你好！'
    }
    sayLove(){
        return this.content+'l love you'
    }
}

const b = new beautifulLady() 
console.log(b.sayHi()); 
console.log(b.sayLove()); 
```

### 重载

#### 函数重载的优势：

**优势1： 结构分明**

让 代码可读性，可维护性提升许多，而且代码更漂亮。

**优势2： 各司其职，自动提示方法和属性：**

每个重载签名函数完成各自功能，输出取值时不用强制转换就能出现自动提示，从而提高开发效率

**优势3： **

更利于功能扩展

#### 函数重载的定义：

> TS 的函数重载比较特殊，和很多其他后端语言的方法重载相比，多了不少规则。学习函数重载，先要了解什么是函数签名。
>
> 函数签名=函数名称+函数参数+函数参数类型+返回值类型四者合成。在 TS 函数重载中，包含了实现签名和重载签名，实现签名是一种函数签名，重载签名也是一种函数签名。

**定义：**包含了以下规则的一组函数就是TS函数重载：

- **规则1：**由一个实现签名+ 一个或多个重载签名合成。

- **规则2：** 但外部调用函数重载定义的函数时，只能调用重载签名，不能调用实现签名，这看似矛盾的规则，其实 是TS 的规定：实现签名下的函数体是给重载签名编写的，实现签名只是在定义时起到了统领所有重载签名的作用，在执行调用时就看不到实现签名了。

- **规则3：**调用重载函数时，会根据传递的参数来判断你调用的是哪一个函数 

- **规则4:**  只有一个函数体，只有实现签名配备了函数体，所有的重载签名都只有签名，没有配备函数体。

- **规则5:  关于参数类型规则完整总结如下：**

  实现签名参数个数可以少于重载签名的参数个数，但实现签名如果准备包含重载签名的某个位置的参数 ，那实现签名就必须兼容所有重载签名该位置的参数类型【联合类型或 any 或 unknown 类型的一种】。

- **规则6： 关于重载签名和实现签名的返回值类型规则完整总结如下：**

  必须给重载签名提供返回值类型，TS 无法默认推导。

  提供给重载签名的返回值类型不一定为其执行时的真实返回值类型，可以为重载签名提供真实返回值类型，也可以提供  void 或 unknown 或 any 类型，如果重载签名的返回值类型是 void 或 unknown 或 any 类型，那么将由实现签名来决定重载签名执行时的真实返回值类型。 当然为了调用时能有自动提示+可读性更好+避免可能出现了类型强制转换，强烈建议为重载签名提供真实返回值类型。

  不管重载签名返回值类型是何种类型【包括后面讲的泛型类型】，实现签名都可以返回 any 类型 或 unknown类型，当然一般我们两者都不选择，让 TS 默认为实现签名自动推导返回值类型。

#### 函数重载示例：

##### 搜索微信消息：

```js
type MessageType = "image" | "audio" | string //微信消息类型

type Message = {
  id: number;
  type: MessageType;
  sendmessage: string;
}

let messages: Message[] = [
  {
    id: 1, type: 'image', sendmessage: "你好啊,今晚咱们一起去三里屯吧",
  },
  {
    id: 2, type: 'audio', sendmessage: "朝辞白帝彩云间，千里江陵一日还"
  },
  {
    id: 3, type: 'audio', sendmessage: "你好！张无忌"
  },
  {
    id: 4, type: 'image', sendmessage: "刘老根苦练舞台绝技！"
  },
  {
    id: 5, type: 'image', sendmessage: "今晚王牌对王牌节目咋样?"
  }
]

function getMessage(value: number, myname: string): Message
function getMessage(value: MessageType, readRecordCount: number): Message[]
function getMessage(value: number | MessageType, readRecordCount: number = 1): Message | undefined | Message[] {
  if (typeof value === "number") {
    return messages.find((msg) => { return 6 === msg.id }) // 有可能是undefined
  } else {
    return messages.filter((msg) => value === msg.type).splice(0, readRecordCount)
  }
}
getMessage(1, "df")
getMessage("image", 2).forEach((msg) => {
  console.log(msg)
})
```

上面的案例中，有一个函数叫getMessage，它有两个重载签名，一个实现签名，第一个重载签名表示用id去查找微信消息，第二个重载签名表示用消息类型去查找微信消息。

在第一条重载签名中，有两个参数，value和myname，myname是没有在实现签名中体现的，这就印证了上面规则5中关于参数类型规则的总结：**实现签名参数个数可以少于重载签名的参数个数**。

并且在第一条重载签名中有一个坑，可以发现它的参数中并没有readRecordCount，并在函数体中确实是使用了readRecordCount，这就会导致错误，ts认为这是不安全的，要解决也很简单，只需要在实现签名中给readRecordCount加上一个默认值就可以了，这是因为实现签名在找不到某个参数时，会去找实现签名要，这里如果在实现签名中该参数正好有一个默认值的话就不会报错了。（注意不能把默认值写在重载签名中，因为它只负责重载工作，不负责真正接收参数值）

在实现签名中没有写返回值类型，这里因为ts会自动根据实际的返回值进行推导，不必多此一举。但是选项签名的返回值类型是必须的，因为ts无法进行推导。

#### 方法重载

##### 封装数组，优化增删改：

```js
class ArrayList {
  constructor(public element: Array<object>) { }
    
  get(index: number) {
    return this.element[index]
  }

  show() {
    this.element.forEach((ele) => {
      console.log(ele);
    })
  }

  remove(value: number): number
  remove(value: object): object
  remove(value: number | object): number | object { 
    this.element = this.element.filter((ele, index) => {
      if (typeof value === "number") {
        //如果是根据数字【元素索引】去删除元素，remove方法返回的是一个数字
        return value !== index
      } else {
        // 如果是根据对象去删除元素，remove方法返回的是一个对象
        return value !== ele
      }
    })
    return value;
  }

}

let stuOne = { stuname: "wnagwu", age: 23 }
let stuTwo = { stuname: "lisi", age: 39 }
let stuThree = { stuname: "liuqi", age: 31 }

let arrayList = new ArrayList([stuOne, stuTwo, stuThree]);
arrayList.show();

let value = arrayList.remove(stuTwo)
console.log("删除的学生对象为:", value)
arrayList.show();
```

#### 构造器重载

**this问题：**

this 其实是一个对象变量，当 new 出来一个对象时，构造器会隐式返回 this  给 new 对象等号左边的对象变量，this 和等号左边的对象变量都指向当前正创建的对象。

以后，哪一个对象调用 TS 类的方法，那么这个方法中的 this 都指向当前正使用的对象【 this 和当前的对象变量中都保存着当前对象的首地址】

> 要区分对象变量和对象，对象变量指的是存储在栈空间中对对象的引用（地址），而对象是指在堆空间中实际的对象体。

**构造器的返回值**

尽管TS类构造器会隐式返回 this，如果我们非要返回一个值，TS 类构造器只允许返回 this，但构造器不需要返回值也能通过编译，更没有返回值类型之说，从这个意义上，TS 构造器可以说成是没有返回值这一说的构造函数。

**构造器重载的意义**

构造器重载和函数重载使基本相同，主要区别是：TS 类构造器重载签名和实现签名都不需要管理返回值，TS 构造器是在对象创建出来之后，但是还没有赋值给对象变量之前被执行，一般用来给对象属性赋值。

在 TS 类中只能定义一个构造器，但实际应用时，TS 类在创建对象时经常需要用到有多个构造器的场景，比如：我们计算一个正方形面积，创建正方形对象，可以给构造器传递宽和高，也可以给构造器传递一个包含了宽和高的形状参数对象，这样需要用构造器重载来解决。

**构造器是方法吗?**

我们说对象调用的才是方法，但是 TS 构造器是在对象空间地址赋值给对象变量之前被调用，而不是用来被对象变量调用的，所以构造器( constructor )可以说成构造函数，但不能被看成是一个方法。

##### 示例：

> 在第二个构造器重载签名中，没有_height参数，但是在函数体中使用了，这会导致ts编译报错，解决方法就是给它加上这个属性，但是这个函数重载中根本不需要这个参数，这违背了函数重载的意义，因此这个方法不可用。
>
> 要解决它，最简单的方法就是在构造器的实现签名中，给_height加上一个默认参数，因为重载签名中找不到的参数就会去实现签名中找默认值，这样就不会报错了。

```js
type ChartParams = {
  width: number
  height: number
}

class Square {
  public width: number
  public height: number
  constructor(_width: number, _height: number)
  constructor(_objParamOrWidth: ChartParams)
  constructor(_objParamOrWidth: ChartParams | number, _height = 0) {
    if (typeof _objParamOrWidth === 'number') {
      this.width = _objParamOrWidth
      this.height = _height
    } else {
      this.width = _objParamOrWidth.width
      this.height = _objParamOrWidth.height
    }
  }
}

export default Square
```

### 类中属性和方法的修饰符：

|                  |  public  | protected  |  private   | 备注                                           |
| :--------------: | :------: | :--------: | :--------: | ---------------------------------------------- |
|    实例化对象    | 可以调用 |  不可调用  |  不可调用  | public是对外暴露的                             |
|    在当前类中    | 可以调用 |  可以调用  |  可以调用  |                                                |
|   在继承子类中   | 可以调用 |  可以调用  |  不可调用  | 私有属性和方法不能在子类中调用                 |
| override父类方法 | 可以重写 |  可以重写  | 不可以重写 | 受保护的表示只能在当前类和子类中进行操作       |
|    接口继承类    | 可以继承 | 不可以继承 | 不可以继承 | 只有全部都是public属性方法的类才可以被接口继承 |

- 优先设定类中的属性和方法都是private（私有的）

- 当需要继承时，并且需要通过继承后进行调用和覆盖方法时，再将它们改为protected（受保护的）

- 当需要将类中属性和方法暴露在外，实例化对象可以调用的，修改为public（公有的）

``` js
class p {
  private age: number = 0
  public name: string = 'person'
}

interface A extends p {
  good(x: number): string
}

const obj: A = {
  age: 0,
  name: 'ilmango',
  good(x: number): string {
    return 'a'
  }
}
```

**上面的例子会报错：**

error: 不能将类型“{ age: number; name: string; good(x: number): string; }”分配给类型“A”。

属性“age”在类型“A”中是私有属性，但在类型“{ age: number; name: string; good(x: number): string; }”中不是。

**如果删除掉对象中的age属性，则又会报错**：

类型 "{ name: string; good(x: number): string; }" 中缺少属性 "age"，但类型 "A" 中需要该属性。

> 因此只有全部都是public属性方法的类才可以被接口继承。

### 抽象类：

> 一个在任何位置都不能被实例化的类就是一个抽象类【abstract class 】，那为什么要使用抽象类？
>
> 1. **提供统一名称的抽象方法，提高代码的可维护性**：
>
>    抽象类通常用来充当父类，当抽象类把一个方法定义为抽象方法，那么会强制在所有子类中实现它，防止不同子类的同功能的方法命名不相同，从而降低项目维护成本。
>
> 2. **防止实例化一个实例化后毫无意义的类。**

**什么样的类可以被定义为抽象类**

从宏观上来说，任何一个实例化后毫无意义的类都可以被定义为抽象类。 比如：我们实例化一个玫瑰花类的对象变量，可以得到一个具体的 玫瑰花 实例对象，但如果我们实例化一个  Flower  类的对象变量，那世界上有一个叫 花 的对象吗？很明显没有，所以 Flower 类 可以定义为一个抽象类，但玫瑰花可以定义为具体的类。

**抽象类的特征**

> 单纯从类的特征上来看和普通类没有区别，只是多了可以有 0 到多个抽象方法这一条，并且不能被实例化：
>
> - 抽象类可以包含只有方法声明的方法【 和方法签名类似，就是多了 abstract 关键字】，也就是抽象方法，继承该抽象类的子类必须实现所有的抽象方法。
>
> - 抽象类也可以包含实现了具体功能的方法，可以包含构造器，但不能直接实例化一个抽象类，只能通过子类来实例化。

```js
abstract class 类名 {   
    一个构造器
    零到到多个抽象方法【只有方法体，没有方法实现的方法】
    零到到多个具体方法
    零到到多个实例属性
    零到到多个静态属性
    零到到多个静态方法
} 
```

**抽象类的应用：适配器**

> 适配器模式（Adapter Pattern）：结构型模式之一，将一个类的接口转换成客户希望的另一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的哪些类可以一起工作。
>
> MouseListenerProcessAdapter类就是一个适配器，它实现了MouseListenerProcess，但是只把一些不常用的方法进行了简单的实现，一些高频方法被定义为了抽象类，这样子类在继承他时，就必须实现这些高频方法。

```js
interface MouseListenerProcess {
  mouseReleased(e: any): void//  鼠标按钮在组件上释放时调用。
  mousePressed(e: any): void//  鼠标按键在组件上按下时调用。
  mouseEntered(e: any): void //鼠标进入到组件上时调用。

  mouseClicked(e: any): void// 鼠标按键在组件上单击（按下并释放）时调用。
  mouseExited(e: any): void//  鼠标离开组件时调用。
}
  
// 适配器Adapter是一个抽象类
abstract class MouseListenerProcessAdapter implements MouseListenerProcess {
  mouseReleased(e: any): void {
    throw new Error('Method not implemented.');
  }
  mousePressed(e: any): void {
    throw new Error('Method not implemented.');
  }
  mouseEntered(e: any): void {
    throw new Error('Method not implemented.');
  }
  abstract mouseClicked(e: any): void;

  abstract mouseExited(e: any): void;
}

class MyMouseListenerProcess extends MouseListenerProcessAdapter {
  mouseClicked(e: any): void {
    throw new Error('Method not implemented.');
  }
  mouseExited(e: any): void {
    throw new Error('Method not implemented.');
  }
}
```

