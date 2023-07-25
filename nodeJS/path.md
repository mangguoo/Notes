# path

>  路径主要是为了解决本地路径的拼接所需要的一个方法集

## path.join

方法使用平台特定的分隔符作为定界符将所有给定的 path 片段连接在一起，然后规范化生成的路径

``` js
path.join("c:/ab/cd","c.txt")
path.join("c:/ab/cd","../c.txt")
path.join("c:/ab/cd","../../c.txt")
```

## path.basename

返回路径中最后的部分,可以去掉扩展名

``` js
var str=path.join('/foo', 'bar', 'baz/asdf.txt')
path.basename(str)
path.basename(str,".txt")
```

## path.dirname

返回路径中目录名

``` js
var str=path.join('/foo', 'bar', 'baz/asdf.txt')
path.dirname(str)
```

## path.extname

返回路径中的扩展名

```js
var str=path.join('/foo', 'bar', 'baz/asdf.txt');
console.log(path.extname(str));
```

## path.parse

将字符串对象转换为对象

``` js
var str=path.join('/foo', 'bar', 'baz/asdf.txt');
console.log(path.parse(str));
```

## path.format

将path对象转换为字符串

``` js
var str=path.join('/foo', 'bar', 'baz/asdf.txt');
var obj=path.parse(str);
console.log(path.format(obj))
```

## `__filename` 与 `__dirname`

分别表示当前文件路径，与当前文件夹路径

``` js
console.log(__dirname, __filename)
path.join(path.join(__dirname, 'a.js')) // 当前目录下的a.js
```

## path.resolve

1、给定的路径序列会**从右到左**进行处理，后面的每个 `path` 会被追加到前面，**直到构造出绝对路径**；

2、如果在处理完所有给定的 `path` 片段之后**还未生成绝对路径**，则会在前面添加当前**工作目录**；

3、生成的路径会被规范化，并且**尾部的斜杠会被删除**（除非路径被解析为根目录）；

4、如果没有传入 `path` 片段，则 **`path.resolve()` 会返回当前工作目录的绝对路径**；

``` js
path.resolve() // 返回当前路径
path.resolve("a.js") // 返回当前路径下的a.js
path.resolve("../a.js") // 返回当前路径的上一层路径下的a.js
```

