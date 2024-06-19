---
title: AbiCoder-0
date: 2024-06-19 20:55:20
categories:
  - ethers
tags:
  - AbiCoder
---

## AbiCoder
该类是编码器的集合，主要将二进制数据进行编码和解码的操作

`ethers.utils.defaultAbiCoder => AbiCoder`
当文件中有使用Interface的库文件被导入时会自动创建一个AbiCoder

`abiCoder.encode(types, values) => string`
根据types数组的solidity类型来对values数组中的值进行编码，最后统一返回编码后拼接的字符串

`abiCoder.decode(types, data) => Result`
与encode方法相对应，通过type数组对data来进行解码，只能对简单类型进行解码只有字符串或是ParamType


