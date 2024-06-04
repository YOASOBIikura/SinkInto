---
title: Provider
categories:
  - ethers
tags:
  - provider
abbrlink: 39264
date: 2024-06-04 21:04:25
---

## Provider简介

Provider是以太坊网络连接的抽象接口，所有需要跟以太坊网络和钱包的交互都需要provider建立连接的基础之上进行。并且它为标准以太坊节点功能提供简洁、一致的接口！

---

### 默认Provider

默认的Provider已经足够强大和安全，并且可以在生产环境中去使用。

通常可以使用ethers进行代用

`ethers.getDefaultProvider([network, [options]]) ==> Provider`

其中它需要网络参数（network），如果没有该参数则会默认设置为以太坊主网(mainmet)。该参数在更多情况下使用URL进行连接例如（http://localhost:8545）

而option则是一个参数对象，它有以下几个属性:

1. alchemy: Alchemy API Token
2. etherscan: Etherscan API Token
3. infura: INFURA Project ID 或 { projectId, projectSecret }
4. pocket: Pocket Network Application ID 或 { applicationId, applicationSecretKey }
5. quorum: 必须满足规定的后端同意数量 (默认: 2 for mainnet, 1 for testnets)

这些参数的作用是当每个链的RPC都会有各自的服务接口，ethers默认提供的API Key是所有使用ethers用户所共享的，这会导致当用户访问达到一定的限额之后会对默认的Key访问进行限制
从而到值DAPP的性能问题，并且有的RPC所提供的服务是付费的这就必须要填写相关的API Key才能于链上合约进行交互。所以在生产环境上尽量配置该API key!

---

## 网络

可以通过networksh参数来获取网络的详细信息，networksh的参数可以是网络名称、chainID等等

***ethers.providers.getNetwork(aNetworkSh) ==> Network
{
   chainId: 1,
   ensAddress: '0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e',
   name: 'homestead'
}***




