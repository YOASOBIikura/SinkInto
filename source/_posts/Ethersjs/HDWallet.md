---
title: HDWallet
date: 2024-06-30 10:52:45
categories:
  - ethers
tags:
  - HD Wallet
---

## 基本常量

`ethers.utils.defaultPath => string`
返回以太坊在HD钱包中的默认路径

`mnemonic.phrase => string`
返回助记词的助记短语

`mnemonic.path => string`
返回助记词的HD路径

`mnemonic.local => string`
返回助记词所使用的单词表语言

## 属性

`hdNode.privateKey => string`
返回HDnode的私钥

`hdNode.publicKey => string`
返回HDnode压缩后的公钥

`hdNode.fingerprint => string`
用来快速匹配父节点和子节点的索引

`hdNode.parentFingerprint => string`
返回父节点的Fingerprint

`hdNode.address => string`
返回HDnode的地址

`hdNode.mnemonic => 助记词`
返回HDnode的助记词

`hdNode.path => string`
返回HDnode的path

`hdNode.neuter() => HDNode`
返回新的HDnode的实例

`hdNode.derivePath(path) => HDNode`
返回新的HDnode，并通过派生的path找到子节点

`ethers.utils.mnemonicToSeed(phrase, password) => string`
将助记词转换位种子

`ethers.utils.mnemonicToEntropy(phrase, wordlist) => string`
将助记词转换成熵

`ethers.utils.isValidMnemonic(phrase, wordlist) => boolean`
判断是否是有效的助记词短语