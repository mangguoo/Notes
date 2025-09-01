# NestJS 自定义提供者

## IoC 容器与依赖注入机制

NestJS 的 **依赖注入（DI）** 基于 **IoC（控制反转）容器** 实现。IoC 容器负责管理和实例化应用中的依赖对象，而不是由开发者手动创建。

### 工作原理

在 NestJS 中，应用启动时会创建一个模块化的 IoC 容器，将所有 **提供者（provider）** 注册到容器中，并通过 **token** 来索引这些提供者。当某个类需要依赖注入时，Nest 会根据依赖参数的类型或指定的 token，从 IoC 容器中查找对应的 provider 实例并注入。

### 依赖注入流程

整个依赖注入过程分为以下步骤：

1. **声明提供者**：使用 `@Injectable()` 装饰器标记类，使其成为可被容器管理的提供者。例如，`CatsService` 类通过 `@Injectable()` 声明后被注册进容器，默认以类名作为 token。
2. **声明依赖关系**：在构造函数中声明依赖项，如 `constructor(private catsService: CatsService)`。
3. **模块注册提供者**：在模块的 `providers` 数组中注册提供者类或自定义 provider。
4. **容器解析注入**：当 Nest 实例化依赖类时，IoC 容器会：

   - 根据 token 查找对应的提供者定义
   - 如果尚无实例，创建新实例并缓存（默认单例模式）
   - 如果已有实例，直接返回缓存的实例

> **重要**：整个 DI 过程是**递归且自动**的。如果被注入的提供者本身还有依赖，Nest 会先解析其依赖，确保依赖图中所有节点按需实例化。

## Provider Token及其类型

**Token** 是 NestJS IoC 容器中标识和查找提供者的键值，每个注册到容器的 provider 都关联一个 token 用于匹配。

### Token类型

#### 1. 类Token

通常情况下，类本身就可作为 token 使用：

```ts
// 简写形式
providers: [CatsService]

// 完整配置（等价于上面）
providers: [{ provide: CatsService, useClass: CatsService }]
```

#### 2. 自定义Token

除了类以外，还支持以下类型：

- **字符串**：如 `'CONNECTION'`
- **Symbol**：如 `Symbol('CONFIG')`
- **TypeScript enum**：如 `ConfigToken.DATABASE`

### 自定义 Token 使用示例

```ts
// 定义自定义提供者
const connectionProvider = {
  provide: 'CONNECTION',
  useValue: connectionObject,
}

@Module({
  providers: [connectionProvider],
})
export class AppModule {}

// 注入自定义 token
@Injectable()
export class CatsRepository {
  constructor(@Inject('CONNECTION') private readonly connection: Connection) {}
}
```

### 使用注意事项

- 对于**非类 token**，必须使用 `@Inject(token)` 显式指定，否则 Nest 无法通过类型反射推断注入目标
- **字符串或 Symbol** token 有助于避免直接依赖具体类名，特别适用于需要抽象化或避免命名冲突的场景

## 自定义 Provider 的意义与用途

**自定义提供者（Custom Provider）** 是在默认类提供者之外，为 Nest IoC 容器定义自定义依赖解析方式的手段。当应用需求超出标准提供者模式时，就需要使用自定义 provider。

### 常见使用场景

#### 1. 自定义实例

需要注入已存在的对象实例，而不是让 Nest 通过构造函数实例化。

**应用场景**：

- 应用启动前已建立的数据库连接
- 第三方库提供的对象实例
- 预配置的工具类实例

#### 2. 复用现有类

使用同一个类的实例满足不同的依赖需求。

**应用场景**：

- 通过不同 token 注入同一服务
- 为同一类创建多个别名

#### 3. 替换或模拟实现

在特定场景下用替代实现来替换提供者。

**应用场景**：

- 单元测试中使用 Mock 对象
- 不同环境使用不同实现

### 自定义 Provider 类型

Nest 提供了多种定义自定义 provider 的方式：


| 类型          | 用途       | 示例               |
| --------------- | ------------ | -------------------- |
| `useValue`    | 提供静态值 | 配置对象、常量     |
| `useClass`    | 指定实现类 | 接口的具体实现     |
| `useExisting` | 创建别名   | 同一实例的多个引用 |
| `useFactory`  | 动态创建   | 复杂的创建逻辑     |

### 基本结构

自定义 provider 本质上是包含以下属性的对象：

```ts
{
  provide: Token,        // 指定 token
  useValue/useClass/useExisting/useFactory: Implementation  // 指定实现方式
}
```

> **注意**：本文将重点讨论 `useFactory` 工厂提供者，其他类型将在后续章节详细介绍。

## Factory Provider（useFactory）机制详解

**工厂提供者（Factory Provider）** 使用 `useFactory` 选项，通过调用工厂函数来动态生成提供者的值。与直接提供类或常量不同，`useFactory` 允许在提供者创建过程中编写任意逻辑。

### 核心机制

工厂函数可以是简单的（无依赖）或复杂的（需要依赖其他 provider）。Nest 提供两项机制来支持依赖注入：

#### 1. 工厂函数参数

工厂函数可以声明参数，Nest 会自动注入对应的依赖对象。

#### 2. inject 属性

通过 `inject` 数组明确列出工厂函数所需的 provider token 列表，Nest 按顺序解析并传入函数参数。

### 完整示例

```ts
const connectionProvider = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider, optionalValue?: string) => {
    const options = optionsProvider.getOptions()
    // 基于注入的配置选项和可选值创建数据库连接
    return new DatabaseConnection(options, optionalValue)
  },
  // 按顺序指定依赖注入
  inject: [
    OptionsProvider,
    { token: 'OptionalToken', optional: true }, // 可选依赖
  ],
}

@Module({
  providers: [connectionProvider, OptionsProvider],
})
export class AppModule {}
```

### 执行流程

1. **解析依赖**：Nest 根据 `inject` 数组解析所需的 provider 实例
2. **调用工厂**：将解析的实例按顺序传入工厂函数
3. **缓存结果**：工厂函数返回值成为对应 token 的 provider 实例
4. **注入使用**：其他地方通过 token 注入时获得工厂创建的对象

### 可选依赖

对于可选依赖，可以这样配置：

```ts
inject: [RequiredProvider, { token: 'OptionalToken', optional: true }]
```

如果找不到可选依赖，Nest 会传入 `undefined` 而不会报错。

### 异步支持

**重要特性**：工厂提供者支持异步创建

- 可以返回 `Promise`
- Nest 会等待 Promise 完成再将结果注入
- 适用于异步初始化资源的场景

## 工厂提供者的优势与应用场景

使用 `useFactory` 定义提供者具有极大的灵活性，主要体现在以下方面：

### 核心优势

#### 1. 动态创建与条件逻辑

工厂函数可以根据运行时条件动态决定提供者的输出。

**典型应用**：

- 根据环境变量选择不同配置
- 根据应用状态决定使用不同实现
- 动态选择数据库连接策略

```ts
const configProvider = {
  provide: 'CONFIG',
  useFactory: () => {
    return process.env.NODE_ENV === 'production' ? productionConfig : developmentConfig
  },
}
```

#### 2. 组合多个依赖

通过 `inject` 注入多个依赖，工厂函数可以将多个服务组合创建复杂对象。

**适用场景**：

- 封装外部库的初始化（数据库客户端、缓存客户端等）
- 将配置、日志等服务注入到工厂中，生成预配置的实例

#### 3. 提供任意类型的值

工厂提供者不局限于返回类实例，可以返回：

- 普通对象
- 数组
- 函数
- 基础类型值

#### 4. 可选依赖与容错

支持可选依赖，提高模块的复用性和健壮性：

- 某些功能模块可选加载
- 服务未注册时仍能正常工作
- 降级处理和兜底方案

### 最佳实践场景

Factory Provider 特别适合以下场景：


| 场景             | 描述                       | 示例                     |
| ------------------ | ---------------------------- | -------------------------- |
| **配置管理**     | 根据环境动态选择配置       | 开发/生产环境配置        |
| **外部库初始化** | 封装第三方库的复杂初始化   | Redis 客户端、数据库连接 |
| **条件实例化**   | 根据条件决定创建哪种实现   | 支付服务的不同提供商     |
| **资源组合**     | 将多个依赖组合成新对象     | 聚合多个 API 的门面服务  |
| **异步初始化**   | 需要异步操作才能创建的资源 | 网络资源、文件读取       |

### 总结

Factory Provider 为开发者提供了在 Nest IoC 容器中**以编程方式构建依赖**的能力，使复杂场景下的依赖注入更加灵活可控。通过合理运用工厂提供者，既能保持代码的模块化和可测试性，又能满足应用在构建对象时的特殊需求和条件逻辑，融合了 IoC 容器的便利与手动控制的灵活性。
