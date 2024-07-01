---
title: Signature
date: 2024-07-01 21:43:56
categories:
  - ethers
tags:
  - Signature
---

`ethers.utils.SigningKey(privateKey)`
对指定的私钥创建一个新的签名密钥

`signingKey.privateKey => string`
返回签名密钥的私钥

`signingKey.publicKey => string`
返回签名密钥的公钥（未压缩）

`signingKey.cpmpressedPublicKey => string`
返回签名密钥的压缩公钥

`signingKey.computeSharedSecret(otherKey) => string`
通过指定的otherKey计算ECDH共享密钥

`SigningKey.isSigningKey(obj) => boolean`
判断obj是否是签名密钥

`ethers.utils.verifyMessage(message, signature) => string`
返回生成签名的地址

`ethers.utils.verifyTypedData(domain, types, value, signature) => string`
返回EIP712签名的地址

`ethers.utils.recoverPublicKey(digest, signature) => string`
返回对摘要进行签名的公钥

`ethers.utils.computePublicKey(key, compressed) => string`
计算key的公钥，通过compressed参数确定是否进行压缩
