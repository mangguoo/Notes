# NestJS 模块系统

## 模块核心概念

在 NestJS 中，`imports`、`exports`、`providers` 是模块化机制的三个核心概念：

### imports

引入其他模块，使当前模块能够使用外部模块提供的功能。当需要使用另一个模块的服务、仓储或工厂时，必须在 `imports` 中导入该模块。

### exports

将当前模块的某些 `providers` 或模块暴露出去，供其他模块使用。服务要在其他模块中复用，必须通过 `exports` 显式导出。

### providers

定义模块中的依赖注入对象（DI tokens），通常是 `Service`、`Repository`、`Factory`、`Helper` 等。NestJS 将这些 `providers` 注册到 IoC 容器中，通过依赖注入提供给控制器或其他服务使用。

## 模块导入机制

### 基本原则

- **导入使用**：模块通过 `imports` 引入其他模块，获取其导出的 `providers`
- **导出共享**：模块通过 `exports` 声明要对外提供的 `providers`
- **严格限制**：未导出的 `providers` 在其他模块中无法直接使用

### 导入的作用域

导入模块会产生两种效果：

1. **直接效果**：可以注入该模块导出的 `providers`
2. **间接效果**：该模块的控制器会注册到应用路由，内部 `providers` 会实例化并执行

**重要**：其他模块只能注入导出的 `providers`，未导出的 `providers` 仅限模块内部使用。

## 全局模块

### 什么是全局模块

全局模块是 NestJS 的特殊模式，通过 `@Global()` 装饰器将模块声明为全局。全局模块只需在应用启动时注册一次，其导出的 `providers` 在整个应用中都可使用，其他模块无需再次导入。

### 全局模块的特点

- **一次注册，全局使用**：在根模块注册后，其他模块可直接注入其导出的服务
- **仍需导出**：全局模块的 `providers` 仍需在 `exports` 中声明才能被注入
- **内部私有**：未导出的 `providers` 仍只在模块内部可用

### 模块类型总结


| 模块类型 | 使用方式 | 导出要求      | 导入要求      |
| ---------- | ---------- | --------------- | --------------- |
| 普通模块 | 局部使用 | 必须`exports` | 必须`imports` |
| 全局模块 | 全局使用 | 必须`exports` | 无需`imports` |

### 示例

```ts
@Global()
@Module({
  providers: [CatsService, InternalAuditService],
  exports: [CatsService], // 只导出 CatsService
})
export class CatsModule {}

// ✅ 可以注入（已导出）
constructor(private readonly cats: CatsService) {}

// ❌ 无法注入（未导出）
constructor(private readonly audit: InternalAuditService) {}
```

## 动态模块

### 什么是动态模块

动态模块是在运行时创建的模块，与静态模块具有相同的属性接口，但需要额外提供一个 `module` 属性来指定模块类。动态模块返回的对象必须符合 `DynamicModule` 接口。

```ts
interface DynamicModule {
  module: Type<any>
  global?: boolean
  imports?: any[]
  providers?: any[]
  controllers?: any[]
  exports?: any[]
}
```

### 创建动态模块

通过在模块类中定义静态方法来创建动态模块，通常命名为：

- **`forRoot()`**：全局一次性配置
- **`register()`**：允许多处使用不同配置

### 基本示例

```ts
import { DynamicModule, Module } from '@nestjs/common'
import { ConfigService } from './config.service'

@Module({})
export class ConfigModule {
  static register(options?: Record<string, any>): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    }
  }
}
```

使用时在其他模块的 `imports` 中调用：

```ts
@Module({
  imports: [ConfigModule.register({ folder: './config' })],
})
export class AppModule {}
```

### 配置注入方式

动态模块需要将配置选项传递给内部的 `providers`，NestJS 提供三种注入方式：

#### useValue（静态值）

直接将配置对象作为值提供，适用于启动时已确定的配置。

```ts
static register(options: Record<string, any>): DynamicModule {
  return {
    module: ConfigModule,
    providers: [
      { provide: 'CONFIG_OPTIONS', useValue: options },
      ConfigService,
    ],
    exports: [ConfigService],
  };
}
```

在服务中注入配置：

```ts
@Injectable()
export class ConfigService {
  constructor(@Inject('CONFIG_OPTIONS') private options: Record<string, any>) {
    // 使用 this.options 访问配置
  }
}
```

#### useFactory

通过工厂函数生成配置对象，支持同步/异步，可注入其他依赖。

**使用示例：**

```ts
// 应用模块中使用
@Module({
  imports: [
    ConfigModule.forRoot(),
    DemoModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (config: ConfigService) => ({
        path: config.get<string>('DEMO_PATH'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**动态模块实现：**

```ts
export const DEMO_OPTIONS = 'DEMO_OPTIONS'

@Module({})
export class DemoModule {
  static forRootAsync(options: DemoModuleAsyncOptions): DynamicModule {
    const asyncProvider: Provider = {
      provide: DEMO_OPTIONS,
      useFactory: options.useFactory,
      inject: options.inject || [],
    }

    return {
      module: DemoModule,
      imports: options.imports || [],
      providers: [asyncProvider, DemoService],
      exports: [DemoService],
    }
  }
}

@Injectable()
export class DemoService {
  constructor(@Inject(DEMO_OPTIONS) private options: DemoModuleOptions) {}
}
```

#### useClass

通过配置服务类生成配置对象，可以利用面向对象特性和依赖注入。

**使用示例：**

```ts
// 应用模块中使用
@Module({
  imports: [
    DemoModule.forRootAsync({
      useClass: DemoConfigService,
    }),
  ],
})
export class AppModule {}
```

**配置服务类：**

```ts
export interface DemoOptionsFactory {
  createDemoOptions(): DemoModuleOptions
}

@Injectable()
export class DemoConfigService implements DemoOptionsFactory {
  constructor(@Inject(SomeDependency) private dep: SomeDependency) {}

  createDemoOptions(): DemoModuleOptions {
    return { path: './config/path' }
  }
}
```

**动态模块实现：**

```ts
static forRootAsync(options: DemoModuleAsyncOptions): DynamicModule {
  const providers: Provider[] = [];

  if (options.useClass) {
    providers.push(
      { provide: options.useClass, useClass: options.useClass },
      {
        provide: DEMO_OPTIONS,
        useFactory: async (optionsFactory: DemoOptionsFactory) =>
          optionsFactory.createDemoOptions(),
        inject: [options.useClass],
      }
    );
  }

  return {
    module: DemoModule,
    imports: options.imports || [],
    providers: [...providers, DemoService],
    exports: [DemoService],
  };
}
```

如上，首先通过 `useClass` 指定的类型注册它本身为 provider，然后定义另一个工厂 `provider` ，`inject` 该类，以调用其
`createDemoOptions()` 方法获取配置。Nest 会先实例化 `DemoConfigService`（包括注入其依赖），再调用工厂 `provider`
的函数，从而获得配置对象并注入给 `DemoService` 等消费者使用。

**注意：** 提供给 `useClass` 的类必须实现对应的配置工厂接口 (如 `DemoOptionsFactory`) 并提供正确的方法。默认约定方法
名为 `create*()`，具体名称取决于接口定义。

#### useExisting

`useExisting`是`useClass` 的一种变体, 用于复用已在其他模块中提供的现有 `provider` 实例，避免创建新实例。

**模块注册方式：** 与 `useClass` 不同，`useExisting` 通常需要搭配 `imports` 使用，以确保要复用的 `provider` 已经在
当前模块上下文中。

**使用示例：**

```ts
@Module({
  imports: [
    OtherModule,
    DemoModule.forRootAsync({
      imports: [OtherModule],
      useExisting: OtherConfigService,
    }),
  ],
})
export class AppModule {}
```

在上例中，`OtherModule` 导出了 `OtherConfigService`，该服务实现了所需的配置工厂接口。通过 `useExisting:  OtherConfigService`，NestJS 将在导入 `DemoModule` 时查找已提供的 `OtherConfigService` 实例并直接使用它，而不是创
建新的实例。这要求 `OtherModule` 在导入 `DemoModule` 之前已注册 `OtherConfigService` 到 Nest 容器。

**已有服务类：**

```ts
@Injectable()
export class OtherConfigService implements DemoOptionsFactory {
  createDemoOptions(): DemoModuleOptions {
    return { path: process.env.DEMO_PATH || './default-path' }
  }
}

@Module({
  providers: [OtherConfigService],
  exports: [OtherConfigService],
})
export class OtherModule {}
```

**动态模块实现：**

```ts
if (options.useExisting) {
  providers.push({
    provide: DEMO_OPTIONS,
    useFactory: async (optionsFactory: DemoOptionsFactory) => optionsFactory.createDemoOptions(),
    inject: [options.useExisting],
  })
}
```

### 配置方式对比


| 方式          | 用途     | 特点                     |
| --------------- | ---------- | -------------------------- |
| `useValue`    | 静态配置 | 启动时确定的配置对象     |
| `useFactory`  | 动态配置 | 通过函数生成，可注入依赖 |
| `useClass`    | 类配置   | 通过类的方法生成配置     |
| `useExisting` | 复用配置 | 使用已存在的服务实例     |

## forRoot 与 forRootAsync

### 命名规范

- **forRoot/forRootAsync**：全局单例配置，应用中调用一次
- **register/registerAsync**：可在多个模块中重复注册不同配置

### 同步 vs 异步


| 方法           | 使用场景             | 配置方式                                  |
| ---------------- | ---------------------- | ------------------------------------------- |
| `forRoot`      | 启动时已知的静态配置 | 直接传入配置对象                          |
| `forRootAsync` | 运行时确定的动态配置 | 使用`useFactory`/`useClass`/`useExisting` |

### 使用示例

**forRoot（同步）：**

```ts
TypeOrmModule.forRoot({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
})
```

**forRootAsync（异步）：**

```ts
TypeOrmModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    type: 'mysql',
    host: config.get('DB_HOST'),
    port: config.get('DB_PORT'),
  }),
})
```

## 全局动态模块

动态模块也可以设置为全局模块，有两种方式：

### 设置方式

**方式一：返回对象中设置 `global: true`**

```ts
static forRoot(options: ConfigOptions): DynamicModule {
  return {
    global: true,
    module: ConfigModule,
    providers: [ConfigService],
    exports: [ConfigService],
  };
}
```

**方式二：使用 `@Global()` 装饰器**

```ts
@Global()
@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    // ...
  }
}
```

### 使用效果

```ts
// AppModule 中导入一次
@Module({
  imports: [ConfigModule.forRoot({ folder: './config' })],
})
export class AppModule {}

// 其他模块中直接注入使用，无需导入
@Injectable()
export class SomeService {
  constructor(private configService: ConfigService) {}
}
```

## 动态模块再导出

模块再导出允许将已配置的动态模块转发给其他模块使用，避免重复配置。

### 实现方式

```ts
@Module({
  imports: [ConfigModule.forRoot({ folder: './config' })], // 导入并配置
  exports: [ConfigModule], // 导出模块类（注意不是 forRoot() 调用）
})
export class CoreModule {}
```

### 使用效果

```ts
// 业务模块只需导入 CoreModule
@Module({
  imports: [CoreModule], // 自动获得已配置的 ConfigService
})
export class UserModule {
  constructor(private configService: ConfigService) {} // 可直接使用
}
```

### 关键要点

- **exports 中使用模块类**：`ConfigModule`，而非 `ConfigModule.forRoot()`
- **配置封装**：在核心模块中统一配置，业务模块通过导入获得服务
- **避免重复**：确保全局使用统一的配置

## 动态模块的继承特性

动态模块具有配置扩展特性：动态返回的对象会**扩展**（而非覆盖）原有的模块定义。

### 扩展效果

- **静态声明**：`@Module` 装饰器中定义的 `providers` 和 `exports`
- **动态生成**：`forRoot()` 方法中额外创建的 `providers` 和 `exports`
- **最终结果**：包含静态和动态的所有 `providers`

### 示例

```ts
@Module({
  providers: [ConnectionProvider], // 静态 provider
  exports: [ConnectionProvider], // 静态导出
})
export class DatabaseModule {
  static forRoot(entities: Entity[]): DynamicModule {
    const repositoryProviders = entities.map(/* 创建 repository providers */)

    return {
      module: DatabaseModule,
      providers: repositoryProviders, // 动态 providers
      exports: repositoryProviders, // 动态导出
    }
  }
}

// 最终模块导出：ConnectionProvider + repositoryProviders
```

## ConfigurableModuleBuilder

NestJS 提供 `ConfigurableModuleBuilder` 工具类来简化动态模块的创建，自动生成配置方法和提供者。

### 定义配置模块

```ts
// config.module-definition.ts
import { ConfigurableModuleBuilder } from '@nestjs/common'

export interface ConfigModuleOptions {
  folder: string
}

export const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN } = new ConfigurableModuleBuilder<ConfigModuleOptions>()
  .setClassMethodName('forRoot')
  .build()
```

### 创建模块

```ts
// config.module.ts
import { Module } from '@nestjs/common'
import { ConfigService } from './config.service'
import { ConfigurableModuleClass } from './config.module-definition'

@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule extends ConfigurableModuleClass {}
```

### 使用配置

```ts
// config.service.ts
import { Injectable, Inject } from '@nestjs/common'
import { MODULE_OPTIONS_TOKEN } from './config.module-definition'

@Injectable()
export class ConfigService {
  constructor(@Inject(MODULE_OPTIONS_TOKEN) private options: ConfigModuleOptions) {}

  get(key: string): string {
    // 实现配置获取逻辑
    return 'some value'
  }
}
```

### 使用模块

```ts
// app.module.ts
@Module({
  imports: [
    // 同步配置
    ConfigModule.forRoot({ folder: './config' }),

    // 异步配置
    ConfigModule.forRootAsync({
      useFactory: () => ({ folder: './config' }),
    }),
  ],
})
export class AppModule {}
```
