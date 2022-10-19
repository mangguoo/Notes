#### 字面量对象：

1. 传入对象的key会隐式转换为字符串：

``` js
var a=true;
var obj={}
obj[a]=2;

console.log(obj.true); // 2

var o={a:1};
var o1={a:2};
var obj={};
obj[o]=10;
console.log(obj[o1]); // 10

var obj={a:1};
var arr=["a"];
console.log(obj[arr]); // 1
```

2. **`for...in`语句**以任意顺序迭代一个对象的除Symbol以外的可枚举属性，包括继承的可枚举属性。

> 使用forin循环时，Key会被转换为字符串，并且forin遍历数组时不会遍历空元素，遍历对象时不会遍历不可枚举属性：

``` js
const arr = [1, 2, 3, , 4, undefined, 5]

console.log(arr[3]) // undefined

for (const key in arr) {
  console.log(key, typeof key)
}
// 0 string
// 1 string
// 2 string
// 4 string
// 5 string
// 6 string

console.log(3 in arr) // false
console.log(5 in arr) // true
// 只要元素不为空，in运算符则返回true
```

3. 对象属性上的this指向当前对象上下文环境中的this：

``` js
var a = 10
const obj = {
  a: 1,
  b: this.a
}
console.log(obj.b) // 10

var a = 100
const o = {
  a: 1,
  b: function () {
    const o1 = {
      a: 10,
      c: this.a // 这里的this指向对象o
    }
    console.log(o1.c)
  }
}
o.b() // 1
```

4. null用来在垃圾回收时。可以将不使用的引用类型回收掉：

``` js
var o={a:1,b:2};
o=null;
```

#### Object的静态方法：

1. ###### 使用Object.assign()实现骚操作：

``` js
Object.assign(div.style, {
  width: '50px',
  height: '50px',
  backgroundColor: 'red',
  borderRadius: '50px'
} as CSSStyleDeclaration)
```

- **`Object.assign()`** 方法将所有[可枚举](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable)（`Object.propertyIsEnumerable()` 返回 true）和[自有](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)（`Object.hasOwnProperty()` 返回 true）属性从一个或多个源对象复制到目标对象，返回修改后的对象。(浅复制)
- `as`是ts的关键字，用来进行类型断言，断言就相当于告诉编译器这个对象是什么类型，这样vscode就会进行代码提示了。

2. ###### Object.entries和Object.fromEntries：

``` js
// 将对象转换为迭代器
var o=Object.entries({a:1,b:2});
// [["a",1],["b",2]]

// 将迭代器转换为对象，可以将map类型转换为对象类型
var o=Object.fromEntries([["a",1],["b",2]]);
console.log(o)
// {a:1,b:2}

var m=new Map();
m.set("a",1);
m.set("b",2);
console.log(Object.fromEntries(m))
// {a:1,b:2}
```

3. ###### 获取对象属性名：

- keys方法会获取传入对象的可遍历属性；
  
- getOwnPropertyNames方法会获取传入对象的所有属性(包括不可枚举，但不包换symbol)；
  
- getOwnPropertySymbols方法会获取传入对象的所有symbol属性；
  
``` js
// 三种获取对象自身所有属性的方法(包括不可遍历和symbol)
Object.getOwnPropertyNames(obj).concat(Object.getOwnPropertySymbols(obj))
[...Object.getOwnPropertyNames(obj),...Object.getOwnPropertySymbols(obj)]
Reflect.ownKeys(obj)
```

4. ###### Object.is()：

> `Object.is()` 方法判断两个值是否为同一个值，如果满足以下任意条件则两个值相等：

- 都是 `undefined`
- 都是 `null`
- 都是 `true` 或都是 `false`
- 都是相同长度、相同字符、按相同顺序排列的字符串
- 都是相同对象（意味着都是同一个对象的值引用）
- 都是数字且
  - 都是 `+0`
  - 都是 `-0`
  - 都是 `NaN`
  - 都是同一个值，非零且都不是 `NaN`
  

`Object.is()` 与 `==`不同。`==` 运算符在判断相等前对两边的变量（如果它们不是同一类型）进行强制转换（这种行为将 `"" == false` 判断为 `true`），而 `Object.is` 不会强制转换两边的值。

#### Object的实例方法：

1. ###### Object.prototype.propertyIsEnumerable()：

> `propertyIsEnumerable(key)` 方法返回一个布尔值，表示指定的属性是否可枚举。

2. ###### Object.prototype.hasOwnProperty()：

```
 > `hasOwnProperty(key)` 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键）。
```

#### getter和setter：

> set 和 get设置的属性名必须相同
>
> set和get并不能直接存储值，需要借助一个第三个临时变量来存储这个值，并且这个临时变量尽量不暴露出现
>
> 如果只设置set，不设置get 则为只写，不可读，如果只设置get，不设置set 则为只读 ，不可写

通过getter和setter做数据驱动显示：

``` js
const div = document.querySelector('div')
Object.defineProperties(div, {
  _x: {
    writable: true,
    value: 0
  },
  _y: {
    writable: true,
    value: 0
  },
  x: {
    enumerable: true,
    set(value) {
      value = ~~value
      this._x = value
      this.style.left = value + 'px'
    },
    get() {
      return this._x
    }
  },
  y: {
    enumerable: true,
    set(value) {
      value = ~~value
      this._y = value
      this.style.top = value + 'px'
    },
    get() {
      return this._y
    }
  }
})

setInterval(function () {
  div.x++
  div.y++
}, 16)
```

