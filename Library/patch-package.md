# patch-package

> 在开发时，有时碰到依赖的类库有bug，需要修改，一般都有这几种处理方式
>
> 1. fork源码，修复bug然后提交pr，等待作者合并，释放新版本
> 2. 提issue 等待作者修复（等同于法1）
> 3. 使用patch-package打补丁，安装依赖后自动打上修改的内容
>
> 显然方法1和方法2在紧张的开发阶段不大现实，因此使用方法3更为合理

## 使用方法

- 要使用patch-package来打补丁，首先要定位到源码包错误的部分，并进行修改:

```shell
$ vim node_modules/some-package/brokenFile.js
```

- 修复完成后，就可以对修改部分生成补丁:

```shell
$ npx patch-package some-package
```

这条合集会在指令执行的目录中生成一个patches文件夹，并且在这个文件夹中生成补丁文件，命名方式为`包名+版本号.patch`

这个patch文件其实就是一些`git diff`记录描述，`patch-package`会将当前`node_modules`下的源码与原始源码进行`git diff`，并在项目根目录下生成一个`patch`补丁文件

- 最后提交补丁文件

```shell
$ git add patches/some-package+3.14.15.patch
$ git commit -m "fix brokenFile.js in some-package"
```

- 配置安装命令

```json
"scripts": {
+  "postinstall": "patch-package"
}
```

当其他同事拉到代码也需要对已知bug进行打补丁，只需要在`npm install`后执行`patch-package`命令即可，这个流程可借助`npm script`实现

## 优势

使用它可以避免`版本升级`带来的困扰，如果该npm包升级，可能会导致原先的修改产生错误，而patch-package有如下特性：

- 如果你装的包版本和你之前生成的补丁中记录的版本不一样，`npx patch-package`会直接报错`ERROR Failed to apply patch for package xxxx at path`
- 使用`git diff`来记录补丁比起重写一份源码的方法更节省空间，`即安全`，`又便捷`

# ppm patch

> 如果项目使用`pnpm`进行包管理的话，就不需要使用`patch-package`了，因为`pnpm`添加了`pnpm patch`和`pnpm pathc-commit`，支持了给依赖打补丁

## 补丁流程

1. 通过`pnpm patch <pkg>`命令拷贝一份依赖库的文件项目，然后用户对该拷贝的项目进行修改
2. 通过`pnpm patch-commit`命令对修改后的代码以及原来的代码进行`diff`，生成一个`xxx.patch`的文件，对应项目的`package.json`会有个`pnpm.patchedDependencies`字段来指向`patch`文件，之前其他人安装依赖后，会自动使用到该patch

## demo

- 首先初始化一个`pnpm`项目:

```shell
$ mkdir patch-demo
$ cd patch-demo
$ pnpm init
```

- 安装一个依赖:

```shell
$ pnpm install axios@1.4.0
```

- 运行下面的命令，生成一个包副本，对包进行修复:

```shell
$ pnpm patch axios@1.4.0
# You can now edit the following folder: /private/var/folders/nx/8sx5fl8x7tlctvn619x60mjw0000gn/T/d7265c3b95a387f7e5cc0440a64c797f

# Once you're done with your changes, run "pnpm patch-commit /private/var/folders/nx/8sx5fl8x7tlctvn619x60mjw0000gn/T/d7265c3b95a387f7e5cc0440a64c797f"
```

- 上面的指令中有两条输出，第一条就是生成的副本，可以在副本中进行修改，修改完成后就运行下面`pnpm`帮我们生成好的补丁指令:

```shell
$ pnpm patch-commit /private/var/folders/nx/8sx5fl8x7tlctvn619x60mjw0000gn/T/d7265c3b95a387f7e5cc0440a64c797f
```

- 指令执行完成之后，就可以看到根目录下生成了一个`patches`文件夹，里面有一个叫`axios@1.4.0.patch`的补丁文件，并且`package.json`文件也发生了更改:

```json
"pnpm": {
  "patchedDependencies": {
    "axios@1.4.0": "patches/axios@1.4.0.patch"
  }
}
```

