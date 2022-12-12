# 使用元掩码开发以太坊 dApps 的常见问题

> 原文：<https://www.moesif.com/blog/blockchain/ethereum/Common-Problems-Developing-Ethereum-DApps-With-Metamask/>

在我写了一篇关于构建一个集成 Web3 监控的以太坊 DApp 的教程之后，我收到了许多公开和私下的问题。这篇文章收集了一些答案来帮助你开发以太坊 dApps。

## 未连接到网络

首先，了解将您的 DApp 连接到区块链网络的逻辑。示例 [getWeb3.js](https://github.com/Moesif/moesif-ethereum-js-example/blob/master/src/utils/getWeb3.js) 实现了连接到 Web3 网络的一种常见模式。

它检查是否有一个`web3`对象已经被注入到全局作用域中。如果没有，代码会尝试连接到 truffle 在本地运行的测试网络。

在生产中，您的 dApp 的许多用户可能正在使用 Metamask、Mist 或其他以太坊客户端，这些客户端会自动注入 web3 对象并连接到本地或远程以太坊节点。

因此，如果您已经运行了元掩码，元掩码就会将一个`web3`对象注入到全局作用域中。因为教程示例使用的是 truffle 的测试网络，所以现在您可能想关闭 Metamask 扩展。

## 让示例使用元掩码

有两件事你需要弄清楚:

*   您计划使用元掩码连接到哪个网络？如果是另一个测试网络还好。确保你有所有需要的数据。
*   您的合同是否部署到元掩码将连接的网络。

## 将合同部署到不同的网络

在教程示例中，合同被部署到 truffle 开发网络，该网络是在运行`truffle develop`时创建的。节点在`http://127.0.0.1:9545`可用。

如果您想要部署到另一个网络，例如`ganach`，那么您需要修改`truffle.js`文件。

```py
module.exports = {
  // See <http://truffleframework.com/docs/advanced/configuration>
  // to customize your Truffle configuration!

  networks: {
    ganache: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    },
    development: {
      host: "127.0.0.1",
      port: 9545,
      network_id: "*" // Match any network id
    }
  }
}; 
```

然后你需要跑

```py
truffle compile

truffle migrate --network ganache 
```

当然，用您选择的名称和设置替换 ganache 设置。

## 如何找到我刚刚部署的智能合约的地址

有几种方法。

首先，当您运行 migrate 命令来部署协定时，该地址会显示在命令行输出中。

```py
Running migration: 1_initial_migration.js
  Replacing Migrations...
  ... 0xbf9932f55ba158d95f4ed41632cf2558c9ce63a63591e6cf45a5b58db435a598
  Migrations: 0x8cdaf0cd259887258bc13a92c0a6da92698644c0
Saving successful migration to network...
  ... 0xd7bc86d31bee32fa3988f1c1eabce403a1b5d570340a3a9cdba53a472ee8c956
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Replacing SimpleStorage...
  ... 0xa19fe7f1c68b20bc7c58ad270cee58767248eef7fa7e58ad847855582f8f1212
  SimpleStorage: 0x345ca3e014aaf5dca488057592ee47305d9b3e10
Saving successful migration to network...
  ... 0xf36163615f41ef7ed8f4a8f192149a0bf633fe1a2398ce001bf44c43dc7bdda0
Saving artifacts... 
```

请注意:simple storage:0x 345 ca 3e 014 AAF 5 DCA 488057592 ee 47305 d9 B3 e 10

第二种方式:如果您在`SimpleStorage.json`打开 JSON 工件，您将看到关于它被部署到的地址和网络的数据。

```py
 "networks": {
    "4447": {
      "events": {},
      "links": {},
      "address": "0x345ca3e014aaf5dca488057592ee47305d9b3e10",
      "transactionHash": "0xa19fe7f1c68b20bc7c58ad270cee58767248eef7fa7e58ad847855582f8f1212"
    }
  }, 
```

第三种方式，在您的 dApp 代码中，如果您有一个已部署契约的实例，就可以获得地址。

```py
const simpleStorage = contract(SimpleStorageContract)
simpleStorage.setProvider(this.state.web3.currentProvider)

simpleStorage.deployed().then((instance) => {
// now you have the instance of the deployed contract, you can get the address.
  console.log(instance.address)
}); 
```

## 收件人地址不是合同地址

当您切换到不同的网络时，您需要将合同部署到正确的网络；但是，当您尝试再次运行 migrate 时，可能会出现如下错误。

```py
 Attempting to run transaction which calls a contract function, but recipient address 0x8cdaf0cd259887258bc13a92c0a6da92698644c0 is not a contract address 
```

要解决这个问题，删除`build/contracts`下的 JSON 文件，因为这些 JSON 文件有关于它被部署到哪里的元数据。

另一个解决方法是尝试`truffle migrate --reset`。

## 我发起了一项交易，但我没有看到任何变化

这可能发生在将网络切换到元掩码并连接之后。

对于每一个使用 ether 的事务，Metamask 将明确地请求用户允许该事务。有时权限弹出窗口是隐藏的。如果发生这种情况，请手动打开元掩码弹出窗口并授予事务权限。

在教程示例中，第一个事务试图设置存储，这会消耗汽油。如果您没有看到存储被设置为 5，请单击元掩码以查看它是否在启动第一个事务之前等待您的批准。

## 元掩码随机数问题:错误:tx 没有正确的随机数。

如果你使用[元掩码](https://metamask.io/)进行开发和测试，有时你会看到这样的错误:

```py
message:"Error: Error: [ethjs-rpc] rpc error with payload {"id":8318498190067,"jsonrpc":"2.0","params":["0xf88914843b9aca0082f3ff94345ca3e014aaf5dca488057592ee47305d9b3e1080a460fe47b10000000000000000000000000000000000000000000000000000000000000019822d46a0a3ecf94d503161fbbf914c90c0b18fedf86a382b126314d71b5ec11bf0985b92a0468281983291f396cd02c80f4428898e51d258f57548ddf483da08962a02ad22"],"method":"eth_sendRawTransaction"} Error: Error: the tx doesn't have the correct nonce. account has nonce of: 4 tx has nonce of: 20 
```

这是在您重新启动测试网络或从另一个测试网络重新启动时造成的。元掩码中的缓存事务历史记录与网络历史记录不匹配。

要解决此问题，请打开 Metamask 中的“设置”选项卡，然后单击*重置网络的*。

## 我更换了 web3 提供者，但是我再也看不到 Moesif 捕获的事件了。

默认情况下， [Moesif 浏览器 JS SDK](https://www.moesif.com/docs/client-integration/browser-js/) 试图在`XMLHttpRequest`层捕获数据。如果检测到全局 web3 对象，如 Metamask 或 Mist 注入的 web3 对象，Moesif 将检测 web3 对象。

在一些高级场景中，您决定修改全局`web3`对象，比如创建一个新的 web3 对象或修改底层提供者。例如，你的 dApp 允许用户更换网络或提供商。

对于这些情况，您需要将新的 web3 对象传递给 Moesif，以确保它被正确地插装，这可以通过`useWeb3()`方法来完成。

```py
moesif.useWeb3(myNewWeb3); 
```

## 在 Moesif 中，我看到重复的事件

如前所述， [Moesif Browser JS SDK](https://www.moesif.com/docs/client-integration/browser-js/) 通过 XMLHttpRequest 和 web3 两种方式捕获数据。

如果您的底层提供者也使用像`HTTPProvider`这样的`XMLHttpRequest`，就会发生这种情况。这是可以的，也是意料之中的。

在`web3`层捕获的所有事件都有一个名为`_web3`的附加元数据字段，它存储有关使用哪个 web3 的信息。

**Moesif 是最先进的 API 分析平台，支持 REST、GraphQL、Web3 Json-RPC 等。成千上万的平台公司利用 Moesif 进行调试、监控和发现见解。**

[了解更多](https://www.moesif.com?utm_source=blog)