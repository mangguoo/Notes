# pnpm

## pnpm的优势：

pnpm 内部使用`基于内容寻址`的文件系统来存储磁盘上所有的文件：

- 不会重复安装同一个包。使用`npm/yarn` 的时候，如果100个包依赖`lodash` ，那么就可能安装了100次`lodash` ，磁盘中就有100个地方写入了这部分代码。但是`pnpm`会只在一个地方写入这部分代码，后面使用会直接使用`hard link`

- 即使一个包的不同版本，pnpm 也会极大程度地复用之前版本的代码。举个例子，比如 lodash 有 100 个文件，更新版本之后多了一个文件，那么磁盘当中并不会重新写入 101 个文件，而是保留原来的 100 个文件的 `hardlink`，仅仅写入那`一个新增的文件`

monorepo的宗旨就是用一个git仓库来管理多个子项目，所有的子项目都存放在根目录的`packages`目录下，那么一个子项目就代表一个`package`

之前在使用npm/yarn的时候，由于node_module的扁平结构，如果A依赖B，B依赖C，那么A当中是可以直接使用C的，但问题是A当中并没有声明C这个依赖。因此会出现这种非法访问的情况。 但pnpm自创了一套依赖管理方式，很好地解决了这个问题，保证了安全性

## pnpm的依赖管理

> pnpm会在全局的store目录里存储项目`node_modules`文件的`hard links`。因为这样一个机制，导致每次安装依赖的时候，如果是个相同的依赖，有好多项目都用到这个依赖，那么这个依赖实际上最优情况(即版本相同)只用安装一次

### **第一阶段：npm@3 之前版本**

```text
node_modules
└─ foo
   ├─ index.js
   ├─ package.json
   └─ node_modules
      └─ bar
         ├─ index.js
         └─ package.json
```

- 依赖树层级太深，会导致 Windows 上的目录路径过长问题
- 相同包在不同的依赖项中需要时，会存在多个相同副本

### **第二阶段：npm@3 版本，扁平化处理**

所有的依赖都被拍平到`node_modules`目录下，不再有很深层次的嵌套关系。这样在安装新的包时，根据node require机制，会不停往上级的`node_modules`当中去找，如果找到相同版本的包就不会重新安装，解决了大量包重复安装的问题，而且依赖层级也不会太深

```text
node_modules
├─ foo
|  ├─ index.js
|  └─ package.json
└─ bar
   ├─ index.js
   └─ package.json
```

但还是存在一些问题：

- 依赖结构的**不确定性**
- 扁平化算法本身的**复杂性**很高，耗时较长
- 项目中仍然可以**非法访问**没有声明过依赖的包

这就是为什么会产生依赖结构的`不确定`问题，也是`lock 文件`诞生的原因，无论是`package-lock.json`(npm 5.x才出现)还是`yarn.lock`，都是为了保证 install 之后都产生确定的`node_modules`结构。尽管如此，npm/yarn本身还是存在`扁平化算法复杂`和`package 非法访问`的问题，影响性能和安全

### **第三阶段：pnpm**

> pnpm并不像yarn/npm一样使用了平铺的存储依赖的方式，而是使用了一种不同寻常的处理方式：创建两个目录，并在其中一个执行`npm add express`， 然后在另一个中执行`pnpm add express`

第一个目录中的`node_modules`的顶级项目：

```txt
.bin
accepts
array-flatten
body-parser
bytes
content-disposition
cookie-signature
cookie
debug
depd
destroy
ee-first
encodeurl
escape-html
etag
express
```

通过pnpm创建的得到的`node_modules`:

```txt
.pnpm
.modules.yaml
express
```

`node_modules`中只有一个叫`.pnpm`的文件夹以及一个叫做`express`的符号链接。pnpm只安装了 `express`，它是唯一一个app必须拥有访问权限的包。观察express目录：

```txt
▾ node_modules
  ▸ .pnpm
  ▾ express
    ▸ lib
      History.md
      index.js
      LICENSE
      package.json
      Readme.md
  .modules.yaml
```

可以看到它根本都没有node_modules文件夹，那express的依赖去哪了呢？答案是`express`只是一个符号链接。当Node.js解析依赖的时候，它使用这些依赖的真实位置。而`express`的真实位置在`node_modules/.pnpm/express@4.17.1/node_modules/express`

`.pnpm/`以平铺的形式储存着所有的包，所以每个包都可以在这种命名模式的文件夹中被找到：

```text
.pnpm/<name>@<version>/node_modules/<name>
```

`.pnpm/`被称之为虚拟存储目录，这个目录中的文件采用的都是硬链接。

这个平铺的结构避免了npm v2创建的嵌套`node_modules`引起的长路径问题，但与npm v3,4,5,6或yarn v1创建的平铺的 `node_modules`不同的是，它保留了包之间的相互隔离

下面就是express包的真实位置。pnpm把包的依赖项与依赖包的实际位置諈同一目录级别。 所以`express`的依赖不在 `.pnpm/express@4.17.1/node_modules/express/node_modules/` 而是在 [.pnpm/express@4.17.1/node_modules/](https://github.com/zkochan/comparing-node-modules/tree/master/pnpm5-example/node_modules/.pnpm/express@4.17.1/node_modules)：

```txt
▾ node_modules
  ▾ .pnpm
    ▸ accepts@1.3.5
    ▸ array-flatten@1.1.1
    ...
    ▾ express@4.16.3
      ▾ node_modules
        ▸ accepts
        ▸ array-flatten
        ▸ body-parser
        ▸ content-disposition
        ...
        ▸ etag
        ▾ express
          ▸ lib
            History.md
            index.js
            LICENSE
            package.json
            Readme.md
```

`express`所有的依赖都软链至了`node_modules/.pnpm/`中的对应目录。 把`express`的依赖放置在同一级别避免了循环的软链

## workspace

> pnpm内置了对单一存储库（也称为多包存储库、多项目存储库或单体存储库）的支持，可以创建一个workspace以将多个项目合并到一个仓库中

默认情况下，如果可用的packages与已声明的可用范围相匹配，pnpm将从工作区链接这些packages。 例如, 如果`bar`引用`"foo": "^1.0.0"`并且`foo@1.0.0`存在工作区，那么pnpm会从工作区将`foo@1.0.0`链接到`bar`。但是如果`bar`的依赖项中有`"foo": "2.0.0"`，而`foo@2.0.0`不存在于工作空间中，则将从npm registry安装`foo@2.0.0`。这种行为带来了一些不确定性

幸运的是，pnpm支持workspace协议`workspace:`。当使用此协议时，pnpm将拒绝解析除本地workspace包含的package之外的任何内容。因此设置为`"foo": "workspace:2.0.0"`时，安装将会失败，因为`"foo@2.0.0"`不存在于此workspace中

当[link-workspace-packages](https://pnpm.io/zh/npmrc#link-workspace-packages)选项被设置为`false`时，这个协议将特别有用。在这种情况下，仅当使用`workspace:`协议声明依赖，pnpm才会从此workspace链接所需的包

## 配置方式

要配置workspace，首先要向包管理工具声明workspace。比如在pnpm中，需要在根目录下创建`pnpm-workspace.yaml`文件，对工作区进行声明：

- 项目结构

```txt
my-monorepo
├─ docs
├─ apps
│  ├─ api
│  └─ mobile
├─ packages
│  ├─ tsconfig
│  └─ shared-utils
└─ sdk
```

- pnpm-workspace.yaml

```yaml
packages:
  - "docs"
  - "apps/*"
  - "packages/*"
```

在上面的例子中，`my-monorepo/apps/`和里面的所有目录`my-monorepo/packages/`都是工作空间，`my-monorepo/docs`目录本身也是一个工作空间。`my-monorepo/sdk/`不是工作区，因为它不包含在工作区配置中

## workspace命名

> 每个workspace都必须有一个唯一的名称，可以在其项目目录下的package.json中进行配置：

- packages/shared-utils/package.json

```json
{  
	"name": "shared-utils"
}
```

此名称将用于:

- 指定应将包安装到哪个工作区

- 在其他工作区中使用此工作区

- 发布包: 将以该名称在npm上发布

## 管理工作区

在monorepo中，当在根路径下运行install命令时，会发生以下事情:

- 检查已安装的工作区依赖项

- 任何工作区都被符号链接到node_module，这意味着可以像引入普通包一样引入它们

- 下载其他依赖到node_module中

这意味着无论何时添加/删除工作区时，或更改它们在文件系统中的位置，都需要在根目录下重新运行`install`以再次设置工作区

## 工作区选项

`pnpm <add|remove> <package> -W`

- -W: --ignore-workspace-root-check ，允许依赖被安装在workspace的根目录

```shell
# 安装eslint作为根目录的devDependencies
$ pnpm add eslint -D -W
```