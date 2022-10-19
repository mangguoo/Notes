1. if表达式运算后隐式转换成布尔值，以下六种情况会转换为false：

   - undefined
   - null
   - ""
   - false
   - 0
   - NaN

2. if使用实例：

   ``` js
   let a = 0
   let b
   
   if (a++) {
     b = 3
   }
   <==  等价  ==>
   if (a === 0) {
     a++
   } else {
     b = 3
     a++
   }
   
   console.log(a, b) // undefined
   ```

3. while循环实现1~100的累加：

   ``` js
   var i=0,s=0;
   while(s+=i,++i<=100);
   console.log(s);
   // 这里的while循环条件表达式中用了一个逗号表达式，逗号表达式会返回
   // 最后一个表达式的值，因此最后一个表达式会决定while循环是否继续
   ```

4. while循环实现打印一百内的素数：

   ```js
   let i = 1
   /* eslint-disable no-labels */
   start: while (i++ < 100) {
     let j = 2
     while (j <= parseInt(i / 2)) {
       if (i % j === 0) continue start
       j++
     }
     console.log(i)
   }
   ```

   