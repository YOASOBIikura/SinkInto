---
title: Provider-2
date: 2024-06-13 21:21:05
categories:
  - ethers
tags:
  - JsonRpcProvider
---

## JsonRpcProvider
主要用于与以太坊RPC节点以及第三方节点进行交互通信使用

### JsonRpcProvider相关API方法

`new ethers.providers.JsonRpcProvider(url) => JsonRpcProvider`
通过指定连接url获取连接的通信JsonRpcProvider

`jsonRpcProvider.getSigner(index) => JsonRpcSigner`
通过连接后的provider获取指定下标的account

`jsonRpcProvider.getUncheckedSigner(index) => JsonRpcUncheckedSigner`
获取没有私钥的的account，使用比较少

`jsonRpcProvider.listAccounts() => Array<string>`
返回provider中管理的所有account列表

`jsonRpcProvider.send(method, params) => any`
发送方法调用，通常发送原始消息

---

## JsonRpcSigner相关API方法

`signer.provider => JsonRpcProvider`
返回这个signer的provider

`signer.connectUnchecked() => JsonRpcUnckeckedSigner`
返回一个新的signer对象，在发送交易时不执行额外的检查

`signer.sendUncheckedTransaction(transaction) => DataHexString`
发送交易并且返回不为透明交易的hash

`signer.unlock(password) => boolean`
使用密码解除signer的锁定

---

注意还有一个相似名字的staticJsonRpcProvider，这个provider会在后台频繁的执行getNetwork的调用，用来确保用户没有修改网络配置。
这个provider经常在链网络不被修改时时候，效率较高。但通常情况下的DeFi应用是很难不换链操作的，所以实际使用依然较少。


### 相关APIprovider

`ethers.providers.EtherscanProvider(apikey)` 
通过指定的APIkey来创建一个新的EtherscanProvider连接到网络，EtherscanProvider是由各种Etherscan APIs的组合所支持。

`ethers.providers.InfuraProvider(apikey)`
通过指定的APIkey来创建InfuraProvider连接到网络，InfuraProvider 被流行的INFURA以太坊服务所支持。

`ethers.providers.AlchemyProvider(apikey)`
通过指定的APIkey来获取新的AlchemyProvider连接网络，AlchemyProvider 是被Alchemy所支持的。

`ethers.provider.CloudflareProvider()`
创建一个新的CloudflareProvider连接到主网，CloudflareProvider 是被 Cloudflare Ethereum Gateway所支持的。


