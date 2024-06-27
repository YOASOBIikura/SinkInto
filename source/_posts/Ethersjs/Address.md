---
title: Address
date: 2024-06-25 21:31:08
categories:
  - ethers
tags:
  - Address
---

## Address
地址是一个20字节的混合大小写的十六进制字符串，通过checkSum Address模式，在给定地址的特殊位置指定大写或是小写来减少输入地址或剪切粘贴时带来的错误风险

`ethers.utils.getAddress(address) => Address`
通过指定的地址字符串返回一个校验和地址

`ethers.utils.getIcap(address) => IcapAddress`
返回一个ICAP address,该方法与getAddress方法具有相同的条件

`ethers.utils.isAddress(address) => boolean`
验证指定的地址是否有效

`ethers.utils.computeAddress(publickOrPrivateKey) => string`
通过公钥或是私钥计算相应的地址，公钥可以压缩也可以不用压缩

`ethers.utils.recoverAddress(digest, signature) => string`
通过ECDSA Public Key Recovery来确定摘要生成签名的公钥地址

`ethers.utils.getContractAddress(transaction) => Address`
返回交易部署合约的地址

`ethers.utils.getCreate2Address(from, salt, initCodeHash) => string`
返回给定CREATE2所生成合约的地址