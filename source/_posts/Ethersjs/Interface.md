---
title: Interface
date: 2024-06-25 20:25:00
categories:
  - ethers
tags:
  - Interface
---

## Interface
接口是与以太坊网络交互的一种抽象，一种与合约交互所需编码和解码的抽象。由于EVM并不理解什么是ABI，ABI只是一组商定的特定格式，用于编码合约所需的各种类型的数据以便它们可以相互交互。

`ethers.utils.Interface(abi) => Interface`
通过一段ABI数据来创建一个合约接口

`interface.fragments => array`
返回合约接口中的所有fragments

`interface.errors => array`
返回接口中所有的error fragments

`interface.events => array`
返回接口中所有的event fragments

`interface.functions => array`
返回接口中所有的function fragments

`interface.deploy => array`
返回接口中所有的constructor fragments

`interface.format(format) => string`
根据指定的格式化形式来返回abi数据

`interface.getFunction(fragment) => FunctionFragment`
可通过函数方法签名、函数名、函数选择器返回function fragment

`interface.getError(fragment) => ErrorFragment`
可通过函数方法签名、函数名、函数选择器返回Error fragment

`interface.getEvent(fragment) => EventFragment`
可通过函数方法签名、函数名、函数选择器返回Event fragment

`interface.getSighash(fragment) => string`
返回签名时的哈希数据

`interface.getEventTopic(fragment) => string`
返回指定fragment的topic hash

`interface.encodeDeploy(vaules) => string`
获取指定值的编码数据，以方便在部署合约时传递到合约构造函数中

`interface.encodeErrorResult(fragment, values) => string`
返回编码后的错误结果，通常在调试时使用

`interface.encodeFilterTopics(fragment, values) => array`
返回编码后的主题过滤器

`interface.encodeFunctionData(fragment, values) => string`
返回编码好的方法与传参数据，可以方便的直接写入交易的data字段中

`interface.encodeFunctionResult(fragment, values) => string`
返回编码后的调用结果，一般很少用

`interface.decodeErrorResult(fragment, data) => Result`
解码调用失败的方法数据，一般在revert之后将自动解码error所以这个方法很少使用

`interface.decodeEventLog(fragment, data, topics) => Result`
对指定的事件和数据进行解码

`interface.decodeFunctionData(fragment, data) => Result`
对编码的方法data进行解码

`interface.decodeFunctionResult(fragment, data) => Result`
解码指定fragment的调用结果

`interface.parseError(data) => ErrorDescription`
返回与数据中匹配的错误选择器并解析详细信息

`interface.parseLog(log) => LogDescription`
搜索匹配的日志主题事件，并对log进行解析

`interface.parseTransaction(transaction) => TransactionDescription`
匹配哈希签名相同的函数，并解析出交易相关的属性

