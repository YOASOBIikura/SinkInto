---
title: Bytes
date: 2024-06-27 20:36:01
categories:
  - ethers
tags:
  - Bytes
---

## 检查

`ethers.utils.isBytes(obj) => boolean`
判断obj是否是有效的Bytes

`ethers.utils.isBytesLike(obj) => boolean`
判断obj是否是Bytes或DataHexString对象

`ethers.utils.isHexString(obj, length) => boolean`
判断obj是否是满足指定长度的十六进制字符串

## 数组与十六进制字符串之间的转换

`ethers.utils.arrayify(DataHexStringOrArrayish, option) => uint8array`
将指定对象转换为uint8数组

`ethers.utils.hexlify(hexstringOrArrayish) => string`
将数组转换为十六进制字符串

`ethers.utils.hexValue(aBigNumberish) => string`
将一个大数类型或数组转换为十六进制字符串

## 数组处理

`ethers.utils.concat(arrayOfBytesLike) => Uint8Array`
将各个BytesLike拼接到同一个Uint8Array中并返回

`ethers.utils.stipZeros(aBytesLike) => Uint8Array`
返回一个没有前导0的byteslike的uint8数组

`ethers.utils.zeroPad(aBytesLike, length) => Uint8Array`
padding指定长度的前导0

## 十六进制字符串处理

`ethers.utils.hexContact(arrayOrBytesLike) => string`
将指定的byteslike连接成一个单一的DataHexString

`ethers.utils.hexDataLength(aBytesLike) => string`
返回指定对象的长度

`ethers.utils.hexDataSlice(aBytesLike, offset, endoffset) => string`
返回一个byteslike的一个切片

`ethers.utils.hexStripZeros(aBytesLike) => string`
返回去掉前导0后的对象

`ethers.utils.hexZeroPad(aBytesLike, length) => string`
padding指定长度的前导0

## 签名转换

`ethers.utils.joinSignature(aSignatureLike) => string`
返回一个aSignatureLike的原始格式

`ethers.utils.splitSignature(aSignatureLikeOrBytesLike) => Signature`
返回一个aSignaturelike的完整扩展格式

## 随机字节

`ethers.utils.randomBytes(length) => Uint8Array`
返回一个指定长度的随机uint8数组

`ethers.utils.shuffled(array) => array`
返回一个使用Fisher-Yates Shuffle打乱后的数组副本