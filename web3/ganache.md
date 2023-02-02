> Ganache 是一个用与本地开发的区块链，用于在以太坊区块链上开发去中心化的应用程序。Ganache 模拟了以太坊网络，你可以在发布到生产环境之前看到你的 DApp 将如何执行
>
> 在以太坊网络上开发去中心化应用，通常的方式是设置一个以太坊客户端，如 Geth 或 OpenEthereum，为你提供[以太坊虚拟机(EVM)](https://link.zhihu.com/?target=https%3A//ethereum.org/en/developers/docs/evm/)环境。虽然这是一个在以太坊上开发去中心化应用的好方法，但它不是最有效和最友好的方法，因为你需要手动配置这些客户端并保持运行。维护一个自我托管的节点可能是昂贵和耗时的，你不希望在开发过程中花费宝贵的时间来排查一个失败的节点
>
> 而有了 Ganache，你所需要做的就是启动应用程序，你就有一个预先配置好的以太坊客户端，有 10 个预先存款和解锁的账户可以使用。这使你能够在整个开发周期内快速测试你的 DApp

## **安装ganache:**

```shell
$ npm install ganache --global
```

## **运行ganache:**

```shell
$ ganache
```

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-c15bdf3fd266ba25585453a832c66166_720w.webp)

默认情况下，Ganache提供了10个测试账户，每个账户有1000个（假）以太币，以及相应的私钥和用于生成私钥的助记词。在开发过程中，你可以使用这个助记词将账户导入到[MetaMask](https://link.zhihu.com/?target=https%3A//metamask.io/)等钱包。

我们可以在启动Ganache时通过指定选项来覆盖所有的默认值；例如，我们可以通过提供私钥与账户余额的映射来对账户创建进行更多的控制，就像这样：

```shell
$ ganache --wallet.accounts "0xfd485338e322f5930f7cf475f385341ec88bfc4f8a0a16f30b2fb417d1bb5427, 1000000000000000000000" "0x05bba0b9f7a251080aa23feee4eab3f75a1abee905c0271008c93e5d2e2e7541, 10000000000000000000000"
```

我们还可以指定一个助记词组来推导所有的初始账户，一个矿工Gas价格，以及区块Gas限制。例如：

```shell
$ ganache --miner.defaultGasPrice 200 --miner.blockGasLimit 90071 --miner.callGasLimit 898989 --wallet.mnemonic "alarm cause brave super lab glide awake hunt rose win sugar idea"
```

### ganache的常用参数：

> 运行命令`ganache --help`可以获得所有 Ganache 的可用选项的列表

我们每次在重启电脑时，相当于是会重新执行一遍ganache运行环境，它生成的账号也会跟着重置，这样我们就得更新metamask中的账户信息，非常不方便。要解决这个问题也每简单，只需要每次运行ganache的时候加上-d参数，它新生成的账户就会与上一次的保持相同，并且每个账户的金额会重置为1000

```shell
$ ganache -d
```



## 以编程方式使用ganache

- 作为一个Web3.js的provider

```js
const Web3 = require("web3");
const ganache = require("ganache");

const web3 = new Web3(ganache.provider());
...
```

- 作为Ethers.js的provider

```js
const ganache = require("ganache");

const provider = new ethers.providers.Web3Provider(ganache.provider());
```

- 作为一个独立的EIP-1193的provider:

```js
const ganache = require("ganache");

const options = {};
const provider = ganache.provider(options);
const accounts = await provider.request({ method: "eth_accounts", params: [] });
...
```

- 作为一个JSON-RPC网络服务器和EIP-1193的provider:

```js
const ganache = require("ganache");

const options = {};
const server = ganache.server(options);
const PORT = 8545;
server.listen(PORT, err => {
  if (err) throw err;

  console.log(`ganache listening on port ${PORT}...`);
  const provider = server.provider;
  const accounts = await provider.request({ method: "eth_accounts", params:[] });
});
...
```

