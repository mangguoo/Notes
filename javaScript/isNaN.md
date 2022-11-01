# isNaN 和 Number.isNaN区别

> 1. isNaN()是ES5的方法，Number.isNaN()是ES6的方法
> 2. 可能有一些童鞋会认为 isNaN直译为“是不是 NaN”，其本意不是，**isNaN本意是通过Number方法把参数转换成数字类型，如若转换成功，则返回false，反之返回true，它只是判断参数是否能转成数字，不能用来判断是否严格等于NaN。**，如果要判断某个值是否严格等于NaN不能用这个方法
> 3. ES6提供了Number.isNaN方法用来判断一个值是否严格等于NaN，它会首先判断传入的值是否为数字类型，如不是，直接返回false。

**区别：**
isNaN方法首先转换类型，而Number.isNaN方法不用
isNaN不能用来判断是否严格等于NaN，Number.isNaN方法可用