1. charCodeAt()：获取指定下标的编码

   ``` js
   "a".charCodeAt(0) // 97
   ```

2. Array.fromCharCode()：查询指定编码对应的字符

   ``` ks
   String.fromCharCode(97) // "a"
   
   Array(26).fill(1).map(function(val, index) => {
   	return String.fromCharCode(i+97)
   }).join("")
   ```

3. substring、slice、 substr：

   ``` js
   str.substring(start, end)
   // 不能使用负数，使用负数表示0之前的位置
   // 但是它有一个特殊的地方，就是可以start可以大于end，实现从右向左获取
   
   str.slice(start, end)
   // 可以使用负数
   
   str.substr(start, count)
   ```

4. repeat、padStart、padEnd：

   ``` js
   var str = 'aa'
   str.repeat(3)
   
   str.padStart(8, 0) // 字符串长度不够8则在前面补0
   ```

5. startWith、endWith：

   ``` js
   str.startWith("a") // 从第一位开始起始是不是a
   str.startWith("b", 1) // 从哪一位开始算起始
   
   str.endWith("b") //是不是以b结尾
   str.endWith("b", 5) // 第五位之前是不是以b结尾
   ```

6. parseQuery:

   ``` js
   const url = 'https://www.163.com/news/index.html?a=1&b=2&c=3'
   function getQuery(url) {
     return url
       .split('?')[1]
       .split('&')
       .reduce(function (v, t) {
         const arr = t.split('=')
         v[arr[0]] = isNaN(arr[1]) ? arr[1] : Number(arr[1])
         return v
       }, {})
   }
   getQuery(url)
   ```

   