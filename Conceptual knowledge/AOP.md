## 什么是 AOP

> AOP（Aspect-Oriented Programming，面向切面编程）是一种编程思想，核心目的是**将业务逻辑与通用功能分离**
>
> * 在传统 OOP 里，我们会用继承、组合来复用代码，但像 **日志、事务管理、安全校验、缓存** 等横跨多个模块的功能（横切关注点），如果直接写在业务方法里，就会造成代码冗余、耦合度高。
> * AOP 就是把这些横切逻辑抽离出来，用“切面（Aspect）”统一处理。

### 核心概念

1. **切面（Aspect）**
   * 横切关注点的模块化封装，例如“日志切面”、“事务切面”。
2. **连接点（Join Point）**
   * 程序运行过程中能够插入切面逻辑的点，比如方法调用、异常抛出等。
3. **切点（Pointcut）**
   * 定义哪些连接点会被织入切面逻辑的表达式，比如“所有 `Service` 包里的方法”。
4. **通知（Advice）**
   * 切面在连接点执行的具体操作。常见类型：
     * **前置通知（Before）** ：方法执行前执行
     * **后置通知（After）** ：方法执行后执行
     * **返回通知（AfterReturning）** ：方法返回结果后执行
     * **异常通知（AfterThrowing）** ：抛出异常时执行
     * **环绕通知（Around）** ：方法执行前后都能织入逻辑
5. **织入（Weaving）**
   * 把切面逻辑应用到目标对象的过程。
   * 可以发生在编译期、类加载期、运行期（Spring AOP 就是运行期基于代理实现的）。

## 日志拦截器示例

- 创建日志拦截器

```ts
// log.interceptor.ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor
} from '@nestjs/common';
import { Observable, tap } from 'rxjs';

@Injectable()
export class LogInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    const req = context.switchToHttp().getRequest();
    const method = req.method;
    const url = req.url;

    console.log(`开始处理请求: ${method} ${url}`);

    return next.handle().pipe(
      tap(() =>
        console.log(`完成处理请求: ${method} ${url} - 耗时 ${Date.now() - now}ms`)
      )
    );
  }
}

```

- 应用到控制器

```ts
// app.controller.ts
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { LogInterceptor } from './log.interceptor';

@Controller()
@UseInterceptors(LogInterceptor)
export class AppController {
  @Get()
  getHello(): string {
    return 'Hello World!';
  }
}
```

## 自定义装饰器 + 元数据 + 拦截器实现切面控制

1. 自定义装饰器 (@Log) 定义

```ts
// log.decorator.ts
import { SetMetadata } from '@nestjs/common';

// 定义元数据键为 'log'，值为 true 的装饰器
export const Log = () => SetMetadata('log', true);
```

上述装饰器使用了 NestJS 的 `SetMetadata` 工具将元数据 `{ log: true }` 与目标方法相关联。当方法使用 `@Log()` 装饰时，即可在拦截器中通过该元数据识别需要执行额外的日志逻辑

2. 日志拦截器定义

```ts
// log.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LogInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // 获取当前处理器上由 @Log 装饰器设置的元数据
    const shouldLog = this.reflector.get<boolean>('log', context.getHandler());
    if (shouldLog) {
      console.log('【日志】执行前...');  // 在方法执行前记录日志
    }
    // 继续执行后续处理，并在响应后根据需要记录日志
    return next.handle().pipe(
      tap(() => {
        if (shouldLog) {
          console.log('【日志】执行后...'); // 在方法执行后记录日志
        }
      }),
    );
  }
}
```

在上面的拦截器中，我们使用 `Reflector.get()` 方法读取了键为 `'log'` 的元数据（即检查目标方法是否使用了 `@Log()` 装饰器）。如果元数据存在，则在调用 `next.handle()` 之前打印一条日志“执行前...”，然后利用 RxJS 的 `tap` 操作符在请求处理完成后再打印“执行后...”[**shiftasia.com**](https://shiftasia.com/community/mastering-custom-decorators-and-metadata-in-nestjs/#:~:text=return%20next.handle%28%29.pipe%28%20tap%28%28%29%20%3D,action%7D%60%29%3B%20%7D%20%7D%29%2C)。若没有找到该元数据（即方法未使用 `@Log`)，则 `shouldLog` 为 `undefined` 或 `false`，拦截器不会执行任何额外的日志操作，直接放行请求。

3. 控制器中应用装饰器和拦截器

```ts
// demo.controller.ts
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { LogInterceptor } from './log.interceptor';
import { Log } from './log.decorator';

@UseInterceptors(LogInterceptor)            // 将日志拦截器应用于此控制器
@Controller('demo')
export class DemoController {

  @Get('with-log')
  @Log()                                   // 使用自定义装饰器开启日志
  getWithLog() {
    return 'This route will be logged';
  }

  @Get('no-log')
  getNoLog() {
    return 'This route will not trigger logging';
  }
}
```

如上所示，我们在控制器类上通过 `@UseInterceptors(LogInterceptor)` 应用了日志拦截器，这样该控制器的所有路由都会经过此拦截器处理。然后，对于需要日志的路由方法（例如 `getWithLog`），加上了自定义装饰器 `@Log()`。当调用 `GET /demo/with-log` 时：

* NestJS 会先进入 `LogInterceptor.intercept()` 方法。拦截器通过 `Reflector` 检查到当前处理的方法有 `@Log` 元数据，因此打印“执行前...”日志。
* 然后控制器方法 `getWithLog()` 执行，返回结果。
* 方法执行完毕后，拦截器的 `tap` 回调被触发，再打印“执行后...”日志。

对于没有使用装饰器的路由（如 `getNoLog`），拦截器在执行时找不到元数据标记，`shouldLog` 为 false，因此不会打印任何日志，直接执行处理并返回结果。
