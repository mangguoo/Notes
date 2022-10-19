1. 数组下标如果可以隐式转换成数值时，自动作为下标数字，如果隐匿转换后是非数值时，则作为对象key插入属性：

   ``` js
   var arr = []
   arr["1"] = 1
   console.log(arr.length)  // 2
   arr["a"] = 1
   console.log(arr.length)  // 2
   ```

2. 使用删除对象元素的方式删除数组元素：

   ``` js
   var arr = [1,2,3,4,5]
   delete arr[2]
   console.log(arr) 
   // [1,2,,4,5]
   // 这种删除方式不改变数组的长度，将对应的值设置为空元素，这样可以防止数组塌陷
   // 使用这种方式就可以很方便的清除数组中的相同元素，而不需要考虑数组塌陷，然后再使用forin来过滤空元素
   ```

3. concat具备复制数组兼添加新元素的功能：

   ``` js
   var arr = [1,2,3]
   var arr1 = arr.concat(4)
   arr[0] = 10
   console.log(arr1)
   // [1,2,3,4]
   ```

4. 使用every实现全选选项：

   ``` js
   <form action="#">
     <input type="checkbox" id="all" />全选<br />
     <input type="checkbox" /><br />
     <input type="checkbox" /><br />
     <input type="checkbox" /><br />
     <input type="checkbox" /><br />
     <input type="checkbox" /><br />
     <input type="checkbox" /><br />
   </form>
   
   <script>
     var inputs
   
     init()
     function init() {
       inputs = [].concat.apply([], document.getElementsByTagName('input'))
       for (var i = 0; i < inputs.length; i++) {
         inputs[i].onclick = clickHandler
       }
     }
   
     function clickHandler() {
       if (this.id === 'all') {
         inputs.forEach(function (item) {
           item.checked = this.checked
         }, this)
       } else {
         inputs[0].checked = inputs.slice(1).every(function (item) {
           return item.checked
         })
       }
     }
   </script>
   ```

5. 利用slice将类数组元素转换成数组：

   ``` js
   function fn() {
     // 将非数组的列表转换为数组
     const arr = Array.prototype.slice.call(arguments)
     console.log(arr)
   }
   
   fn(1, 2, 3, 4)
   ```

6. 数组查找方法的选用：

   ``` txt
   当在数组中查找时
   如果需要判断有没有这个数值或者字符或者布尔值这种非引用地址的元素时 使用includes
   如果需要获取这个数值或者字符或者布尔值这种非引用地址元素的下标时  使用indexOf
   如果需要从后向前获取这个数值或者字符或者布尔值这种非引用地址元素的下标时  使用lastIndexOf
   如果需要获取数组中某个引用元素是否满足某个条件的元素，使用find
   如果需要获取数组中某个引用元素是否满足某个条件的元素下标，使用findIndex
   如果需要从后向前获取数组中某个引用元素是否满足某个条件的元素，使用findLast
   如果需要从后向前获取数组中某个引用元素是否满足某个条件的元素下标，使用findLastIndex
   如果需要获取到数组中所有满足条件的元素时，使用筛选 filter
   ```

7. includes的特殊用法：

   ``` js
   var a="a";
   if(a==="a" || a==="b" || a==="c" || a==="d")
       <==等价于==>
   if(["a","b","c","d"].includes(a))
   ```

8. indexOf的使用：

   ``` js
   if(~[1,2,3].indexOf(2)){
     // 查询成功
     // 查询失败会返回-1，而~-1=0
   }
   
   // 查询数组中所有2的下标
   var  arr=[1,2,3,1,2,3,4,1,2,3,4];
   var index=-1;
   while(~(index=arr.indexOf(2,index+1))){
       console.log(index);
   }
   ```

9. 使用reduce和reduceRight模拟其它数组方法：

   ``` js
   // 模拟map
   const arr = [1, 2, 3, 4, 5]
   const arr1 = arr.reduce(function (v, t) {
     v.push(t + 10)
     return v
   }, [])
   
   // 模拟filter
   const arr1 = arr.reduce(function (v, t) {
     if (t > 3) v.push(t)
     return v
   }, [])
   
   // 模拟some
   const bool = arr.reduce(function (v, t) {
     if (t > 3) v = true
     return v
   }, false)
   
   // 模拟find
   arr = [
     { id: 1001, name: '商品1', price: 3000 },
     { id: 1004, name: '商品2', price: 4000 },
     { id: 1003, name: '商品3', price: 5000 },
     { id: 1004, name: '商品4', price: 6000 },
     { id: 1005, name: '商品5', price: 7000 }
   ]
   const item = arr.reduce(function (v, t) {
     if (!v && t.id === 1004) v = t
     return v
   }, null)
   
   // 模拟findLast
   const item = arr.reduceRight(function (v, t) {
     if (!v && t.id === 1004) v = t
     return v
   }, null)
   
   // 模拟flat
   const arr = [0, -1, [1, 2, 3], [4, 5, [6, 7, [8, 9]]]]
   const arr1 = arr.reduce(function (pre, val) {
     ;(function f(val, pre) {
       if (!Array.isArray(val)) {
         pre.push(val)
       } else {
         for (let i = 0; i < val.length; i++) {
           if (Array.isArray(val[i])) return f(val[i], pre)
           else pre.push(val[i])
         }
       }
     })(val, pre)
     return pre
   }, [])
   ```

10. 用reduce来拼接字符串：

    ``` js
     obj = {
      user: 'xietian',
      password: 'xie123',
      sex: '男',
      age: 30
    }
    
    const str = Object.keys(obj).reduce(function (v, t) {
        return v + t + '=' + obj[t] + '&'
    }, '').slice(0, -1)
    
    const arr = [
      { site: '网易', url: 'http://www.163.com' },
      { site: '京东', url: 'http://www.jd.com' },
      { site: '淘宝', url: 'http://www.taobao.com' },
      { site: '腾讯', url: 'http://www.qq.com' },
      { site: '百度', url: 'http://www.baidu.com' },
      { site: '美团', url: 'http://www.meituan.com' }
    ]
    
    const ul = document.getElementById('ul')
    ul.innerHTML = arr.reduce(function (v, t) {
      return v + "<li><a href='" + t.url + "'>" + t.site + '</a></li>'
    }, '')
    ```

11. 打乱数组：

    ``` js
    // map不会遍历空元素，因此要通过fill方法来填充
    var arr = Array(100)
      .fill(1)
      .map(function (t, i) {
        return i + 1
      })
    
    // 通过sort随机打乱
    arr.sort(function () {
      return Math.random() - 0.5
    })
    ```

12. for...of可以遍历空元素：

    ``` js
    const arr = new Array(20)
    for (const val of arr) {
      console.log(1)
    }
    ```

13. 中断forEach循环：

    ``` js
    const arr = [1, 2, 3, 4, 5, 6, 7]
    
    try {
      arr.forEach(function (item) {
        console.log(item)
        if (item === 4) throw new Error('aaa')
      })
    } catch (err) {}
    ```

    

   