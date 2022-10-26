### art-template

#### CSR时前端通过art渲染页面

##### art-template引入

``` js
npm i art-template -S

<script src="./node_modules/art-template/lib/template-web.js"></script>
```

##### 基础使用

``` js
<div id="app"></div>
<script id="tp" type="text/html">
    {{if user}}
    <h2>{{user.name}}</h2>
    <h2>{{user.age}}</h2>
    {{/if}}
</script>
<script>
    var user={name:"xietian",age:30};
    document.querySelector("#app").innerHTML=template("tp",{user:user});
</script>
```

##### 标准输出

```js
{{value}}
{{data.key}}
{{data['key']}}
{{a ? b : c}}
{{a || b}}
{{a + b}}
```

##### 原文输出

>  原文输出语句不会对 HTML 内容进行转义处理

```js
{{@ value }}
```

##### 条件

``` js
{{if value}} ... {{/if}}
{{if v1}} ... {{else if v2}} ... {{/if}}
```

```js
{{if a>3}}
  <li>{{@ c}}</li>
{{/if}}
```

##### 循环

```js
{{each target}}
    {{$index}} {{$value}}
{{/each}}
```

``` js
{{each target}}
    <li><a href={{$value.href}}>{{$value.site}}</a></li>
{{/each}}
```

- target 支持 array 与 object 的迭代，其默认值为 $data。
- \$value 与 $index 可以自定义：{{each target $val $key}}。

```js
{{each target $key $val}}
    <div>{{$key}}:{{$val}}</div>
{{/each}}
```

##### 变量

```js
{{set temp = data.sub.content}}
```

#### SSR时Node中express通过art渲染页面

##### 前置知识

- app.engine

  注册扩展名解析，如果遇到某种扩展名，使用某个方法解析该扩展名的文件

  ```js
  // 遇到art扩展名文件，使用express-art-template插件解析
  app.engine("art",require("express-art-template"));
  ```

- app.set

  部分第三方的一些插件在express框架中使用时需要设置一些基础参数，便于在使用时直接调用，这里有一部分是固定属性，也可以自定义属性

  有了这两个设置之后，在使用app.render时就会去默认的使用art作为视图解析引擎(渲染时可以不加扩展名)，并且把view文件夹当做默认寻找模板文件的路径

  ```js
  // 设置views(也就是模板文件夹)的调用路径是view文件夹
  app.set("views",path.join(__dirname,"./view"));
  // 设置视图解析的方式(app.render)是通过art这个注册的插件
  app.set("view engine","art");
  ```

##### 基础使用

```js
npm i art-template express-art-template -S
```

```js
const express=require("express");
var app=express();

app.engine('art', require('express-art-template'));//默认文件夹是根目录，这里指定为view文件夹
app.set("views",path.join(__dirname,"./view"));//设置views目录为view文件夹
app.set("view engine","art");//设置view内容默认为art文件，渲染时可以不加扩展名

app.get("/",function(req,res){
    res.render("index",{data});
})

app.listen(4001);
```

##### 高级用法

- **模板继承：模板继承允许你构建一个母版**

```js
<!--layout.art-->
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>{{block 'title'}}My Site{{/block}}</title>
    {{block 'head'}}<link rel="stylesheet" href="main.css">{{/block}}
</head>
<body>
    {{block 'content'}}{{/block}}
</body>
</html>
```

```js
<!--index.art-->
{{extend './layout.art'}}

{{block 'title'}}
	{{title}}
{{/block}}

{{block 'head'}}
    <link rel="stylesheet" href="custom.css">
{{/block}}

{{block 'content'}}
	<p>This is just an awesome page.</p>
{{/block}}
```

```js
var express=require("express");
var app=express();

app.engine('art', require('express-art-template'));
app.set("views",path.join(__dirname,"./view"));
app.set("view engine","art");

app.get("/",function(req,res){
   // index.art将应用继承过来的母版
   res.render("./index.art",{title:"xietian"});
})

app.listen(4001);
```

- **子模板**

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div>{{include "./template.art" man}}</div>
</body>
</html>
```

```js
<- template.art ->
<p>你好</p>
<div>{{name}}今年{{age}}岁了，{{sex}}性</div>
<select name="city" id="">
    {{each city}}
        <option>{{$value}}</option>
    {{/each}}
</select>
```

- **过滤器：过滤器函数的参数接受目标值**

``` js
var express=require("express");
var template=require("art-template");
var app=express();

app.engine('art', require('express-art-template'));
app.set("views",path.join(__dirname,"./view"));
app.set("view engine","art");

template.defaults.imports.randomColor=function(){
   var col="#";
   for(var i=0;i<6;i++){
      col+=Math.floor(Math.random()*16).toString(16);
   }
   return col;
}
template.defaults.imports.getSum=function(a,b){
  return a+b;
}

app.get("/",function(req,res){
   res.render("./template.art",{data:"xietian"});
})

app.listen(4001);
```

```js
{{extend './layout.art'}}
{{block 'content'}}
    <div style='color:{{randomColor}}'}>asdb</div>
    <div>{{getSum(5,6)}}</div>
{{/block}}
```

#### 使用数据自动生成静态页面

- **server：server.js**

``` js
const template = require('art-template')
const fsp = require('fs/promises')
const express = require('express')
const multer = require('multer')()
const path = require('path')
const cors = require('cors')
const app = express()

app.engine('art', require('express-art-template'))
app.use(express.static('./public'))
app.use(multer.none())
app.use(cors())

app.post('/info', async function (req, res) {
  const { filename, title, list, info, src, csshref, age, name, sex, city } = req.body
  const obj = { filename, title, list: list.split(','), info, src, csshref, man: { name, age, sex, city } }
  await writeFile(obj)
  res.end('生成成功！')
})

template.defaults.imports.randomColor = function () {
  return Array(6)
    .fill(1)
    .reduce((v) => v + (~~(Math.random() * 16)).toString(16), '#')
}

async function writeFile(item) {
  const str = template(path.join(__dirname, './view/template.art'), item)
  await fsp.writeFile(path.join(__dirname, 'public/html', item.filename + '.html'), str, 'utf8')
}

app.listen(4002)
```

- **server：template.art**

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{block 'title'}}Document{{/block}}</title>
    <link rel="stylesheet" href="{{csshref}}" />
  </head>
  <body>
    {{block 'list'}}
    <ul>
      {{each list}}
      <li>{{$value}}</li>
      {{/each}}
    </ul>
    {{/block}} {{block 'info'}}
    <div>{{info}}</div>
    {{/block}}
    <div style="width:100px;height:100px;background-color:{{randomColor}}">
      <span>1</span>
      <span>2</span>
    </div>
    <div class="div1"></div>
    <div>{{include "./c.art" man}}</div>
    <img src="{{src}}" />
  </body>
</html>
```

- **client：index.html**

```js
<form action="#">
  <label for="">filename:</label> <input type="text" name="filename" /><br />
  <label for="">title:</label> <input type="text" name="title" /><br />
  <label for="">list:</label> <input type="text" name="list" /><br />
  <label for="">info:</label> <input type="text" name="info" /><br />
  <label for="">src:</label>
  <select name="src" id="">
    <option>/img/img_72.jpg</option> // server端public静态文件路由下的img文件夹
    <option>/img/img_73.jpeg</option>
  </select>
  <label for="">csshref:</label>
  <select name="csshref" id="">
    <option>/css/a.css</option> // server端public静态文件路由下的css文件夹
    <option>/css/b.css</option>
  </select><br />
  <label for="">man:</label> <input type="text" name="name" /><br />
  <label for="">age:</label> <input type="text" name="age" /><br />
  <label for="">sex:</label> <input type="text" name="sex" /><br />
  <label for="">city:</label>
  <select name="city" id="" multiple>
    <option>北京</option>
    <option>上海</option>
    <option>深圳</option>
  </select>
  <button type="submit">提交</button>
</form>

<script>
  const form = document.querySelector('form')
  form.addEventListener('submit', submitHandler)

  async function submitHandler(e) {
    e.preventDefault()
    const fd = new FormData(form)
    await fetch('http://localhost:4002/info', {
      method: 'post',
      body: fd
    })
    location.href = 'http://localhost:4002/html/' + fd.get('filename') + '.html'
  }
</script>
```

