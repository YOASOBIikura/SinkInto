---
title: Signers-0
date: 2024-06-16 10:29:29
categories:
  - ethers
tags:
  - Signers
---

## Signer
在ethers中Signer是以太坊账户的抽象，主要用来签名交易和信息，以方便后续将交易放松到以太坊网络
由于Signer是一个抽象类，它必须依托于一个具体的类对象如wallet,provider，JsonRpcSigner等进行获取或是实例化

`signer.connect(provider) => signer`
如果provider不支持该方法会返回错误，默认返回连接provider的账户

`signer.getAddress() => address`
返回该Signer的地址

`signer.isSigner(obj) => boolean`
判断当对象是一个signer返回true

`signer.getBalance(blockTag) => Bignumber`
查询指定区块中该账户的余额

`signer.getChainId() => Bignumber`
返回钱包连接网络的链ID

`signer.getGasPrice() => Bignumber`
返回当前的gas费率

`signer.getTransactionCount(blockTag) => number`
返回该账户已经发送的交易数量

`signrt.call(transactionRequest) => string`
进行交易发送，返回交易结果，此账户地址用作交易中from字段

`signer.estimateGas(transactionRequest) => Bignumber`
返回发送交易的预估gas值

`signer.resolveName(ensName) => address`
返回指定ensName相关联的地址

`signer.signMessage(message) => string`
返回签名信息之后信息用于验证，其中的message必须是utf8的字节数组形式

`signer.signTransaction(transactionRequest) => string`
返回对交易进行签名后的数据

`signer.sendTransaction(transactionRequest) => TransactionResponse`
发送交易，并返回交易结果

`signer._signTypedData(domain, types, vaule) => string`
采用EIP-712规范对类型数据结构作为领域签署类型化的数据记性签名

`signer.checkTransaction(transactionRequest) => transactionRquest`
确认交易请求并返回一个交易副本，通常包含sendTransaction所包含的所有属性

`signer.populateTransaction(transactionRequest) => transactinRequest`
于上诉的checkTransaction方法大致相同

---

## Wallet
Wallet继承了Signer，可以同过配置私钥来对交易和信息进行签名

`ethers.Wallet(privateKey, provider) => Wallet`
返回钱包实例

`ethers.Wallet.createRandom([options]) => Wallet`
返回一个带有私钥的新钱包，如果检测当前环境没有安全的熵源会报错，并且通过该方法创建的钱包将具有助记词

`ehters.Wallet.fromEncryptedJson(json,password, [progress]) => Wallet`
从一个加密的Json钱包中创建实例

`ehters.Wallet.fromMnemonic(mnemonic[path, [wordlist]]) => wallet`
通过助记词创建钱包实例

`wallet.encrypt(password,[option, [progress]]) => string`
通过password返回一个Json形式的加密钱包

---

## VoidSigner
一种简单的只读Signer，在交易中不能进行签名，只能对有限的只读操作有效

`ethers.VoidSigner(address, [provider]) => VoidSigner`
通过指定的地址创建一个只读Signer




