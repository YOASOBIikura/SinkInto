---
title: Provider-3
date: 2024-06-15 13:04:35
categories:
  - ethers
tags:
  - 其他Provider
---

## FallbackProvider

它是ethers中最高级的provider,通过配置参数接收多个provider作为后端，可以给每个provider配置优先级和权重，一次请求转发所有的provider,最终比对请求结果并返回。

`ethers.provider.FallbackProvider(providers, quorum) => FallbackProvider`
通过指定provider来设定多个请求后端，providers可以是Provider或FallbackProviderConfig数组，配置后优先级和权重默认为1

`provider.providerConfigs => FallbackProviderConfig`
描述后端Provider的配置列表

`provider.quorum => number`
在得到结果之前，后端响应的quorum必须达成一致。默认情况下，这个值是权重之和的一半。

---

## FallbackProviderConfig

`fallbackProviderConfig.provider => Provider`
返回配置的provider项

`fallbackProviderConfig.priority => number`
表示provider所使用的优先级，相同优先级则随机选择，优先级越低优先程度更高

`fallbackProviderConfig.stalltimeout => number`
返回超时时间，超时后则使用其他provider进行请求，并且该结果也会被加入到quorum中

`fallbackProviderConfig.weight => number`
返回这个provider的权重

---

## IpcProvider

IpcProvider 允许JSON-RPC API在文件系统的本地文件名上使用。 Geth, Parity和其他节点都会开放了这个功能

---

## UrlJsonRpcProvider

这个类打算作为子类而不是直接使用。只需要额外生成一个SON-RPCURL后，就能通过JsonRpcProvider创建一个Provider

---

## Web3Provider

Web3Provider为了方便一个基于web3.js应用程序迁移到ethers，主要是通过将现有的Web3-compatible (比如一个 Web3HttpProvider, Web3IpcProvider 或者 Web3WsProvider) 打包进ethers.js作为ethers.js Provider，并将相应的接口暴露出去，它可以和ethers.js库的其他部分一起使用

`ehters.providers.Web3Provider(external, network) => provider`
通常用于浏览器连接钱包创建provider

---

## WebSocketProvider

WebSocketProvider 连接到一个JSON-RPC websocket兼容的后端，它允许持久连接、多路复用请求和发布-子事件，以实现更即时的事件调度,但是WebSockets对服务器资源的占用会很大， 因为它们必须管理和维护每个客户端的状态。由于这个原因，许多服务也可能会为使用他们的WebSocket端点而收取额外的费用

---

## Types

`feeData.getPrice => BigNumber`
获取不支持EIP-1559的遗留交易的gas价格

`feeData.maxFeePerGas => BigNumber`
获取最近产出的区块的baseFee

`feeData.maxPriorityFeePerGas => BigNumber`
获取交易中建议的最大优先费率

`filter.address => address`
想要筛选的地址

`filter.topics => string<Data>`
想要筛选的主题

`filter.fromBlock => BlockTag`
用于搜索匹配条件的起始区块

`filter.toBlock => BlockTag`
用于搜索匹配条件的结束区块

`filter.blockHash => string<DataHexString>`
指定符合blockhash条件的日志

`log.blockNumnber => number`
包含该日志的交易的区块高度

`log.blockHash => string<DataHexString>`
包含该日志的区块hash

`log.removed => boolean`
在区块重组的过程中，某系交易可能会被孤立而没有被挖出，则值为true.如果后面被某区块挖出后该值为false

`log.transactionLogIndex => number`
获取日志在交易数据中的索引

`log.address => address`
生成该日志的合约地址

`log.data => string<DataHexString>`
获取日志数据

`log.topics => Array`
获取日志主题的列表

`log.transactionHash => string`
获取包含该日志的交易hash

`log.transactionIndex => number`
获取包含该日志交易的索引

`log.logIndex => number`
在整个区块中该条日志的索引

`transactionRequest.noce => number`
交易的noce值

`transactionRequest.data => DataHexString`
交易数据

`transactionRequest.value => Bignumber`
发送交易中的ETH

`transactionRequest.gasLimi => Bignumber`
这笔交易所允许的最大gas使用上限(如果没有显示的指定，ethers会通过estimateGas来确定该值)

`transactionRequest.gasPrice => Bignumber`
获取该交易的gas费率

`transactionRequest.maxFeePerGas => Bignumber`
指定将支付EIP-1559基础费用的最高gas费率

`transactionRequest.maxPriorityFeePerGas => Bignumber`
指定将支付EIP-1559基础费用的优先gas费率

`transactionRequest.chainId => Number`
获取已被授权的chain ID,有EIP-155指定

`transactionRequest.type => Number`
这笔交易envelope的EIP-2718类型，在网络上这个默认值是null

`transactionRequest.accessList => AccesssListish`
获取包含的AccessList,仅适用于EIP-2930和EIP-1559的交易

`transaction.confirmations => number`
该笔交易所在的区块被挖出时，所有已经挖出的区块数量

`transaction.raw => string`
序列化交易

`transaction.wait(confirms) => TransactionReceipt`
指定等待挖出交易的区块数量，一旦满足要求则返回Receipt

`receipt.logBloom => string`
返回该交易中所有日志包含的全部地址和主题

`receipt.confirmations  => number`
当该交易被挖出时的已被挖出的区块数量

`receipt.cumulativeGasUsed   => Bignumber`
截止到该交易为止所有交易使用的Gas总和

`receipt.status => boolean`
为1表示交易成功，为0则表示交易失败