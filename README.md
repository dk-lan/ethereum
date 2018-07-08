# 智能合约开发环境搭建
## Solidity 安装
## geth 安装
#### Mac
```javascript
brew tap ethereum/ethereum
brew install ethereum
```

#### 启动环境
geth是一个以太坊客户端，现在利用geth启动一个以太坊（开发者）网络节点。
- `geth --datadir testNet --dev console 2>> test.log` 执行命名后，会进入geth控制台
    - `datadir testNet` 参数是指定数据存放文件夹
    - `dev` 启用开发者网络（模式），开发者网络会使用POA共识，默认预分配一个开发者账户并且会自动开启挖矿。
    - `datadir` 后面的参数是区块数据及秘钥存放目录。第一次输入命令后，它会放在当前目录下新建一个testNet目录来存放数据。
    - `console`进入控制台
    - `2>> test.log` 表示把控制台日志输出到test.log文件

#### 准备账户
部署智能合约需要一个外部账户，我们先来看看分配的开发者账户，在控制台使用以下命令查看账户：`eth.accounts`，回车后，返回一个账户数组，里面有一个默认账户，如：
`["0x14f68c062ef8a22f8214a7d1d65ec16bc7ec4bef"]`  
也可以使用`personal.listAccounts`查看账户  
再来看一下账户里的余额，使用一下命令 `eth.getBalance(eth.accounts[0])`，回车后可以看到大量的余额，如：1.15792089237316195423570985008687907853269984665640564039457584007913129639927e+77

开发者账户因余额太多，如果用这个账户来部署合约时会无法看到余额变化，为了更好的体验完整的过程，这里选择创建一个新的账户

#### 创建账户
`personal.newAccount("dktest")`创建一个新的账户，dktest 为新账户的密码，回车后，返回一个新账户
`eth.accounts` 可以看到账户列表。
`eth.getBalance(eth.accounts[1])`可以查看新账户的余额

#### 给新账户转钱
我们知道没有余额的账户是没法部署合约的，那我们就从默认账户转1以太币给新账户
- `tail -f test.log` 在另一个命令窗口打开日志
- `eth.accounts` 查看账户
- `eth.sendTransaction({from: '0x14f68c062ef8a22f8214a7d1d65ec16bc7ec4bef', to: '0xaf9e999b640e8cf57c94529909acceccdce0ff32', value: web3.toWei(1, "ether")})`
- 在打开的tail -f test.log日志终端里，可以同时看到挖矿记录再次查看新账户余额，可以新账户有1个以太币
- `eth.getBalance(eth.accounts[1])`能查看到账户余额

#### 解锁账户
在部署合约前需要先解锁账户（就像银行转账要输入密码一样），使用以下命令 `personal.unlockAccount(eth.accounts[1], 'dktest')`
解锁成功后，账户就准备完毕啦，接下来就是编写合约代码

## 编写合约代码
现在我们来开始编写第一个智能合约代码，solidity 代码如下
```javascript
pragma solidity ^0.4.18;
contract hello {
    string greeting;

    function hello(string _greeting) public {
        greeting = _greeting;
    }

    function say() constant public returns (string) {
        return greeting;
    }
}
```

简单解释下，我们定义了一个名为hello的合约，在合约初始化时保存了一个字符串（我们会传入hello world），每次调用say返回字符串。
把这段代码写(拷贝)到[Browser-Solidity](https://ethereum.github.io/browser-solidity/)，如果没有错误，先点击 `Start to compile`，然后点击 `Details` 获取部署代码，

在弹出的对话框中找到WEB3DEPLOY部分，点拷贝，粘贴到编辑器后，修改初始化字符串为 `var _greeting = "Hello World" ;`

## 部署合约
Browser-Solidity生成的代码，拷贝到编辑器里修改后的代码如下：
```javascript
var _greeting = "Hello World";
var helloContract = web3.eth.contract([{"constant":true,"inputs":[],"name":"say","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]);
var hello = helloContract.new(
   _greeting,
   {
     from: web3.eth.accounts[1], 
     data: '0x608060405234801561001057600080fd5b506040516102a83803806102a8833981018060405281019080805182019291905050508060009080519060200190610049929190610050565b50506100f5565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061009157805160ff19168380011785556100bf565b828001600101855582156100bf579182015b828111156100be5782518255916020019190600101906100a3565b5b5090506100cc91906100d0565b5090565b6100f291905b808211156100ee5760008160009055506001016100d6565b5090565b90565b6101a4806101046000396000f300608060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063954ab4b214610046575b600080fd5b34801561005257600080fd5b5061005b6100d6565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009b578082015181840152602081019050610080565b50505050905090810190601f1680156100c85780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b606060008054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561016e5780601f106101435761010080835404028352916020019161016e565b820191906000526020600020905b81548152906001019060200180831161015157829003601f168201915b50505050509050905600a165627a7a72305820f8c0d0473e49b7154342dea90dedd970a412bab12084939aa3655ebf2dfc43150029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```
第1行：修改字符串为Hello World
第2行：修改合约变量名
第3行：修改合约实例变量名，之后可以直接用实例调用函数。
第6行：修改部署账户为新账户索引，即使用新账户来部署合约。
第8行：准备付的gas费用，IDE已经帮我们预估好了。
第9行：设置部署回调函数。

拷贝会geth控制台里，回车后，看到输出如：Contract mined! address: 0x7176f1de1a4c097af4edbe7aa7ccc112edb41b60 transactionHash: 0x01cf704a2dc1bd07fa433b49124d1dd265265113c7c00cd65bbbef6cd756023c

说明合约已经部署成功。

在打开的tail -f test.log日志终端里，可以同时看到挖矿记录

现在我们查看下新账户的余额，余额比之前少了

在线 solidity 环境调用函数时，将无法改变from的地址。所以你只能扮演铸币者的角色，可以铸造货币并发送给其他人，而无法扮演其他人的角色。这点在线solidity环境将来会做改进。

## 运行合约
`hello.say()` 输出“Hello World”则说明我们第一个合约Hello World，成功运行了。。