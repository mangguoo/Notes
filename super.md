#### super作为函数

super作为函数在子类的constructor中调用时，**代表的是父类的构造函数**。但是，虽然super代表的是父类的构造函数，但它内部的**this指向的是当前子类的构造出的实例**，**并且super作为函数只能用在子类的构造函数中**。

``` js
class A {
  constructor() {
    console.log(new.target.name, this)
  }
}
class B extends A {
  constructor() {
    super()
  }
}
new A() //A A {}
new B() //B B {}
```

#### super作为对象

- ##### super用在原型方法中

  1. super指向**父类的原型对象**

     ``` js
     class C {
       constructor() {}
     }
     
     // 静态属性
     C.cc = 7
     // 实例属性
     C.prototype.cc = 100
     
     class D extends C {
       constructor() {
         super()
         console.log(super.cc + ' and I am in D') //100
       }
     }
     ```

  2. 通过super调用父类方法时，**这些方法内部的this指向子类的实例**

     ```js 
     class C {
       constructor() {
         this.x = 11
       }
       fun() {
         this.x = 3
       }
       print() {
         console.log(this.x)
       }
     }
     
     class D extends C {
       constructor() {
         super()
         this.x = 90
       }
       f() {
         super.fun()
         this.x = 5
         super.print() //5
         // 如果此处没有this.x=5 则返回的是3，如果没有3，返回的是90，没有90，返回11
       }
     }
     ```

  3. 当通过super为子类属性**赋值**时，super就是this

     ``` js
     class C {
       constructor() {}
     }
     
     class D extends C {
       constructor() {
         super()
       }
       e() {
         super.e = 20 //这里的super是this
         console.log(super.e) // 这里的super指向的是父类的原型对象 undefined
         console.log(this.e) // 20
       }
     }
     ```

- ##### super用在静态方法中

  1. **super指向父类**（不是父类的原型对象）

     ``` js
     class E {
       constructor() {}
       static fun() {
         console.log('我是父类的fun')
       }
       fun() {
         console.log('我是父类原型对象的fun')
       }
     }
     
     class F extends E {
       constructor() {
         super()
       }
       static son() {
         super.fun() // 此时指向的是父类的fun  即静态fun
       }
       son() {
         super.fun() // 此时指向的是父类的原型对象  即普通fun
       }
     }
     let ff = new F()
     F.son() // 我是父类的fun
     ff.son() // 我是父类原型对象的fun
     ```

  2. 在子类的静态方法中通过super调用父类方法时，**这些方法内部的this指向子类**（不是子类的实例）

     ``` js
     class E {
       constructor() {
         this.x = 2
       }
       static print() {
         console.log(this.x)
       }
     }
     
     class F extends E {
       static x = 11
       constructor() {
         super()
         this.x = 4
       }
       static p() {
         this.x = 8
         // 如果此处没有this.x=8，则F.p()的执行会是11(执行顺序导致)
         // 如果F.x=11也没有，则输出undefined，不会输出构造函数中的2
         super.print()
       }
     }
     ```

     