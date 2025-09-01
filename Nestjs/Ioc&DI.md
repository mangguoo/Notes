## IoC

> IoC（Inversion of Control，控制反转）
>
> 定义：把对象创建和依赖关系的管理，交给框架或容器，而不是由程序员在代码里手动控制。
>
> 核心思想：谁来“控制”对象的生命周期？传统方式是我们自己 new，IoC 则是容器来帮我们管理。

## DI

> DI（Dependency Injection，依赖注入）
>
> 定义：是 IoC 的一种实现方式，把一个类所需要的依赖对象，通过“注入”的方式传进去（而不是类自己创建）。

**注入方式**

- 构造函数注入

A 需要 B，不自己 new B()，而是外部注入进来

```js
class A {
  constructor(private b: B) {}
}
```

- Setter 注入：通过方法设置
- 属性注入：通过装饰器或容器直接赋值


## 两者关系

IoC 是思想，DI 是手段

IoC 描述了“不要自己管依赖，交给容器来反转控制”；

DI 是一种具体做法：把依赖对象注入进去，常见于 Spring、NestJS、Angular 等框架


## Reflect Metadata

> Reflect Metadata 是 ES7 的一个提案，它主要用来在声明的时候添加和读取元数据。

TypeScript 在 1.5+ 的版本已经支持它，你只需要：

* `npm i reflect-metadata --save`。
* 在 `tsconfig.json` 里配置 `emitDecoratorMetadata` 选项。

Reflect Metadata 的 API 可以用于类或者类的属性上，如：

```ts
function metadata(
  metadataKey: any,
  metadataValue: any
): {
  (target: Function): void;
  (target: Object, propertyKey: string | symbol): void;
};
```

`Reflect.metadata` 当作 `Decorator` 使用，当修饰类时，在类上添加元数据，当修饰类属性时，在类原型的属性上添加元数据，如：

```ts
@Reflect.metadata('inClass', 'A')
class Test {
  @Reflect.metadata('inMethod', 'B')
  public hello(): string {
    return 'hello world';
  }
}

console.log(Reflect.getMetadata('inClass', Test)); // 'A'
console.log(Reflect.getMetadata('inMethod', new Test(), 'hello')); // 'B'
```

## emitDecoratorMetadata

> `emitDecoratorMetadata` 是 TypeScript 提供的一个 **编译选项** ，用在 **配合装饰器（decorators）使用** 时，作用是：
>
> **在编译阶段自动为被装饰的类、方法、属性等，生成类型元数据（metadata），并通过 `Reflect.metadata` 存储。**

`tsconfig.json`

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

`demo.ts`

```js
function LogType(target: any, key: string) {
  const type = Reflect.getMetadata("design:type", target, key);
  console.log(`${key} type:`, type); 
}

class Person {
  @LogType
  name: string = "abc";
}
```

如果没有开启 `emitDecoratorMetadata`，运行时 `Reflect.getMetadata("design:type", target, key)` 会是 `undefined`，因为编译器不会把类型信息放进去。

开启后再进行编译，TypeScript 会在生成的 JS 中自动加上类似：

```js
__decorate([
    LogType,
    __metadata("design:type", String)
], Person.prototype, "name", void 0);

```

这样，`Reflect.getMetadata("design:type", target, key)` 就能得到 `String` 这个构造函数。

---

#### 常见 metadata key

当启用 `emitDecoratorMetadata` 后，TS 会自动在编译产物里写入这些：

* `design:type` —— 属性或参数的类型（例：`String`, `Number`, `Boolean`, `MyClass` 等）
* `design:paramtypes` —— 方法或构造函数的参数类型数组
* `design:returntype` —— 方法的返回值类型

---

#### 应用场景

* **依赖注入（DI / IoC 容器）** ：通过 `design:paramtypes` 自动推断构造函数参数类型，从而实例化依赖对象
* **验证框架（class-validator）** ：通过元数据知道字段类型，自动做验证
* **ORM 框架（TypeORM、Sequelize）** ：通过属性的 `design:type` 自动推断数据库字段类型


## 依赖注入实现

```typescript
// tsconfig 需启用：
// "experimentalDecorators": true,
// "emitDecoratorMetadata": true
import 'reflect-metadata';

type Class<T = any> = abstract new (...args: any[]) => T;

// —— 编译期：要求“构造参数全是 Class 或无参”——
type EnforceClassParams<C extends Class> =
  C extends new (...args: infer P) => any
    ? (P[number] extends Class ? C : never)
    : never;

class Container {
  private registry = new Set<Class>();
  private singletons = new Map<Class, any>();
  private resolving = new Set<Class>(); // 循环依赖检测

  register<T>(target: Class<T>) {
    this.registry.add(target);
  }

  resolve<T>(target: Class<T>): T {
    if (!this.registry.has(target)) {
      throw new Error(`未注册：${target.name}，请先使用 @Injectable() 或 container.register()`);
    }

    if (this.singletons.has(target)) {
      return this.singletons.get(target);
    }

    if (this.resolving.has(target)) {
      throw new Error(`检测到循环依赖：${target.name}`);
    }
    this.resolving.add(target);

    // 运行期：从 metadata 解析依赖
    const paramTypes: Class[] = Reflect.getMetadata('design:paramtypes', target) || [];

    // 运行期校验：只能注入“类”，排除 String/Number/Object 等
    const args = paramTypes.map((dep, idx) => {
      if (!this.isInjectableClass(dep)) {
        const hint =
          dep === String ? 'string' :
          dep === Number ? 'number' :
          dep === Boolean ? 'boolean' :
          dep === Object ? 'object/interface' :
          typeof dep;
        throw new Error(
          `${target.name} 构造参数 #${idx + 1} 不是可注入的类（实际是 ${hint}）。` +
          `请将其改为可注入的 Class，或改用 Token/Factory 注入。`
        );
      }
      if (!this.registry.has(dep)) {
        throw new Error(`依赖 ${dep.name} 未注册，请加 @Injectable() 或手动 container.register(${dep.name})`);
      }
      return this.resolve(dep);
    });

    const instance = new target(...args);
    this.singletons.set(target, instance);
    this.resolving.delete(target);
    return instance;
  }

  private isInjectableClass(dep: any): dep is Class {
    // 过滤内建包装类与非 class
    if (dep === String || dep === Number || dep === Boolean || dep === Object) return false;
    return typeof dep === 'function' && !!dep.prototype;
  }
}

export const container = new Container();

// —— 关键：让返回的装饰器“只接受构造参数全为 Class 的类” ——
// 使用返回类型上的泛型约束（编译期）
export function Injectable() {
  return function <C extends Class>(target: EnforceClassParams<C>) {
    container.register(target);
  };
}

// ========== 示例 ==========
@Injectable()
class OtherService {
  a = 1;
}

@Injectable()
class TestService {
  constructor(public readonly otherService: OtherService) {}
  testMethod() {
    console.log(this.otherService.a);
  }
}

container.resolve(TestService).testMethod(); // 1

// ❌ 下例会在“装饰器位置”直接编译报错（构造参数含 string，违反约束）
// @Injectable()
// class BadService {
//   constructor(private apiUrl: string) {}
// }
```

## Controller 与 Get 的实现

```ts
@Controller('/test')
class SomeClass {
  @Get('/a')
  someGetMethod() {
    return 'hello world';
  }

  @Post('/b')
  somePostMethod() {}
}
```

这些 `Decorator` 也是基于 `Reflect Metadata` 实现，可以将 `metadataKey` 定义在 `descriptor` 的 `value` 上：

```ts
const METHOD_METADATA = 'method'；
const PATH_METADATA = 'path'；

const Controller = (path: string): ClassDecorator => {
  return target => {
    Reflect.defineMetadata(PATH_METADATA, path, target);
  }
}

const createMappingDecorator = (method: string) => (path: string): MethodDecorator => {
  return (target, key, descriptor) => {
    Reflect.defineMetadata(PATH_METADATA, path, descriptor.value);
    Reflect.defineMetadata(METHOD_METADATA, method, descriptor.value);
  }
}

const Get = createMappingDecorator('GET');
const Post = createMappingDecorator('POST');
```

接着，创建一个函数，映射出 `route`：

```ts
function mapRoute(instance: Object) {
  const prototype = Object.getPrototypeOf(instance);

  // 筛选出类的 methodName
  const methodsNames = Object.getOwnPropertyNames(prototype)
                              .filter(item => !isConstructor(item) && isFunction(prototype[item]))；
  return methodsNames.map(methodName => {
    const fn = prototype[methodName];

    // 取出定义的 metadata
    const route = Reflect.getMetadata(PATH_METADATA, fn);
    const method = Reflect.getMetadata(METHOD_METADATA, fn)；
    return {
      route,
      method,
      fn,
      methodName
    }
  })
};
```

因此，我们可以得到一些有用的信息：

```ts
Reflect.getMetadata(PATH_METADATA, SomeClass); // '/test'

mapRoute(new SomeClass());

/**
 * [{
 *    route: '/a',
 *    method: 'GET',
 *    fn: someGetMethod() { ... },
 *    methodName: 'someGetMethod'
 *  },{
 *    route: '/b',
 *    method: 'POST',
 *    fn: somePostMethod() { ... },
 *    methodName: 'somePostMethod'
 * }]
 *
 */
```

最后，只需把 `route` 相关信息绑在 `express` 或者 `koa` 上就可以了
