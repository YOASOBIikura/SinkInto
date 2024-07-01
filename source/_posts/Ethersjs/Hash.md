---
title: Hash
date: 2024-06-29 14:58:18
categories:
  - ethers
tags:
  - Hash
---

## Hash
ethers中有一些好用的哈希函数，他们在日常开发中是非常必要

`ethers.utils.id(text) => string`
通过以太坊的身份函数来计算文本的keccak256的哈希值

`ethers.utils.keccack(aBytesLike) => string`
返回指定bytes的keccak256的哈希值

`ethers.utils.ripemd160(aBytesLike) => string`
返回指定bytes的RIPMED-160的哈希值

`ethers.utils.sha256(aBytesLike) => string`
返回指定bytes的sha256的哈希值

`ethers.utils.sha512(aBytesLike) => string`
返回指定bytes的sha512的哈希值

`ethers.utils.computeHmac(alg, key, data) => string`
通过指定的alg的哈希算法返回带有key的HMAC的data数据
其中算法可以通过`ethers.utils.SupportedAlgorithm.sha256、ethers.utils.SupportedAlgorithm.sha256`来指定

`ethers.utils.hasMessage(message) => string`
对指定的message计算信息摘要

`ethers.utils.namehash(name) => string`
对指定的内容返回器ENS name的hash值

`TypeDataEncoder.getcode(domain, types, values) => string`
返回通过EIP712编码后的domain数据

`TypeDataEncoder.getPayload(domain, types, values) => any`
返回各种JSON-RPC各种调用方法时的各种标准payload

`TypeDataEncoder.getPrimaryType(types) => string`
返回一个root type

`TypeDataEncoder.hash(domain, types, values) => string`
返回计算后的EIP712哈希

`TypeDataEncoder.hashDomain(domain) => string`
返回计算后的EIP712的domain哈希

`TypeDataEncoder.resolveNames(domain, types, value, resolveName) => any`
返回value的副本

`ethers.utils.solidityPack(types, values) => string`
返回对types中对应类型打包的非标准编码值

`ethers.utils.solidityKeccack256(types, values) => string`
返回对types中对应类型打包的非标准编码的Keccack256哈希值

`ethers.utils.soliditySha256(types, values) => string`
返回对types中对应类型打包的非标准编码的Sha256哈希值