---
title: Strings And Transaction
date: 2024-07-02 21:25:03
categories:
  - ethers
tags:
  - Strings
---

## Bytes32String

`ethers.utils.parseBytes32String(byteslike) => string`
返回对bytes32编码数据进行解码的字符串

`ethers.utils.formatBytes32String(text) => string`
返回指定内容的bytes32字符串的表示

## UTF-8String

`ethers.utils.toUtf8Bytes(text, form) => Uint8Array`
返回指定内容的UTF-8字节的表示

`ethers.utils.toUtf8CodePoints(text, form) => Uint8Array`
返回text的codePoints数组

`ethers.utils.toUtf8String(aBytesLike, error) => string`
返回byteslike的UTF-8表示的字符串

`ethers.utils.accessListify(anAccesslistish) => AccessList`
将指定内容标准化为一个AccessList

`ethers.utils.parseTransaction(BytesLike) => Transaction`
解析经过序列化的交易属性

`ethers.utils.serializeTransaction(tx, signature) => string`
返回经过序列化后的交易对象，可以通过签名参数返回签名的和未签名的交易对象


