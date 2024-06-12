---
title: Provider-1
categories:
  - ethers
tags:
  - provider
abbrlink: 60980
date: 2024-06-12 20:10:46
---

## Provider基本API使用

### 账户相关方法

`provider.getBalance(address, blockTag) => BigNumber`
最基本的API，可以查询链上指定高度地址的余额

`provider.getCode(addres, blockTag) => string<DataHexString>`
查询该地址指定区块高度的合约代码，如果该区块高度没有代码则返回'0x'

`provider.getStorageAt(address, pos, blockTag) => string<DataHexString>`
返回指定区块高度下，在address合约存储中pos位置存储插槽的值，基本很少用到

`provider.getTransactionCount(address, blockTag) => number`
返回在指定区块高度下，该账户的交易次数，并且这个次数是下一次网络交易的Noce

---

### 区块相关方法

`provider.getBlock(blockNum) => Block`
根据区块数来获取相关区块信息

`provider.getBlockWithTransactions(blockNum) => BlockWithTransactions`
根据指定的区块高度获取该区块所有交易信息对象集合

---

### 以太坊域名服务相关API

`provider.getResolver(name) => EnsResolver`
返回一个EnsResolver实例，可用于进一步查询ENS命名的实体，比较少用

`provider.lookupAddress(address) => string`
通过地址来反向查找服务域名，如果名称不存在或者无法匹配则返回null

`provider.resolveName(name) => string<address>`
查找一个域名的地址，如果该域名没有设置则返回null

---

## EnsResolver相关API

`resolver.getContentHash() => string`
返回任何存储的EIP-1577的内容hash，通常是IPFS地址

`resolver.getText(key) => string`
根据键值返回任何存储的EIP-634的文本实体

---

### Logs相关方法

`provider.getLogs(filter) => Array<log>`
根据过滤器返回log数组，太久的事件可能会被丢弃，而查询数据量过大的请求可能会被拒绝！

---

### Network状态相关API

`provider.getNetwork() => Network`
返回所连接网络(链)的相关信息

`provider.getBlockNumber() => number`
返回最近挖出的区块序号(区块高度)

`provider.getGasPrice() => Number`
返回当前交易中gasPrice预估值

`provider.getFeeData() => FeeData`
返回一笔交易中需要的交易费用

`provider.ready() => Network`
用于测试网络连接，通常用于测试等待网络连接等

---

### Transaction相关API

`provider.call(transaction, blockTag) => string<DataHexString>`
通过call方法执行交易，该交易不支付gas，不能改变链状态，通常用于合约的可读方法查询数据

`provider.estimateGas(transaction) => BigNumber`
返回提交交易所需的gas值，这个预估值通常可能并不准确

`provider.getTransaction(hash) => TransactionResponse`
返回该交易Hash的详细信息，如果交易未知则返回null

`provider.getTransactionReceipt(hash) => TransactionReceipt`
返回该交易hash的收据信息，如果交易还没被挖出则返回null

`provider.sendTransaction(transaction) => TransactionResponse`
向区块链中发起交易，交易必须经过签名，并且必须合法

`provider.waitForTransaction(hash) => TxReceipt`
该方法可设置阻塞和非阻塞，当交易被挖出后返回交易回执，否则返回null或者被阻塞一直等待

---

### Event Emitter方法

`provider.on(eventName, listener) => this`
对指定事件名添加监听器

`provider.once(eventName, listener) => this`
对指定事件添加一次性的监听器，监听完成后即被移除

`provider.emit(eventName, ...args) => boolean`
通知所有指定事件名的监听器，并把参数传递给他们

`provider.off(eventName, listener) => this`
移除指定事件的监听器，如果没有指定监听器则移除关于指定事件的所有监听器

`provider.removeAllListeners(eventName) => this`
移除所有指定事件的监听器，如果没有指定事件则移除所有监听器

`provider.listenerCount(eventName) => Number`
获取指定事件的监听器数量，没有指定事件则返回所有监听器的数量

`provider.listeners(evenName) => Array<Listener>`
返回指定事件的监听器集合

`provider.isProvider(object) => boolean`
判断指定对象是一个provider时返回true
