# GitHubAction

> GitHubActions是一个持续集成和持续交付的平台，它可以帮助你通过自动化的构建（包括编译、发布、自动化测试）来验证你的代码，从而尽快地发现集成错误。github于2019年11月后对该功能全面开放，现在所有的github用户可以直接使用该功能。GitHub 提供 Linux、Windows 和 macOS 虚拟机来运行您的工作流程，或者您可以在自己的数据中心或云基础架构中托管自己的自托管运行器

## 什么是YAML

> 编写GithubAction的流程时，需要创建一个workflow工作流，workflow必须存储在你的项目库根路径下的`.github/workflows`目录中，每一个 workflow对应一个具体的.yml 文件（或者 .yaml）
> yml是YAML（YAML Ain’t Markup Language）语言的文件，以数据为中心，比properties、xml等更适合做配置文件，主要有以下几个特点:

- 大小写敏感
- 使用缩进表示层级关系
- 缩进只能使用空格，不能用 TAB 字符
- 缩进的空格数量不重要，只要层级相同的元素左对齐即可
- ‘#’ 表示注释

## Github Action基本概念

- workflow: 一个workflow就是一个完整的工作流过程，每个workflow包含一组jobs任务
- job : jobs任务包含一个或多个job ，每个job包含一系列的steps步骤
- step : 每个ste 步骤可以执行指令或者使用一个action动作
- action : 每个action动作就是一个通用的基本单元

## GitHubAction的使用

```yaml
name: hello-github-actions
# 触发 workflow 的事件
on:
  push:
    # 分支随意
    branches:
      - master
# 一个workflow由执行的一项或多项job
jobs:
  # 一个job任务，任务名为build
  build:
    #运行在最新版ubuntu系统中
    runs-on: ubuntu-latest
    #步骤合集
    steps:
      #新建一个名为checkout_actions的步骤
      - name: checkout_actions
        #使用checkout@v2这个action获取源码
        uses: actions/checkout@v2 
      #使用建一个名为setup-node的步骤
      - name: setup-node
        #使用setup-node@v1这个action
        uses: actions/setup-node@v1
        #指定某个action 可能需要输入的参数
        with:
          node-version: '14'
      - name: npm install and build
        #执行执行某个shell命令或脚本
        run: |
          npm install
          npm run build
      - name: commit push
        #执行执行某个shell命令或脚本
        run: |
          git config --global user.email xxx@163.com
          git config --global user.name xxxx
          git add .
          git commit -m "update" -a
          git push
         # 环境变量
        env:
          email: xxx@163.com  
```

### name

> 这个字段表示Workflow的名字，可以随便设置。如果省略该字段，默认为当前workflow的文件名

### on

> 表示触发该工作流的事件，它可以是一个字符串，也可以是一个数组。该事件还对触发条件进行限制

```yaml
#push时触发
on: push

#push和merge时触发
on: [push, merge]

#当master分支push时触发，可以限定分支或标签。
on:
  push:
    branches:
      - master
```

完整的事件列表，请查看[官方文档](https://help.github.com/en/articles/events-that-trigger-workflows)。除了代码库事件，GitHub Actions 也支持外部事件触发，或者定时运行

### jobs

> jobs表示要执行的一项或多项任务。jobs可以包含一个或多个job，一个job就是一个任务，这个任务可以包含多个步骤(steps)。需要注意的是每一个Job都是并发执行的并不是按照申明的先后顺序执行的， 如果多个job 之间存在依赖关系，那么你可能需要使用 needs：

```yaml
jobs:
  job1:
    ...
  job2:
    needs: job1
    ...
  job3:
    needs: [job1, job2]
    ...
```

### env

> 可以使用env给job或者step来配置环境变量：

- jobs->job->env
- jobs->job->steps.env

### steps

> steps可以包含一个或多个步骤，每个step步骤可以进行如下设置:

- name：步骤名称，步骤的名称
- env：该步骤所需的环境变量
- id: 每个步骤的唯一标识符
- uses: 使用哪个action，这个表示使用别人预先设置好的Actions（比如因为我代码中要用到python，所以就用了actions/setup-python@v1来设置python环境，不用我自己设置了）
- with: 指定某个action可能需要输入的参数
- run: 执行哪些指令，具体运行什么命令行代码
- continue-on-error: 设置为true允许此步骤失败job仍然通过
- timeout-minutes: step的超时时间

```yaml
name: Sync To Gitee
on: [ push ]
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
    #创建一个打印环境变量的步骤
    - name: PrintName
      env:
        name: "zhangsan"
      run: |
        echo $name
    #创建一个安装Python环境的步骤    
    - name: SetUpPython
      uses: actions/setup-python@v1
      with:
        python-version: 3.7
    #创建一个安装Python包的步骤 
    - name: Install dependencies
      run: |
          python -m pip install --upgrade pip
          pip install requests
          pip install bs4
          pip install lxml     
```

> 使用uses指的是这一步骤需要先调用哪个Action。Action是组成工作流最核心最基础的元素。每个Action可以看作封装的独立脚本，有自己的操作逻辑，我们只需要 uses并通过with传入参数即可。比如actions/checkout@v2就是官方社区贡献的用来拉取仓库分支的Action，你不需要考虑安装git命令工具，只需要把分支参数传入即可

## Action

> Github Actions 是GitHub的持续集成服务。持续集成由很多操作组成，比如登录远程服务器，发布内容到第三方服务等等，这些相同的操作完全可以提取出来制作成脚本供所有人使用。GitHub允许开发者把每个操作写成独立的脚本文件，存放到代码仓库，使得其他开发者可以引用该脚本，这个脚本就是一个Action。如果你需要某种功能的Action可以从GitHub社区共享的[action官方市场](https://github.com/marketplace?type=actions)查找，也可以自己编程Action开源出来供大家使用

既然actions是代码仓库，当然就有版本的概念，用户可以引用某个具体版本的action。下面都是合法的action引用:

```txt
actions/setup-node@74bc508 # 指向一个 commit
actions/setup-node@v1.0    # 指向一个标签
actions/setup-node@master  # 指向一个分支
```

## GitHub Actions中使用密文

> 在持续集成的过程中，我们可能会使用到自己的敏感数据，GithubActions提供了Secrets变量来帮我们屏闭这些敏感数据。我们可以在github repo上依次点击 Settings->Secrets->Actions->New repository secret创建一个敏感数据例如:OSS_KEY_ID，OSS_KEY_SECRET，然后我们就可以在GithubAction脚本中使用这些变量了:

```yaml
-  name: setup aliyun oss
    uses: manyuanrong/setup-ossutil@master
    with:
        endpoint: oss-cn-beijing.aliyuncs.com
        access-key-id: ${{ secrets.OSS_KEY_ID }}
        access-key-secret: ${{ secrets.OSS_KEY_SECRET }}
```

## 示例

> 使用GitHubAction实现Push代码发送邮件通知功能（前提是邮箱需要开通 SMTP 服务）

```yaml
name: github-action-email
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # 检出代码
    - name: CheckoutRepo
      uses: actions/checkout@v2
    # 获取master分支上最新一条提交的git log
    - name: GetLastLog
      id: git_log
      uses: Edisonboy/latest-git-log-action@main
      with:
        tag: origin/master
    # 发送邮件    
    - name: Send email
      uses: dawidd6/action-send-mail@v3
      with:
        server_address: smtp.qq.com
        server_port: 465
        username: ${{secrets.MAIL_USERNAME}}
        password: ${{secrets.MAIL_PASSWORD}}
        subject: Github Actions job result
        to: ${{secrets.MAIL_TOUSERNAME}}
        from: ${{secrets.MAIL_USERNAME}}
        body: ${{github.repository}} push log : ${{steps.git_log.outputs.log}}
```

> - `${{github.repository}}`: 获取当前仓库的所有者和仓库名称
>
> - `${{steps.git_log_outputs.log}}`: 获取step id为git_log的输出

> 使用GitHubAction部署slate文档

```yaml
name: Deploy Slate
on:
  push:
    branches: [ 'main' ]
jobs:
  deploy:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      # 检出代码
      - name: Checkout
        uses: actions/checkout@main
      # 构建Docker映像并缓存各阶段
      - name: docker
        uses: whoan/docker-build-with-cache-action@v5
        with:
          username: chainupcustody
          password: "${{ secrets.DOCKER_HUB_PASSWORD }}"
          image_name: slate
      # 执行编译脚本，在docker环境中打包代码
      - name: run deploy.sh
        id: run-deploy
        run: ./deploy.sh
      # 将打包后的文件发布至gh-pages分支，供github page发布
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

> 目前 Docker 官方维护了一个公共仓库[Docker Hub](https://hub.docker.com/)。在Docker中，当我们执行docker pull xxx的时候，它实际上是从[Docker Hub](https://hub.docker.com/)中去查找，这就是Docker公司为我们提供的公共仓库
>
> `whoan/docker-build-with-cache-action`，在这个`action`中，我们可以提供一个`registry`参数来指定一个docker仓库地址，以覆盖默认使用的docker hub仓库地址。这里的username和password参数就是表示Docker Hub的帐户及密码