## UMD

> UMD（Universal Module Definition）是一种模块化方案，它旨在使模块能够同时在多种环境下运行，比如在 AMD（Asynchronous Module Definition）、CommonJS（如 Node.js）和直接在浏览器全局环境中都可以使用。这样，无论你的代码运行在何种模块加载器或无模块环境下，都能正确加载模块。

下面是一个简单的 UMD 模型示例：

```js
(function (root, factory) {
  // AMD 环境
  if (typeof define === 'function' && define.amd) {
    define([], factory);
  }
  // CommonJS 环境（如 Node.js）
  else if (typeof exports === 'object') {
    module.exports = factory();
  }
  // 浏览器全局环境
  else {
    root.myModule = factory();
  }
}(typeof self !== 'undefined' ? self : this, function () {
  // 模块的私有变量和方法
  var message = "Hello, UMD!";

  function sayHello() {
    return message;
  }

  // 向外暴露的 API
  return {
    sayHello: sayHello
  };
}));
```

1. 自执行函数

   外层包裹一个自执行函数，通过传入全局对象（在浏览器中通常是window，在Node.js中是global，这里使用this作为通用处理）以及工厂函数factory。
2. 环境判断
   - AMD：如果检测到define函数且define.amd为真，则说明当前环境支持AMD模块，调用define方法注册模块。
   - CommonJS：如果检测到exports为对象，则说明当前环境为 CommonJS（如 Node.js），使用module.exports导出模块。
   - 全局变量：如果以上都不满足，则将模块直接挂载到全局对象上，通常在浏览器环境中使用。
3. 模块内容

   在factory函数中定义模块的内部逻辑和数据，并返回一个对象作为模块的接口。这样，在不同环境下加载后都能调用模块的 API。

## CommonJS

> CommonJS 是 Node.js 使用的模块化规范，主要通过 require 和 module.exports 实现模块的导入和导出。它的核心机制包括：
> 
> - 模块导出 (module.exports 或 exports)：将模块内的变量或函数暴露出去，供其他文件使用。
> - 模块导入 (require)：引入其他模块，获取 module.exports 导出的内容。
> - 模块缓存：在首次 require 时执行并缓存模块，后续 require 直接返回缓存，不会重复执行。
> 
> CommonJS 模块的工作原理主要涉及 模块解析、加载、执行、缓存 这几个关键步骤。Node.js 在 require() 时，会进行一系列操作，使模块能够被正确引入和使用。我们来详细拆解 CommonJS 的工作流程。

1. 模块加载流程

    当执行require('module')时，Node.js进行以下操作：

    (1) 解析模块路径
    - 如果require('fs')这样调用的是Node.js内置模块，则直接返回模块，不需要文件解析。
    - 如果require('./math')这样调用的是自定义模块，Node.js需要解析文件路径，寻找对应的.js文件。
    - 如果require('some-package')这样调用的是npm依赖，Node.js会从node_modules目录中查找package.json，找到main字段对应的入口文件。

(2) 读取和编译模块

  - Node.js读取模块文件内容，并将其封装在一个函数作用域内部，以防止变量污染全局作用域。
  - 事实上，Node.js会将模块代码包裹在一个IIFE（立即执行函数）中，类似于：

    ```js
      (function(exports, require, module, __filename, __dirname) {
          // 这里是 math.js 的代码
      });
    ```

这样 module.exports、require、__filename、__dirname 等变量可以在模块内部使用。

(3) 执行模块
解析 module.exports，执行模块代码，导出需要暴露的内容。

require() 返回 module.exports，供调用方使用。

1. 模块缓存机制
Node.js 会缓存已加载的模块，避免重复解析和执行，提高性能。缓存存储在 require.cache 中：

javascript
Copy
Edit
console.log(require.cache); // 查看缓存的模块
如果某个模块已经被 require() 过，再次 require() 时，会直接返回缓存，不会重复执行该模块。

示例
math.js
javascript
Copy
Edit
console.log('math.js 被执行了');

module.exports = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b
};
app.js
javascript
Copy
Edit
const math1 = require('./math'); // 第一次 require，math.js 代码会执行
const math2 = require('./math'); // 第二次 require，直接从缓存中获取，不会执行 math.js

console.log(math1 === math2); // true，表明同一个模块实例被复用
输出：

lua
Copy
Edit
math.js 被执行了
true
可以看到，math.js 只被执行了一次，后续的 require() 直接使用缓存。

3. module.exports 和 exports 的关系
module.exports 是真正的导出对象，require() 返回的就是它的值。

exports 是 module.exports 的 引用，可以用于快捷赋值，但不能直接赋值为新对象，否则 module.exports 不会生效。

错误示例

javascript
Copy
Edit
exports = { foo: 'bar' }; // 这样不会修改 module.exports
正确示例

javascript
Copy
Edit
module.exports = { foo: 'bar' }; // 这样才是正确的导出方式
4. require 的查找规则
Node.js 按照以下顺序查找模块：

内置模块（如 fs、path）

文件模块

直接指定路径 require('./math.js')

省略扩展名 require('./math')，Node.js 会按 .js → .json → .node 顺序补全并尝试加载

目录作为模块

require('./myModule')，若 myModule 是个目录，会查找 myModule/index.js

node_modules 目录

逐级向上查找 node_modules 目录，直到找到模块或报错 MODULE_NOT_FOUND

5. require() 的循环引用
如果两个模块相互 require，Node.js 会返回 部分加载的模块，而不是陷入死循环。

示例：

a.js
javascript
Copy
Edit
console.log('a.js 被执行了');
const b = require('./b'); // 引入 b.js
console.log('在 a.js 中，b.done =', b.done);
module.exports.done = true;
b.js
javascript
Copy
Edit
console.log('b.js 被执行了');
const a = require('./a'); // 引入 a.js
console.log('在 b.js 中，a.done =', a.done);
module.exports.done = true;
app.js
javascript
Copy
Edit
require('./a');
执行 node app.js 输出：

javascript
Copy
Edit
a.js 被执行了
b.js 被执行了
在 b.js 中，a.done = undefined
在 a.js 中，b.done = true
当 b.js require('./a') 时，a.js 尚未执行完毕，此时 module.exports.done 还是 undefined，所以 b.js 拿到的是一个未完全加载的 a.js 模块。




