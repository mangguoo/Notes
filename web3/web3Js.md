> Web3.js是一个库集合，允许您使用HTTP、IPC或WebSocket与本地或远程以太坊节点进行交互。**Web3.js**允许我们开发与区块链交互的网站或客户端。例如，它允许我们将以太币从一个账户发送到另一个账户、从智能合约读取和写入数据、创建智能合约等等
>
> 各种高级语言编写的程序可以使用web3 interface来与EVM交互，在此过程中使用是的JSON-RPC（一个无状态且轻量级的远程过程调用（RPC）传送协议，其传递内容透过 JSON 为主）
>
> 使用以太坊开发区块链应用程序有几个不同的方面：
>
> - 智能合约开发 —— 使用 Solidity 编程语言编写部署到区块链的代码
> - 开发与区块链交互的网站或客户端 —— 编写代码，通过智能合约从区块链读取和写入数据，而客户端要操作区块链就需要使用web3.js
>
> 下图说明了**web3.js**如何与以太坊区块链交互：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-89486b3d926d430a64bc6b287dd4563c_r.jpg)

## web3.js和ehters.js

> web3.js和ethers.js都是以太坊JavaScript库
>
> - **Ether.js**由加拿大开发人员Rick Moore开发和维护
>
> - **Web3.js**由以太坊基金会开发和维护
>
> - **因此，对web3.js**有更广泛的支持，因为有更多的开发人员支持它

**ethers.js**和**web3.js**之间的一个主要区别是它们如何处理密钥管理以及与以太坊区块链的交互：

1. **Web3.js**假设有一个本地节点连接到应用程序。假设节点存储密钥、签署交易、读取以太坊区块链并与之交互。实际上这种情况并不常见——大多数用户并没有在本地运行geth。Metamask通过浏览器应用程序有效地模拟了该环境，因此大多数web3应用程序都需要Metamask来保存密钥、签署交易并与以太坊主网交互

2. 另一方面，**ethers.js**采用了一种不同的方法，我们认为这为开发人员提供了更大的灵活性。**Ethers.js**将“节点”分成两个不同的角色：

- 一个保存密钥和签署交易的“钱包”
- 充当与以太坊网络的匿名连接、检查状态和发送交易的“提供者”

## 如何使用web3.js

### 安装web3.js

```shell
$ npm install web3
```

### **Infura RPC URL**

要在主网上使用JSON RPC连接到以太坊节点，我们需要访问以太坊节点，有两种方法可以做到这一点：

1. 一种方法就是使用Geth或Parity运行自己的以太坊节点。但这需要从区块链下载大量数据并保持同步

2. 或者我们也可以使用ganache来模拟或fork一个自己本地测试的链

3. 可以使用Infura提供的api访问以太坊节点，而无需自己运行。Infura是一项免费提供远程以太坊节点的服务。我们只需要注册infura，然后获取到想要连接网络的API密钥和RPC URL。我们的Infura RPC URL 应如下所示：

```text
https://mainnet.infura.io/YOUR_INFURA_API_KEY
```

然后就可以通过该url访问以太坊节点了：

```js
var web3 = new Web3(url) //定义变量web3
```

### 获取链上数据

```js
// 这是使用ganache进行测试
var web3 = new Web3(Web3.givenProvider || "http://localhost:8545");

// 获取当前区块数量
web3.eth.getBlockNumber().then(res => {
    console.log(res)
})

// 获取链id
web3.eth.getChainId().then(res => {
    console.log(res)
})

//获取某个账号的余额(参数是要查询帐户的公钥)
web3.eth.getBalance("0x6b5aC29F2a2Ca361BE4fed60862C51D2F853842a").then(res => {
    console.log(web3.utils.fromWei(res, "ether"))
})
```

### 链上转帐

> 要进行转帐操作，需要知道当前帐户的公钥，因此需要钱包的授权，这是以metamask为例

可以调用如下api来获取metamask的授权，metamask会弹出一个窗口，需要我们确定授权哪个帐户。授权成功后就可以获取到一些钱包数据，第一条就是钱包公钥

```js
web3.eth.requestAccounts().then(res=>{
    console.log("授权",res)
    ourCount = res[0]
})
```

我们也可以在获取授权完成之后再通过另一个api来获取钱包公钥：

```js
web3.eth.requestAccounts().then(res=>{
    console.log("授权",res)
}).then(() => {
    return web3.eth.getAccounts()
}).then((res) => {
    console.log(res)
})
```

获取到授权，并且得到了授权帐户的公钥后，就可以发起转帐了：

```js
web3.eth.sendTransaction({
    from: ourCount, // 这个公钥必须是通过metamask授权过的帐户，不然转帐会失败
    to: toAccount,
    value: web3.utils.toWei("1", "ether")
})
    .then(function (receipt) {
        console.log("转完了")
    });
```
