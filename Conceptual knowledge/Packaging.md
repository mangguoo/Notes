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

### 模块加载流程

> 当执行require('module')时，Node.js进行以下操作：

1. 解析模块路径

- 如果require('fs')这样调用的是Node.js内置模块，则直接返回模块，不需要文件解析。
- 如果require('./math')这样调用的是自定义模块，Node.js需要解析文件路径，寻找对应的.js文件。
- 如果require('some-package')这样调用的是npm依赖，Node.js会从node_modules目录中查找package.json，找到main字段对应的入口文件。

2. 读取和编译模块

  - Node.js读取模块文件内容，并将其封装在一个函数作用域内部，以防止变量污染全局作用域。
  - 事实上，Node.js会将模块代码包裹在一个IIFE（立即执行函数）中，类似于：

    ```js
    (function(exports, require, module, __filename, __dirname) {
      // 这里是 math.js 的代码
    });
    ```

	这样module.exports、require、__filename、__dirname等变量可以在模块内部使用。

3. 执行模块

- 解析module.exports，执行模块代码，导出需要暴露的内容。

- require() 返回module.exports，供调用方使用。

### 模块缓存机制

> Node.js会缓存已加载的模块，避免重复解析和执行，提高性能。缓存存储在require.cache中：

```js
console.log(require.cache); // 查看缓存的模块
```

如果某个模块已经被require()过，再次require()时，会直接返回缓存，不会重复执行该模块。

**示例**

- math.js

```js
console.log('math.js 被执行了');
module.exports = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b
};
```

- app.js

```js
const math1 = require('./math'); // 第一次 require，math.js 代码会执行
const math2 = require('./math'); // 第二次 require，直接从缓存中获取，不会执行 math.js
console.log(math1 === math2); // true，表明同一个模块实例被复用
```

- 输出：

```js
// math.js 被执行了
// true
```

可以看到，math.js只被执行了一次，后续的require()直接使用缓存。

### module.exports和exports的关系

> module.exports是真正的导出对象，require()返回的就是它的值。
>
> exports是module.exports的引用，可以用于快捷赋值，但不能直接赋值为新对象，否则module.exports不会生效。

```js
// 错误示例
exports = { foo: 'bar' }; // 这样不会修改 module.exports
// 正确示例
module.exports = { foo: 'bar' }; // 这样才是正确的导出方式
```

### require的查找规则
> Node.js按照以下顺序查找模块：

1. 内置模块（如 fs、path）

2. 文件模块

- 直接指定路径require('./math.js')

- 省略扩展名require('./math')，Node.js会按`.js → .json → .node`顺序补全并尝试加载

3. 目录作为模块

- require('./myModule')，若myModule是个目录，会查找myModule/index.js

4. node_modules目录

  逐级向上查找node_modules目录，直到找到模块或报错MODULE_NOT_FOUND

### require()的循环引用

> 如果两个模块相互require，Node.js会返回部分加载的模块，而不是陷入死循环。

**示例：**

- a.js

```javascript
console.log('a.js 被执行了');
const b = require('./b'); // 引入 b.js
console.log('在 a.js 中，b.done =', b.done);
module.exports.done = true;
```

- b.js

```javascript
console.log('b.js 被执行了');
const a = require('./a'); // 引入 a.js
console.log('在 b.js 中，a.done =', a.done);
module.exports.done = true;
```

- app.js

```javascript
require('./a');
```

- 执行node app.js输出：

```javascript
// a.js 被执行了
// b.js 被执行了
// 在 b.js 中，a.done = undefined
// 在 a.js 中，b.done = true
```

当b.js require('./a') 时，a.js尚未执行完毕，此时module.exports.done还是undefined，所以b.js拿到的是一个未完全加载的a.js模块。

## ESModule

> ES Module（简称 ESM）是 ES6（ECMAScript 2015）引入的官方模块系统，它的核心特点包括：
>
> 1. **静态分析**：ESM 是 **编译时加载**（相比 CommonJS 的运行时 `require()`），在解析代码时就确定模块依赖关系。
> 2. **导入导出声明**：使用 `import/export` 关键字进行模块管理。
> 3. **按需加载**：支持动态导入 (`import()`)，适用于按需加载模块。
> 4. **严格模式**：ESM 默认采用严格模式 (`use strict`) 进行代码解析。

### ESM的缓存机制

> ESM**和CommonJS一样**，同一个模块只会被执行一次，多次`import`只会返回同一个被缓存起来的实例。

- counter.js

```js
console.log('counter.js 被执行');

let count = 0;
export function increment() {
    count++;
    console.log('count:', count);
}
```

- app.js

```js
import { increment } from './counter.js';

increment(); // count: 1
increment(); // count: 2

// 再次导入，不会重新执行 counter.js
import { increment as inc } from './counter.js';
inc(); // count: 3
```

- 执行`node app.js`输出

```js
// makefileCopyEditcounter.js 被执行
// count: 1
// count: 2
// count: 3
```

可以看到**`counter.js`只被执行了一次**（即`console.log('counter.js被执行')` 只出现一次）。**`count`变量在不同`import`之间是共享的**，证明`import`**不会创建新实例**，而是**从缓存中获取同一个模块的导出对象**。

ESM**会缓存已经导入的模块**，这个缓存存储在**模块记录表（Module Record）**中。每次`import`一个已加载的模块时，都会直接从缓存中获取，而不是重新执行该模块的代码。**但是如果是不同的URL或不同的模块路径，即使文件内容相同，仍然会被当作不同的模块加载！**

```js
import './utils.js';  // 加载 utils.js
import './utils.js';  // 直接从缓存获取
import '/absolute/path/to/utils.js'; // ❌ 视为新模块，会重新执行
```

### **ESM的工作原理**

> ESModule（ESM）通过**静态分析**在代码执行之前**构建模块依赖图（Dependency Graph）**，这使得它可以**高效地管理模块加载**，并且**避免同步阻塞**。
>
> **模块依赖图（Dependency Graph）**是**模块与模块之间的依赖关系图**，它描述了哪些模块依赖于哪些其他模块。在ESM中，`import`语句是**静态的**，编译阶段就能解析模块关系，构建依赖图。
>
> *并且由于ESM是**静态解析**，构建工具（如 Webpack、Rollup、ESBuild）可以**消除未使用的代码（Tree Shaking）**，减少最终打包体积，提高性能。*

假设我们有以下模块结构：

```js
// app.js -> utils.js -> math.js
```

- math.js

```js
export function add(a, b) {
    return a + b;
}
```

- utils.js

```js
import { add } from './math.js';

export function calculate(a, b) {
    return add(a, b) * 2;
}
```

- app.js

```js
import { calculate } from './utils.js';

console.log(calculate(2, 3)); // 输出 10
```

- index.html

```html
<script type="module" src="app.js"></script>
```

在代码**执行之前**，JavaScript引擎会**静态解析**`import`语句，构建模块依赖图：

```
// app.js 依赖 utils.js
// utils.js 依赖 math.js
```

在执行`app.js`之前，ESM**已经解析好所有模块的依赖关系**，这样JavaScript引擎可以**高效地并行加载模块**，而不是像CommonJS那样**同步阻塞加载**。

### CommonJS的同步阻塞

> 在CommonJS中，`require()`是**同步的**，每次调用`require()`，Node.js都要**立即解析并执行模块代码**，这可能导致**同步阻塞**。

- math.js

```js
console.log('math.js 被加载');
module.exports = {
    add: (a, b) => a + b,
};
```

- app.js

```js
console.log('app.js 开始');
const math = require('./math.js');
console.log('app.js 继续执行');
```

- 执行`node app.js`，输出:

```js
// app.js 开始
// math.js 被加载
// app.js 继续执行
```

但是`require('./math.js')`**阻塞了`app.js`的执行**，直到`math.js`加载完毕。

### ESM动态导入

> ESM**支持动态导入**`import()`，用于懒加载模块，提高性能。

```js
async function loadMath() {
    const math = await import('./math.js'); // 仅在需要时加载
    console.log(math.add(2, 3)); // 输出 5
}

loadMath();
```

### ESM注意事项

- **`import`必须在顶层**

```js
if (true) {
    import { increment } from './counter.js'; // ❌ 语法错误
}

import { increment } from './counter.js'; // ✅ 在顶层导入
increment();
```



