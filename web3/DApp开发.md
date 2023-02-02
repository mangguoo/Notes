## 开发环境

> Truffle是一个世界级的智能合约开发框架，专门为智能合约而生
>
> - 管理智能合约的生命周期
> - 自动化合约测试
> - 可编程，可部署，可发布合约
> - 不用过多的关注网络管理
> - 强大的交互式控制台

### **部署环境**

```shell
$ npm install truffle -g
$ truffle init
```

**`truffle init`用于生成开发环境的脚手架，它所生成的目录结构如下：**

- **contracts**/ : 存放solidity智能合约文件

- **migrations/** : truffle使用migration system 来控制合约的部署

- **test/** : 测试文件存放文字（javascript or solidity）

- **truffle-config.js** : 配置文件

### 配置文件

- `/truffle-config.js`

```js
module.exports = {
  /**
   * Networks define how you connect to your ethereum client and let you set the
   * defaults web3 uses to send transactions. If you don't specify one truffle
   * will spin up a managed Ganache instance for you on port 9545 when you
   * run `develop` or `test`. You can ask a truffle command to use a specific
   * network from the command line, e.g
   *
   * $ truffle test --network <network-name>
   */

  networks: {
    // Useful for testing. The `development` name is special - truffle uses it by default
    // if it's defined here and no other network is specified at the command line.
    // You should run a client (like ganache, geth, or parity) in a separate terminal
    // tab if you use this network and you must also set the `host`, `port` and `network_id`
    // options below to some value.
    //
    development: {          // 开发环境预设配置，用于本地测试，连接ganache
     host: "127.0.0.1",     // Localhost (default: none)
     port: 8545,            // Standard Ethereum port (default: none)
     network_id: "*",       // Any network (default: none)
    },
    //
    // An additional network, but with some advanced options…
    // advanced: {               // 更具体的本地开发环境预设配置，进行了更复杂的配置，from表示部署运行智能合约所消耗的gas从哪个账户进行扣除，默认是所生成的第一个帐号
    //   port: 8777,             // Custom port
    //   network_id: 1342,       // Custom network
    //   gas: 8500000,           // Gas sent with each transaction (default: ~6700000)
    //   gasPrice: 20000000000,  // 20 gwei (in wei) (default: 100 gwei)
    //   from: <address>,        // Account to send transactions from (default: accounts[0])
    //   websocket: true         // Enable EventEmitter interface for web3 (default: false)
    // },
    //
    // Useful for deploying to a public network.
    // Note: It's important to wrap the provider as a function to ensure truffle uses a new provider every time.
    // goerli: {
    //   provider: () => new HDWalletProvider(MNEMONIC, `https://goerli.infura.io/v3/${PROJECT_ID}`),
    //   network_id: 5,       // Goerli's id
    //   confirmations: 2,    // # of confirmations to wait between deployments. (default: 0)
    //   timeoutBlocks: 200,  // # of blocks before a deployment times out  (minimum/default: 50)
    //   skipDryRun: true     // Skip dry run before migrations? (default: false for public nets )
    // },
    //
    // Useful for private networks
    // private: {
    //   provider: () => new HDWalletProvider(MNEMONIC, `https://network.io`),
    //   network_id: 2111,   // This network is yours, in the cloud.
    //   production: true    // Treats this network as if it was a public net. (default: false)
    // }
  },

  // Set default mocha options here, use special reporters, etc.
  mocha: {
    // timeout: 100000
  },

  // Configure your compilers
  compilers: { // 编译器配置，optimizer为优化选项，可以打开
    solc: {
      version: "0.8.17", // Fetch exact version from solc-bin (default: truffle's version)
      // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
      settings: {          // See the solidity docs for advice about optimization and evmVersion
       optimizer: {
         enabled: false,
         runs: 200
       },
       evmVersion: "byzantium"
      }
    }
  }

  // Truffle DB is currently disabled by default; to enable it, change enabled:
  // false to enabled: true. The default storage location can also be
  // overridden by specifying the adapter settings, as shown in the commented code below.
  //
  // NOTE: It is not possible to migrate your contracts to truffle DB and you should
  // make a backup of your artifacts to a safe location before enabling this feature.
  //
  // After you backed up your artifacts you can utilize db by running migrate as follows:
  // $ truffle migrate --reset --compile-all
  //
  // db: {
  //   enabled: false,
  //   host: "127.0.0.1",
  //   adapter: {
  //     name: "indexeddb",
  //     settings: {
  //       directory: ".db"
  //     }
  //   }
  // }
};
```

## 简单的智能合约

- `/contracts/StudentStorage.sol`

```solidity
// SPDX-License-Identifier: GPL-3.0 、、
// 源码遵循协议， MIT...
pragma solidity >=0.4.16 <0.9.0; //限定solidity编译器版本

contract StudentStorage { // 合约名必须与文件名相同
    // 创建两个状态变量(状态变量默认存储方式为storage)
    uint public age;
    string public name;

    // 复杂数据类型(struct、动态数组、映射、string等)才需要设置memory，而像int类型这种基本数据类型作为函数参数时默认为memory
    function setData(string memory _name, uint _age) public {
        // string memory a; // 定义局部变量(尽量定义为memory)
        name = _name;
        age = _age;
    }
    
    // pure纯函数
    // function test(uint x ,uint y) public pure returns (uint){
    //     return x + y;
    // }

    // view视图函数
    function getData() public view returns (string memory, uint) {
        return (name, age);
    }
}
```

### 数据存储位置

> solidity数据存储位置有三类：storage，memory和calldata，不同存储位置的gas成本不同
>
> - storage类型的数据存在链上，类似计算机的硬盘，消耗gas多
> - memory和calldata类型的临时存在内存里，消耗gas少

1，storage：合约里的状态变量默认都是storage，存储在链上

2，memory：函数里的参数和临时变量一般用memory，存储在内存中，不上链

3，calldata：和memory类似，存储在内存中，不上链。与memory的不同点在于calldata变量不能修改（immutable），一般用于函数的参数

### 作用域

> 变量的作用域：Solidity中变量按作用域划分有三种，分别是状态变量（state variable），局部变量（local variable）和全局变量(global variable)

1，状态变量：状态变量是数据存储在链上的变量，所有合约内函数都可以访问 ，gas消耗高。状态变量在合约内、函数外声明。可以在函数里更改状态变量的值

2，局部变量：局部变量是仅在函数执行过程中有效的变量，函数退出后，变量无效。局部变量的数据应该存储在内存里不上链。局部变量在函数内声明

3，全局变量：全局变量是全局范围工作的变量，都是solidity预留关键字。他们可以在函数内不声明直接使用（类似于 msg.sender,block.number）

### 作用域类型

> 作用域类型分为状态变量的作用域类型和函数的作用域类型

**状态变量**

- Public – 公共状态变量可以在内部访问，也可以通过消息访问。对于公共状态变量，将生成一个自动getter函数
- Internal – 内部状态变量只能从当前合约或其派生合约内访问
- Private – 私有状态变量只能从当前合约内部访问，派生合约内不能访问

**函数**

- external – 外部函数是合约接口的一部分，这意味着可以从其他合约或通过事务调用它们。但是内部无法调用

- public – 外部调用和内部都可以调用

- internal – 只能从当前合约或从当前合约派生的合约中访问，外部无法访问它们，由于它们没有通过合约的 ABI 向外部公开，所以它们可以接受内部类型的参数，比如映射或存储引用

- private – 私有函数类似于内部函数，但它们在派生合约中不可见

### 视图函数与存函数

> 在solidity中，函数可以被标记为view或者pure，标记成这两种状态后可以减少gas开销

- **view**: 表示该函数只访问外部状态，但不修改外部状态
- **pure**: 表示该函数既不访问外部状态，也不修改外部状态

## 部署合约

1. 要部署写好的智能合约，第一步我们先要编写部署脚本

- `/migrations/1_deploy.js`(注意: 部署脚本必须以数字开头，否则无法识别)

```js
// 使用artifacts.require导入写好的智能合约，不需要写路径，框架帮我们做了这件事情
const Contacts = artifacts.require("StudentStorage.sol")
// const Contacts1 = artifacts.require("other.sol")

// 部署脚本中必须导出一个函数
module.exports = function(deployer){
    deployer.deploy(Contacts)
    // 如果有多个智能合约，可以以同样的方式导入并部署
    // deployer.deploy(Contacts1)
}
```

2. 写好脚本之后就可以在truffle-config.js文件所在目录下执行以下指令：

```shell
$ truffle migrate
```

这条指令其实会在部署前先对我们写好的智能合约进行编译，相当于执行了:

```shell
$ truffle compiler
```

编译指令会把我们的智能合约编译为一个json文件，里面在abi就是通过web3js调用智能合约的关键

编译完成后，该指令才会继续执行部署，在部署过程中会消耗gas，由于我们没有配置from，因此它会默认从第一个账户中扣除

## 调试合约

首先我们要可以通过如下指令打开调试控制台(调试之前，必须保证我们的智能合约已经部署到了本地区块链中，合约修改之后，也得重新上链):

```shell 
$ truffle console
```

然后就可以开始调试我们写好的智能合约了，首先我们要创建一个合约对象:

```js
const obj = await StudentStorage.deployed()
```

创建好合约对象之后，就可以开始进行操作了:

```js
obj.address // 获取合约地址

// 该方法调用完成后会输出一个对象，这是由于这个方法要在链上存储一些数据，所以返回了一些相关信息，包括所消耗的gas值
obj.setData('ilmango', 18)

obj.getData()
// 可以看到返回的数字是一个bignumber对象
Result {
    '0': 'ilmango',
    '1': BN {
        negative: 0,
        words: [ 100, <1 empty item> ],
        length: 1,
        red: null
    }
}

// 这两个表达式都会返回一个函数，因为公共状态变量会被自动生成为getter，这是为了防止别人修改我们的链上数据，不停的消耗gas值
obj.name
obj.age
// 因此如果想要获取到name和age，要通过如下方式获取
obj.name()
obj.age()
```

## 测试合约

> 使用truffle脚本对合约功能进行测试

1. 首先要编写测试脚本，脚本为纯js文件

- /scripts/test.js

```js
const StudentStorage = artifacts.require('StudentStorage')

module.exports = async function(callabck){
    const studentStorage = await StudentStorage.deployed()

    await studentStorage.setData("kerwin",100)
    console.log(await studentStorage.getData())
    console.log(await studentStorage.name())
    console.log(await studentStorage.age())
    callabck() //必须写，否则不结束
 }
```

2. 运行测试

```shell
$ truffle exec ./scripts/test.js
```

## 打通web3JS与truffle

### 合约部分

- `/contracts/StudentListStorage.sol`

```solidity
// SPDX-License-Identifier: GPL-3.0 
// 源码遵循协议， MIT...
pragma solidity >=0.4.16 <0.9.0;

contract StudentListStorage{
    //结构体
    struct Student {
        uint id;
        string name;
        uint age;
        address account; // address也是一种数据类型，表示用户地址
    }
    // 动态数组(没有指定数组长度，如果指定了长度，则表示事先确定了内存空间，不可扩展)
    Student[] public StudentList; //自动生成getter()

    function addList(string memory _name,uint _age) public returns (uint){
        uint count = StudentList.length;
        uint index = count + 1;
        StudentList.push(Student(index,_name,_age,msg.sender)); // 这是的msg.sender是一个全局变量，表示该智能合约的调用者
        return StudentList.length;
    }

    function getList() public view returns (Student[] memory) {
       Student[] memory list = StudentList;
       return list;
    }
}
```

- `/migrations/2_deploy.js`(所有以数字开头的脚本都会被顺序执行)

```js
const Contacts = artifacts.require("StudentListStorage.sol")

module.exports = function(deployer){
    deployer.deploy(Contacts)
}
```

- `/scripts/test2.js`

```js
const Contacts = artifacts.require("StudentListStorage.sol")

module.exports =async  function(callback){
    const studentStorage = await Contacts.deployed()

    await studentStorage.addList("kerwin",100)
    let res =await studentStorage.getList()
    console.log(res)
    console.log(await studentStorage.StudentList(1)) // 这是必须传一个参数，因为这个getter是一个数组，要传入对应要访问的下标
    
    callback() // callback代表脚本执行完毕
}
```

初次部署：`truffle migrate`

重新部署：`truffle migrate --reset`

除非特别指定，`truffle migrate`命令将从最后完成的迁移脚本开始运行，就比如说上一次部署了前缀1的脚本，那么下一次部署时就会把这个脚本路过，不会重新部署。而reset参数就是表示从头开始运行所有的迁移脚本，而不是从最后完成的开始

> **注意：**以太坊智能合约的设计是部署后不允许修改的(上链之后即不可修改)，所以智能合约修改后重新部署数据便会丢失(相当于部署了一个新的智能合约)，但是难免有业务需求的变更，这时候便出现了可升级式智能合约，其原理是将数据合约和业务合约进行分离，用业务合约去调用数据合约，这样当需求变更时，只需要重新部署业务合约即可，而在数据合约里存储的数据则不会丢失。目前智能合约的数据和业务分离有两种方式，第一种是通过合约的地址进行调用，另一种是通过一个Proxy代理合约进行代理调用

### 前端部分

```html
<input type="text" id="myname">
<input type="number" id="myage">
<button id="add">add</button>
<ul id="list"></ul>

<script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
<script type="module">
  // 先使用web3js连接区块链虚拟机
  var web3 = new Web3(Web3.givenProvider || "http://localhost:8545");
  // 通过钱包进行授权
  let account = await web3.eth.requestAccounts()
  // 授权用户的地址
  console.log(account)
  // 连接智能合约程序
  // 这里的abi和contract address都可以在编译后的合约json文件中找到
  const studentStorage = await new web3.eth.Contract(abi, contract address);
  console.log(studentStorage)

  add.onclick = async function () {
    // 与在truffle调试环境中不同，在前端环境中获取到合约对象后，想要调用其中的方法，要在method属性中去找
    // 注意，涉及到上链操作时一定要跟上send方法，并且要写上来源用户地址，因为这个操作会消耗gas，所以会触发metamask转账
    await studentStorage.methods.addList(myname.value, myage.value).send({
      from: account[0]
    }) 
    //添加完后，获取列表
    getList()
  }

  async function getList() {
    // 对于不上链，只是读取链上数据的操作，要加上call方法
    let res = await studentStorage.methods.getList().call()
    list.innerHTML = res.map(item => `
      <li>${item.id}--${item.name}---${item.age}</li>
    `).join("")
  }
</script>
```

## 交易所实例

### 创建代币

> **什么叫做代币？**代币可以在以太坊中表示任何东西：
>
> - 在线平台中的信誉积分
> - 金融资产类似于公司股份的资产
> - 像美元一样的法定货币
> - 一盎司黄金
> - 及更多...

> **ERC-20**就是一套**基于以太坊网络的标准代币发行协议**。有了ERC-20，**开发者们得以高效、可靠、低成本地创造专属自己项目的代币；**我们甚至可以将ERC-20视为以太坊网络为早期区块链世界做出的最重要贡献，ERC-20F协议地址：https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
>
> ERC-20协议的功能示例包括：
>
> - 将代币从一个帐户转到另一个帐户
> - 获取帐户的当前代币余额
> - 获取网络上可用代币的总供应量
> - 批准一个帐户中一定的代币金额由第三方帐户使用

- `/contract/IlmangoToken.sol`

```solidity
// SPDX-License-Identifier: GPL-3.0
// 源码遵循协议， MIT...
pragma solidity >=0.4.16 <0.9.0; //限定solidity编译器版本
// openzeppelin-solidity是一个用于solidity中的工具库，可以通过npm install openzeppelin-solidity来安装
import "openzeppelin-solidity/contracts/utils/math/SafeMath.sol"; 

contract IlmangoToken {  
    // 通过这种方式可以给uint256后面使用 sub ,add方法
    using SafeMath for uint256;
    // 根据ERC-20规范定义一些东西
    // 定义币种名称
    string public name = "IlmangoToken";
    // 定义币种
    string public symbol = "IMT";
    // 定义最小单位 1IlmangoToken = 10 ** decimals
    uint256 public decimals = 18; 
    // 发行代币的总额
    uint256 public totalSupply;
    // mapping 定义映射用户地址与余额
    mapping(address => uint256) public balanceOf;
    // 该constructor在部署时运行一次
    constructor(){
        // 在部署时定义代币发行的总额
        totalSupply = 1000000 * (10 ** decimals);
        // 把发行的所有代币都存放进部署合约的人的账户中，在ganache测试环境中，部署人默认为第一个账户
        balanceOf[msg.sender] = totalSupply;
    }
    // 定义转账事件，这个事件必须在转账完成后触发一次，以保证转账信息被记录在区块链中(防止他人篡改)
    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    // 定义授权事件，这个事件必须在授权交易所后执行一下，以保证授权信息被记录在区块链中
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);
    // 转账方法
    function transfer(address _to, uint256 _value) public returns (bool success){
        // require类似于if，address(0)可以看做空地址
        // 使用require有一个好处，就是在条件不成立的时候他不会消耗我们的gas值
        require(_to!=address(0));
        _tranfer(msg.sender, _to, _value);
        return true;
    }
    // 这是一个internal函数，只能在合约内调用
    function _tranfer(address _from,address _to, uint256 _value) internal {
        require(balanceOf[_from]>=_value);
        //从哪个账号发起的调用者
        balanceOf[_from] = balanceOf[_from].sub(_value);
        balanceOf[_to] = balanceOf[_to].add(_value);
        //触发事件
        emit Transfer(_from,_to,_value);
    }
    // 授权方法
    /*
        {
            "kerwin":{
                A:100, // kerwin授权给A交易所100IMT
                B:200
            },
            "xiaoming":{
                A:200,
                B:100
            }
        }
    */
    function approve(address _spender, uint256 _value) public returns (bool success){
        // msg.sender  当前网页登录得账号
        // _spender 第三方得交易所得账号地址
        // _value 授权的金额
        require(_spender!=address(0));
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender,_spender,_value);
        return true;
    }
    // 这个方法是给被授权的交易用来转账的方法
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success){
        // _from 某个放款账号
        // _to 收款账户
        // msg.sender 交易所账户地址
        require(balanceOf[_from]>=_value);
        require(allowance[_from][msg.sender]>=_value);
        allowance[_from][msg.sender] = allowance[_from][msg.sender].sub(_value);
        _tranfer(_from, _to, _value);
        return true;
    }
}
```

- `/migrations/3_deploy.js`

```js
const Contacts = artifacts.require("IlmangoToken.sol")

module.exports = function(deployer){
    deployer.deploy(Contacts)
}
```

- `/scripts/test-tranfer.js`

```js
const IlmangoToken = artifacts.require("IlmangoToken.sol")

const fromWei = (bn)=>{
    // 在脚本中可以直接获取到web3
    return web3.utils.fromWei(bn,"ether");
}
const toWei = (number)=>{
    return web3.utils.toWei(number.toString(),"ether");
}

module.exports = async function (callback){
    const ilmangoToken =await IlmangoToken.deployed()
    // 虽然我们写的balanceOf是一个mapping，但是他会被映射为一个getter方法
    let res1 = await ilmangoToken.balanceOf("0x2c2C8F5B4041907c7BFBD3Af2ffF6a832FA019ec");
    console.log("第一个账号的余额:",fromWei(res1))
    
    await ilmangoToken.transfer("0xFe6023eEF9bb6dA8e5F33A073eddE35b02F1C0F0",toWei(10000),{
        from:"0x2c2C8F5B4041907c7BFBD3Af2ffF6a832FA019ec"
    })
    let res2 = await ilmangoToken.balanceOf("0x2c2C8F5B4041907c7BFBD3Af2ffF6a832FA019ec");
    console.log("第一个账号的余额:",fromWei(res2))
    
    let res3 = await ilmangoToken.balanceOf("0xFe6023eEF9bb6dA8e5F33A073eddE35b02F1C0F0");
    console.log("第二个账号的余额:",fromWei(res3))
    
    callback()
}
```

### metamask添加代币

> 在我们使用自己的助记词登录metamask后，就可以打开它，点击添加资产来添加代币：
>
> - 在点击添加资产后，metamask会让我们输入代币合约地址
> - 输入代币合约地址之后，它会自动识别symbol和decimals
> - 点击添加后，如果这个智能合约符合ERC-20规范，那metamask就可以自动通过banlanceOf读取我们当前用户地址的余额

## 交易所-存款取款

- `/contracts/Exchange.sol`

```solidity
// SPDX-License-Identifier: GPL-3.0
// 源码遵循协议， MIT...
pragma solidity >=0.4.16 <0.9.0; //限定solidity编译器版本
import "openzeppelin-solidity/contracts/utils/math/SafeMath.sol"; 
// 导入代币
import "./IlmangoToken.sol";

contract Exchange { 
    using SafeMath for uint256; //为了uint256后面使用 sub ,add方法，，
    // 收费账户地址(用来收入小费)
    address public feeAccount ;
    // 费率(收入小费的比率)
    uint256 public feePercent;
    // 把以太币地址保存为常量
    address constant ETHER = '0x0000000000000000000000000000000000000000';
    // 保存交易所的资产明细
    // {
    //     '币种地址一': {
    //         'kerwin': 100,
    //         'ilmango': 100
    //     },
    //     '币种地址二': {
    //         'kerwin': 100,
    //         'ilmango': 100
    //     },
    // }
    mapping(address=> mapping(address=>uint256)) public tokens;
    
    // 在部署合约时定义收费地址和费率
    constructor(address _feeAccount,uint256 _feePercent){
        feeAccount = _feeAccount;
        feePercent = _feePercent;
    } 
    
    // 定义收款事件
    event Deposit(address token,address user,uint256 amount, uint256 balance);
    // 定义取款事件
    event WithDraw(address token,address user,uint256 amount, uint256 balance);
    
    // 存以太币，由于以太币比较特殊，它需要被定义为payable，定义为payable之后就代表这是一个充值方法
    // 因为存代币只是修改token，存以太币是要获取钱包授权并且扣款的，并且被扣的款是会被存在当前合约地址上面的
    // 调用它时，就会把以太币充值到当前合约地址(如在是在web端通过web3调用该方法，它是会调起metamask要求充值的)
    function depositEther() payable public{
        // msg.sender 表示发送者
        // msg.value 表示发送数值，这两个值都由send方法定义
        tokens[ETHER][msg.sender] = tokens[ETHER][msg.sender].add(msg.value);
        emit Deposit(ETHER, msg.sender, msg.value, tokens[ETHER][msg.sender]);
    }
    // 存其他货币(存币都需要获取钱包的授权)
    function depositToken(address _token,uint256 _amount) public{
    	// msg.sender 表示发送者，这个值由send方法定义
        require(_token!=ETHER);
        // 这是我们是要将用户授权给交易所的代币划到交易的账户中(tokens)，这也就是说需要调用交易所的transferFrom方法
        // 那么首先要获取到代币的合约实例，我们可以通过导入IlmangoToken，然后把它在区块链中的合约地址传给它，即可获取到合约实例
        // 当前存币方法肯定是由用户来调用的，因此msg.sender是用户，要转给交易所address(this)
        // 而在IlmangoToken的transferFrom中，只要用户授权给了交易所代币，那么交易所就是可以完成转账的
        // 这是使用了require来判断transferFrom方法是否返回true，只有转账成功才能进行下一步
        require(IlmangoToken(_token).transferFrom(msg.sender,address(this),_amount));
        tokens[_token][msg.sender] = tokens[_token][msg.sender].add(_amount);
        // 触发存储事件
        emit Deposit(_token, msg.sender, _amount, tokens[_token][msg.sender]);
    }
    
    // 提取以太币
    function withdrawEther(uint256 _amount) public{
        // 提款时首先要确定该用户的以太币足够
        require(tokens[ETHER][msg.sender] >= _amount);
        // 减去用户的相应数量以太币
        tokens[ETHER][msg.sender] = tokens[ETHER][msg.sender].sub(_amount);
        // 通过payable方法从合约地址上转出相应数量的以太币给用户
        payable(msg.sender).transfer(_amount);
        emit WithDraw(ETHER,msg.sender,_amount ,tokens[ETHER][msg.sender]);
    }
    //提取IMT
    function withdrawToken(address _token,uint256 _amount) public{
        require(_token!=ETHER);
        // 首先要确定用户的IMT足够
        require(tokens[_token][msg.sender] >= _amount);
        // 在用户的账户中减去相应数量的IMT
        tokens[_token][msg.sender] = tokens[_token][msg.sender].sub(_amount);
        // 通过代币合约给用户转IMT，这是第一个参数就是to，而from则是transfer的发起者，也就是合约本身
        require(IlmangoToken(_token).transfer(msg.sender, _amount));
        emit WithDraw(_token,msg.sender,_amount ,tokens[_token][msg.sender]);
    }
}
```

- `/migrations/3_deploy.js`

```js
const Contacts = artifacts.require("IlmangoToken.sol")
const Exchange = artifacts.require("Exchange.sol")

module.exports = async function(deployer){
    // 获取当前区块链中的所有账户地址
    // 在布署脚本中是可以获取到web3实例的
    const accounts= await web3.eth.getAccounts()
    // 部署代币合约
    await deployer.deploy(Contacts)
    // 部署交易所合约，并且传入它需要的两个参数(收款地址和费率)
    // 这里的accounts[0]就是区块链中的第一个用户地址，在truffle测试环境中，gas就是默认在它身上扣，现在收取小费的地址也设为它
    await deployer.deploy(Exchange,accounts[0],10);
}
```

- `/scripts/test-deposit.js`（存币）

```js
const IlmangoToken = artifacts.require("IlmangoToken.sol")
const Exchange = artifacts.require("Exchange.sol")
const ETHER_ADDRESS = '0x0000000000000000000000000000000000000000' // 0x 后面 40 个 0

const fromWei = (bn)=>{
    return web3.utils.fromWei(bn,"ether");
}
const toWei = (number)=>{
    return web3.utils.toWei(number.toString(),"ether");
}

module.exports = async function (callback){
    const ilmangoToken =await IlmangoToken.deployed()
    const exchange =await Exchange.deployed()
    // 该方法用于获取当前区块链中的所有地址，由于是truffle环境，所以只有初始化时创建的10个地址
    const accounts= await web3.eth.getAccounts()
    // 存储ether，由于这是在truffle环境中，因此参数直接写在depositEther方法中
    // 如果在前端通过web3来调用就得写成下面这样，也就是说需要通过metamask的授权
    // exchange.depositEther().send({
    //     from:accounts[0],
    //     value:toWei(10)
    // })
    await exchange.depositEther({
        from:accounts[0],
        value:toWei(10)
    })
    // 获取accounts[0]在交易所中ether的余额
    let res1 = await exchange.tokens(ETHER_ADDRESS,accounts[0])
    console.log(fromWei(res1))
    
    // 如果要转代币，我们首先要授权交易所，然后再存储到交易所中
    // 在前端通过web3调用时都得使用send方法，然后通过metamask的授权
    await ilmangoToken.approve(exchange.address,toWei(100000),{
        from:accounts[0]
    }) 
    await exchange.depositToken(ilmangoToken.address,toWei(100000),{
        from:accounts[0]
    })
    let res2 = await exchange.tokens(ilmangoToken.address,accounts[0]) // IlmangoToken.address代表IlmangoToken的合约地址
    console.log(fromWei(res2))

    callback()
}
```

- `/scripts/test-withdraw.js`（取币）

```js
const IlmangoToken = artifacts.require("IlmangoToken.sol")
const Exchange = artifacts.require("Exchange.sol")
const ETHER_ADDRESS = '0x0000000000000000000000000000000000000000' // 0x 后面 40 个 0

const fromWei = (bn)=>{
    return web3.utils.fromWei(bn,"ether");
}

const toWei = (number)=>{
    return web3.utils.toWei(number.toString(),"ether");
}

module.exports = async function (callback){
    const ilmangoToken =await IlmangoToken.deployed()
    const exchange =await Exchange.deployed()
    const accounts= await web3.eth.getAccounts()

    // 从交易所中取出5ETH
    await exchange.withdrawEther(toWei(5),{
        from:accounts[0]
    })
    // 查看余额
    let res = await exchange.tokens(ETHER_ADDRESS,accounts[0])
    console.log(fromWei(res))

    // 从交易所中提取50000IMT
    await exchange.withdrawToken(ilmangoToken.address,toWei(50000),{
        from:accounts[0]
    })
    // 查看余额
    let res = await exchange.tokens(ilmangoToken.address,accounts[0])
    console.log(fromWei(res))

    callback()
}
```

## 交易所-流动池

- `/contracts/Exchange.sol`

```solidity
// SPDX-License-Identifier: GPL-3.0
// 源码遵循协议， MIT...
pragma solidity >=0.4.16 <0.9.0; //限定solidity编译器版本
import "openzeppelin-solidity/contracts/utils/math/SafeMath.sol"; 
// 导入代币
import "./IlmangoToken.sol";

contract Exchange { 
    using SafeMath for uint256; //为了uint256后面使用 sub ,add方法，，
    // 收费账户地址(用来收入小费)
    address public feeAccount ;
    // 费率(收入小费的比率)
    uint256 public feePercent;
    // 把以太币地址保存为常量
    address constant ETHER = '0x0000000000000000000000000000000000000000';
    // 保存交易所的资产明细
    // {
    //     '币种地址一': {
    //         'kerwin': 100,
    //         'ilmango': 100
    //     },
    //     '币种地址二': {
    //         'kerwin': 100,
    //         'ilmango': 100
    //     },
    // }
    mapping(address=> mapping(address=>uint256)) public tokens;
    //订单结构体，例如：100IMT ==> 1ether
    struct _Order {
        uint256 id;
        address user;
        address tokenGet; // 给什么货币(IMT)
        uint256 amountGet; // 给多少(100)
        address tokenGive; // 得到什么货币(ether)
        uint256 amountGive; // 得到多少(1)
        uint256 timestamp;
    }
    // 用于保存订单数据，同时也用于生成订单编号
    uint256 public orderCount;
    // 保存订单映射，key为订单编号，value为订单详情
    mapping(uint256 => _Order) public orders;
    // 被取消的订单并不会被删除，而是会保存入该映射中，订单编号 => true
    mapping(uint256 => bool) public orderCancel;
    // 完成的订单也不会被删除，而是会被保存入该映射中，订单编号 => true
    mapping(uint256 => bool) public orderFill;
    
    // 在部署合约时定义收费地址和费率
    constructor(address _feeAccount,uint256 _feePercent){
        feeAccount = _feeAccount;
        feePercent = _feePercent;
    } 
    
    // 定义收款事件
    event Deposit(address token,address user,uint256 amount, uint256 balance);
    // 定义取款事件
    event WithDraw(address token,address user,uint256 amount, uint256 balance);
    // 创建订单事件
    event Order(uint256 id, address user, address tokenGet, uint256 amountGet, address tokenGive, uint256 amountGive, uint256 timestamp);
    // 取消订单事件
    event Cancel(uint256 id, address user, address tokenGet, uint256 amountGet, address tokenGive, uint256 amountGive, uint256 timestamp);
    // 填充订单事件
    event Trade(uint256 id, address user, address tokenGet, uint256 amountGet, address tokenGive, uint256 amountGive, uint256 timestamp);
    
    // 存以太币，由于以太币比较特殊，它需要被定义为payable，定义为payable之后就代表这是一个充值方法
    // 因为存代币只是修改token，存以太币是要获取钱包授权并且扣款的，并且被扣的款是会被存在当前合约地址上面的
    // 调用它时，就会把以太币充值到当前合约地址(如在是在web端通过web3调用该方法，它是会调起metamask要求充值的)
    function depositEther() payable public{
        // msg.sender 表示发送者
        // msg.value 表示发送数值，这两个值都由send方法定义
        tokens[ETHER][msg.sender] = tokens[ETHER][msg.sender].add(msg.value);
        emit Deposit(ETHER, msg.sender, msg.value, tokens[ETHER][msg.sender]);
    }
    // 存其他货币(存币都需要获取钱包的授权)
    function depositToken(address _token,uint256 _amount) public{
    	// msg.sender 表示发送者，这个值由send方法定义
        require(_token!=ETHER);
        // 这是我们是要将用户授权给交易所的代币划到交易的账户中(tokens)，这也就是说需要调用交易所的transferFrom方法
        // 那么首先要获取到代币的合约实例，我们可以通过导入IlmangoToken，然后把它在区块链中的合约地址传给它，即可获取到合约实例
        // 当前存币方法肯定是由用户来调用的，因此msg.sender是用户，要转给交易所address(this)
        // 而在IlmangoToken的transferFrom中，只要用户授权给了交易所代币，那么交易所就是可以完成转账的
        // 这是使用了require来判断transferFrom方法是否返回true，只有转账成功才能进行下一步
        require(IlmangoToken(_token).transferFrom(msg.sender,address(this),_amount));
        tokens[_token][msg.sender] = tokens[_token][msg.sender].add(_amount);
        // 触发存储事件
        emit Deposit(_token, msg.sender, _amount, tokens[_token][msg.sender]);
    }
    
    // 提取以太币
    function withdrawEther(uint256 _amount) public{
        // 提款时首先要确定该用户的以太币足够
        require(tokens[ETHER][msg.sender] >= _amount);
        // 减去用户的相应数量以太币
        tokens[ETHER][msg.sender] = tokens[ETHER][msg.sender].sub(_amount);
        // 通过payable方法从合约地址上转出相应数量的以太币给用户
        payable(msg.sender).transfer(_amount);
        emit WithDraw(ETHER,msg.sender,_amount ,tokens[ETHER][msg.sender]);
    }
    //提取IMT
    function withdrawToken(address _token, uint256 _amount) public{
        require(_token!=ETHER);
        // 首先要确定用户的IMT足够
        require(tokens[_token][msg.sender] >= _amount);
        // 在用户的账户中减去相应数量的IMT
        tokens[_token][msg.sender] = tokens[_token][msg.sender].sub(_amount);
        // 通过代币合约给用户转IMT，这是第一个参数就是to，而from则是transfer的发起者，也就是合约本身
        require(IlmangoToken(_token).transfer(msg.sender, _amount));
        emit WithDraw(_token,msg.sender,_amount ,tokens[_token][msg.sender]);
    }
    
    //查余额
    function balanceOf(address _token, address _user) public view returns (uint256) {
        return tokens[_token][_user];
    }
    
    // 创建订单
    function makeOrder(address _tokenGet, uint256 _amountGet, address _tokenGive, uint256 _amountGive) public {
        // 要创建订单，首先要确定订单创建者确实有足够的商品
        require(balanceOf(_tokenGive, msg.sender) >= _amountGive, unicode"创建订单时余额不足");
        // 订单数量加一，这个订单数量会用作创建订单时的订单编号
        orderCount = orderCount.add(1);
        // block.timestamp是一个全局变量，表示当前时间戳
        orders[orderCount] = _Order(orderCount,msg.sender,_tokenGet,_amountGet,_tokenGive,_amountGive,block.timestamp);
        //发出订单
        emit Order(orderCount,msg.sender,_tokenGet,_amountGet,_tokenGive,_amountGive,block.timestamp);
    }
    // 取消订单
    function cancelOrder(uint256 _id) public {
        // 通过映射调取到要取消的订单
        _Order memory myorder = orders[_id];
        // 确认它是我们要取消的订单
        require(myorder.id == _id);
        // 将其放入取消列表中
        orderCancel[_id] = true;
        // 触发取消事件
        emit Cancel(myorder.id,msg.sender,myorder.tokenGet,myorder.amountGet,myorder.tokenGive,myorder.amountGive,block.timestamp);
    }
    // 完成订单
    function fillOrder(uint256 _id) public {
        // 调取到要完成的订单，并确认无误
        _Order memory myorder = orders[_id];
        require(myorder.id == _id);
        /*
            订单创建者：ilmango 
            订单完成者：msg.sender 
            订单：100IMT(筹码) ==> 1ether(商品)

            ilmango, 少了1ether
            ilmango, 多了100KWT

            msg.sender, 多了1ether
            msg.sender, 少了100KWT
        */
        // 计算小费，feePercent是在合约部署是传入的
        uint256 feeAmout = myorder.amountGet.mul(feePercent).div(100);
        // 确定订单创建者确实有足够的商品
        require(balanceOf(myorder.tokenGive, myorder.user) >= myorder.amountGive,unicode"创建订单的用户的余额不足");
        // 确定订单填充者确实有足够的钱来买
        require(balanceOf(myorder.tokenGet, msg.sender) >=myorder.amountGet.add(feeAmout),unicode"填充订单的用户的余额不足");
        
        // 填充者扣除get + 小费
        tokens[myorder.tokenGet][msg.sender] = tokens[myorder.tokenGet][msg.sender].sub(myorder.amountGet.add(feeAmout));
        // 收取小费到feeAccount
        tokens[myorder.tokenGet][feeAccount] = tokens[myorder.tokenGet][feeAccount].add(feeAmout);
        // 创建者添加get
        tokens[myorder.tokenGet][myorder.user] = tokens[myorder.tokenGet][myorder.user].add(myorder.amountGet);
        
        // 填充者增加give
        tokens[myorder.tokenGive][msg.sender] = tokens[myorder.tokenGive][msg.sender].add(myorder.amountGive);
        // 创建者减少give
        tokens[myorder.tokenGive][myorder.user] = tokens[myorder.tokenGive][myorder.user].sub(myorder.amountGive);
        
        // 将订单增加到fill列表中
        orderFill[_id] = true;
        // 触发交易事件，注意这里交易事件的主体是myorder.user(创建者)，而不是msg.sender
        emit Trade(myorder.id,myorder.user,myorder.tokenGet,myorder.amountGet,myorder.tokenGive,myorder.amountGive,block.timestamp);
    }
}
```

- `/scripts/test-order.js`

```js
// 导入代币和交易所
const IlmangoToken = artifacts.require("IlmangoToken.sol")
const Exchange = artifacts.require("Exchange.sol")
// 定义以太币的合约地址
const ETHER_ADDRESS = '0x0000000000000000000000000000000000000000' // 0x 后面 40 个 0

const fromWei = (bn) => {
    return web3.utils.fromWei(bn, "ether");
}
const toWei = (number) => {
    return web3.utils.toWei(number.toString(), "ether");
}
const wait = (seconds) => {
    const milliseconds = seconds * 1000
    return new Promise((resolve) => setTimeout(resolve, milliseconds))
}

module.exports = async function (callback) {
    // 所有的代码都放进try...catch中，因为我们下面的await只能在resolve后继续执行
    try {
        const ilmangoToken = await IlmangoToken.deployed()
        const exchange = await Exchange.deployed()
        const accounts = await web3.eth.getAccounts()

        // account0 ----> account1 10wIMT
        await ilmangoToken.transfer(accounts[1], toWei(100000), {
            from: accounts[0]
        })
        
        // account0 ----> 交易所 10ether
        await exchange.depositEther({
            from: accounts[0],
            value: toWei(100)
        })
        let res1 = await exchange.tokens(ETHER_ADDRESS, accounts[0])
        console.log("account[0]-在交易所的以太币", fromWei(res1))

        // account0 ----> 交易所 10wIMT
        await ilmangoToken.approve(exchange.address, toWei(100000), {
            from: accounts[0]
        })
        await exchange.depositToken(ilmangoToken.address, toWei(100000), {
            from: accounts[0]
        })
        let res2 = await exchange.tokens(ilmangoToken.address, accounts[0])
        console.log("account[0]-在交易所的IMT", fromWei(res2))

        // account1 ----> 交易所 5ether
        await exchange.depositEther({
            from: accounts[1],
            value: toWei(50)
        })
        let res3 = await exchange.tokens(ETHER_ADDRESS, accounts[1])
        console.log("account[1]-在交易所的以太币", fromWei(res3))

        // account1 ----> 交易所 5wIMT
        await ilmangoToken.approve(exchange.address, toWei(50000), {
            from: accounts[1]
        })
        await exchange.depositToken(ilmangoToken.address, toWei(50000), {
            from: accounts[1]
        })
        let res4 = await exchange.tokens(ilmangoToken.address, accounts[1])
        console.log("account[1]-在交易所的IMT", fromWei(res4))

        let orderId = 0;
        let res;

        // 创建订单
        res = await exchange.makeOrder(ilmangoToken.address, toWei(1000), ETHER_ADDRESS, toWei(0.1), { from: accounts[0] });
        // res.logs[0].args是固定写法，它代表的就是我们在智能合约中创建订单后抛发的事件中所保存的参数
        orderId = res.logs[0].args.id
        console.log("创建一个订单")
        await wait(1)
        
        // 取消订单
        res = await exchange.makeOrder(ilmangoToken.address, toWei(2000), ETHER_ADDRESS, toWei(0.2), { from: accounts[0] });
        orderId = res.logs[0].args.id
        await exchange.cancelOrder(orderId, { from: accounts[0] })
        console.log("取消一个订单")
        await wait(1)
        
        // 完成订单
        res = await exchange.makeOrder(ilmangoToken.address, toWei(3000), ETHER_ADDRESS, toWei(0.3), { from: accounts[0] });
        orderId = res.logs[0].args.id
        await exchange.fillOrder(orderId, { from: accounts[1] })
        console.log("完成一个订单")

        // 查看结果，注意：交易所合约中的feeAccount是account[0]，所以account[1]完成订单时，他的小费实际是付给account[0]的
        console.log("account[0]-在交易所的IMT", fromWei(await exchange.tokens(ilmangoToken.address, accounts[0])))
        console.log("account[0]-在交易所的以太币", fromWei(await exchange.tokens(ETHER_ADDRESS, accounts[0])))

        console.log("account[1]-在交易所的IMT", fromWei(await exchange.tokens(ilmangoToken.address, accounts[1])))
        console.log("account[1]-在交易所的以太币", fromWei(await exchange.tokens(ETHER_ADDRESS, accounts[1])))
    } catch (error) {
        console.log(error)
    }
    callback()
}
```

