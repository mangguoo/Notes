# dotenv

> Dotenv是一个零依赖模块，可将环境变量从`.env`文件加载到`process.env`

## 安装

```shell
$ npm install dotenv --save
```

## 使用

创建一个`.env`文件在项目的根目录中:

```txt
S3_BUCKET="YOURS3BUCKET"
SECRET_KEY="YOURSECRETKEYGOESHERE"

# 多行值
# dotenv的15.0.0版本以后支持换行符
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
...
Kh9NV...
...
-----END RSA PRIVATE KEY-----"

# 当然也可以使用\n来表示换行
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nKh9NV...\n-----END RSA PRIVATE KEY-----\n"
```

然后迟早在项目中导入并配置dotenv:

```js
require('dotenv').config()
console.log(process.env)
```

```js
import * as dotenv from 'dotenv'
dotenv.config()
import express from 'express'
```

# dotenv-expand

> `Dotenv-expand`在`dotenv`的基础上增加了变量扩展。如果您发现自己需要扩展已经存在于计算机上的环境变量，那么就可以使用`dotenv-expand`

## 安装

```shell
$ npm install dotenv-expand --save
```

## 使用

在项目的根目录中创建一个`.env`文件:

```
PASSWORD="s1mpl3"
DB_PASS=$PASSWORD
```

在应用程序中尽早导入和配置dotenv，然后拓展 dotenv:

```js
var dotenv = require('dotenv')
var dotenvExpand = require('dotenv-expand')

var myEnv = dotenv.config()
dotenvExpand.expand(myEnv)

console.log(process.env)
```