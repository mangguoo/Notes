1. new Function构造函数的真正作用是可以接收字符串，这样就可以从服务器中接收函数体代码：

   ``` js
   var ef=new Function("a,b","return a+b")
   var ef=new Function("a","b","return a+b")
   ```

2. 预解析：

   ``` j s
   var a = 1
   function a() {
     console.log(a)
   }
   a() // a is not a function
   ```

   ``` js
   var a
   function a() {
     console.log(a)
   }
   a() // ƒ a() { console.log(a) }
   ```

   ``` js
   console.log(a) // function a() {}
   var a = 1
   function a() {}
   ```

   ``` js
   var a
   function a(a) {
     var a = 1
     console.log(a)
   }
   a(a) // 1
   ```

   > 在块级作用域中声明函数：
   >
   > 在块语句中声明不数不会提升至全局进行预解析包括If、for等语句块。
   >
   > 函数定义在块语句中与定义在全局中的行为是类似的，也就是说函数
   >
   > 会在块语句中进行提升，但是函数与let，const定义的变量不同，它
   >
   > 是没有块级作用域的，因此在块语句中定义的函数可以在全局中访问。 
   ``` js
   var a
   if (a === undefined) {
     function a() {
       console.log('aa')
     }
   }
   a() // 'aa'
   ```

   ``` js
   var a = 1
   if (a === 2) {
     function a() {
       console.log('aa')
     }
   }
   a() // TypeError: a is not a function
   ```

   ```js
   console.log(a)
   var a = 1
   {
     console.log(a)
     a = 2
     function a() {}
   }
   console.log(a)
   
   // undefined
   // ƒ a() {}
   // 2
   ```

   ``` js
   var a = 1
   {
     a = 2
     function a() {}
     a = 3
     function a() {}
     a = 4
     function a() {}
     a = 5
     console.log(a) //5
   }
   console.log(a) //4
   // 同名函数在块内预解析预赋值的同时将同名函数上的同名变量挤出块外赋值
   ```

   

3. 直接调用回调函数与使用arguments调用回调函数的区别：

   ``` js
   function fn(f) {
     arguments[0]() //this arguments
     f() // this window
   }
   function fn1() {
     console.log(this)
   }
   fn(fn1)
   ```

4. 循环回调：

   ``` js
   function fn1(f, a) {
     if (!a) a = 0
     a++
     console.log(a)
     f(fn1, a)
   }
   
   function fn2(f, a) {
     if (a < 10) f(fn2, a)
   }
   
   fn1(fn2)
   
   // 1
   // 2
   // 3
   // 4
   // 5
   // 6
   // 7
   // 8
   // 9
   // 10
   ```

   ```js
   function showRed(fn1, fn2) {
     var f = arguments.callee
     setTimeout(function () {
       console.log('红灯')
       fn1(fn2, f)
     }, 2000)
   }
   
   function showYellow(fn1, fn2) {
     var f = arguments.callee
     setTimeout(function () {
       console.log('黄灯')
       fn1(fn2, f)
     }, 1000)
   }
   function showGreen(fn1, fn2) {
     var f = arguments.callee
     setTimeout(function () {
       console.log('绿灯')
       fn1(fn2, f)
     }, 2000)
   }
   
   showRed(showYellow, showGreen)
   showGreen(showRed, showYellow)
   ```

   