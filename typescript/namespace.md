# namespace

## namespace自动合并

>  与接口类似，如果有多个同名的命名空间，那么typescript将递归合并名称相同的命名空间

**编译前：**

```ts
namespace app {
  export function test() {
    console.log("test");
  }
}

namespace app {
  export function test1() {
    console.log("test");
  }
}
```

**编译后：**

> 他定义了一个叫app的全局变量，相同的命名空间共用了一个全局变量，因此看上去两个命名空间合并了

```js
var app;
(function (app) {
    function test() {
        console.log("test");
    }
    app.test = test;
})(app || (app = {}));
(function (app) {
    function test1() {
        console.log("test");
    }
    app.test1 = test1;
})(app || (app = {}));
```

## 导出命名空间

- **编译前：**

```ts
namespace app {
  export function test() {
    console.log("test");
  }
}

namespace app {
  export function test1() {
    console.log("test");
  }
}

export default app;
```

- **编译后：**

```js
"use strict";
exports.__esModule = true;
var app;
(function (app) {
    function test() {
        console.log("test");
    }
    app.test = test;
})(app || (app = {}));
(function (app) {
    function test1() {
        console.log("test");
    }
    app.test1 = test1;
})(app || (app = {}));
exports["default"] = app;
```

