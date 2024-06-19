---
title: Contract-0
date: 2024-06-17 20:29:50
categories:
  - ethers
tags:
  - Contracts
---

## Contract
用来作为链上合约的抽象，可作为js与链上合约交互的实体，通过Contract发送交易或执行相关方法改变链上状态

`ethers.Contract(address, abi, SignerOrProvider) => Contract`
返回一个链上合约实例

`contract.attach(address) => Contract`
通过地址链接到指定的合约

`contract.connect(SignerOrProvider) => Contract`
通过指定的Signer或Provider的不同返回不同权限的Contract，通常指定Provider的实例只能读数据，而传入Signer则返回一个代表Signer的合约实例

`contract.deployed() => Contract`
等待合约部署成功，并返回合约实例

`contratc.isIndexed(value) => boolean`
用于检查指定的值在合约事件中是否被索引

---

## Events

`contract.queryFilter(event, [fromBlockTag, toBlockTag]) => Array`
返回指定区块中匹配的事件

`contract.listenerCount(event) => number`
返回订阅该event的监听器数量，如果没有提供参数则返回所有监听器的总数

`contract.listeners(event) => Array`
返回监听该事件的监听器列表

`contract.off(event, listener) => this`
取消监听器对指定事件的监听

`contract.on(event, listener) => this`
监听event事件

`contract.once(event, listener) => this`
对指定事件只监听一次

`contract.removeAllListeners(event) => this`
取消所有监听器对指定事件的监听，没有指定事件则取消对所有事件的监听

---


