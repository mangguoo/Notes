## declare

### 声明模块：

```ts
declare module 'lodash' {
    export function join(arr: any[]): void
}
```

```ts
import lodash from 'lodash'

console.log(lodash.join(["abc", "cba"]))
```

### 声明变量/函数/类：

> 在/index.html模板中写了一段script代码：

```html
<script>
    let whyName = "coderwhy"
    let whyAge = 18
    let whyHeight = 1.88

    function whyFoo() {
        console.log("whyFoo")
    }

    function Person(name, age) {
        this.name = name
        this.age = age
    }
</script>
```

在webpack打包完成后，打包的bundle.js是会被插入到这个模板中的，也就是说我们在/src/main.ts中写如下代码是可以访问到在模板中定义的变量的：

```ts
console.log(whyName)
console.log(whyAge)
console.log(whyHeight)
whyFoo()
const p = new Person("why", 18)
console.log(p)
```

但是这些代码是会报错的，因为ts编译器根本不知道我们在哪里定义了这些东西。解决方法就是对这些变量进行声明，delclare声明就像是一种断言，告诉tsc让它放心，我的程序中一定有这个类型的变量，你放心访问就可以了:

```ts
declare let whyName: string
declare let whyAge: number
declare let whyHeight: number

declare function whyFoo(): void

declare class Person {
    name: string
    age: number
    constructor(name: string, age: number)
}
```

### 声明文件：

> 在某些情况下，我们也可以声明文件：
>
> - 比如在开发vue的过程中，默认是不识别我们的.vue文件的， 那么我们就需要对其进行文件的声明；
> - 比如在开发中我们使用了 jpg 这类图片文件，默认typescript也是不支持的，也需要对其进行声明；

```ts
declare module '*.jpg' {
    const src: string
    export default src
}

declare module '*.vue' {
    import { DefineComponent } from 'vue'
    const component: DefineComponent
    export default component
}
```

```ts
import nhltImage from './img/nhlt.jpg'
import ComponentA from './component/ComponentA.vue'
```

### 声明命名空间：

> 对象可以使用namespace进行声明，因为namespace本质上就是通过立即执行函数返回了一个对象

```html
<script>
    let obj = {
        name: "ilmango",
        age: 18,
        showMes: (message) => {
            console.log(message);
        }
    }
</script>
```

```ts
declare namespace obj {
    export let name: string
    export let age: number
    export function showMes(message: string): void
}
```

```ts
console.log(obj.name)
console.log(obj.age)
obj.showMes("aaa")
```

### 声明模板html文件中引入的第三方库：

> 比如我们在index.html中直接引入了jQuery，我们可以进行命名空间的声明。因为此库是通过<script>标记（而不是模块加载器ESModule）加载的，所以它的声明使用命名空间来定义其形状。
>
> 可以看到jquery会在全局声明一个标识符\$来标识jquery，因此我们可以通过namespace来声明它。

#### 声明jquery

```ts
declare namespace $ {
    export function ajax(settings: any): any
}
```

```ts
$.ajax({
    url: "http://123.207.32.32:8000/home/multidata",
    success: (res: any) => {
        console.log(res)
    }
})
```

#### 为Express.Response添加属性：

> 声明文件，可以在此文件中给已有类型添加属性，它可以给js文件添加类型支持，在 .d.ts 文件中声明一个类型一定要用declare，如果你有此文件，则一定要在 tsconfig.json文件中开启 baseUrl 配置，baseUrl要为 ./ ，在include中配置的路径都是以baseUrl为基准的，比如app.d.ts文件的路径为 ./app.d.ts，并要修改一下 tsconfig中"include"的配置: ["src", "app.d.ts"]（把类型声明文件加入到编译中）

```ts
// 由于这个文件中没有导入任何模块，所以这个文件中的类型是全局的(脚本模式)
// 如果在模块中想要进行外参类型声明，可以使用declare global：
// declare global {
//   namespace Express {
//     interface Response {
//       api: (code: number, data: any) => void;
//     }
//   }
// }

declare global {
  interface Window {
    // 为window添加属性
    test: string;
  }
}
// 利用namespace会合并的特性，给Express添加属性
declare namespace Express {
  // 由于我们这里是想给Express.Request添加属性，所以这里要用interface进行合并，而不是导出一个新的类型
  interface Response {
    api: (code: number, data: any) => void;
  }
}
```

