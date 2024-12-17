---
title: sol-02
date: 2024-12-10 11:46:23
categories:
  - Solidity
tags:
  - 内部函数
---

64 -> 32字节 -> 一位16进制是半个字节
   -> uint256

uint8 -> 1字节 -> 二位十六进制表示
uint16 -> 2字节 -> 四位十六进制表示
uint32 -> 4字节 -> 八位十六进制表示
uint64 -> 8字节 -> 十六位十六进制表示
uint128 -> 16字节 -> 三十二位十六进制表示
uint256 -> 32字节 -> 六十四位十六进制表示

# ABI编码函数

在合约开发中，对数据进行编码是较为常用的操作，而合约中的编码极度依赖以下几个函数。

```
abi.encode() => bytes
```
该内部函数可以对任何数据编码成十六进制的byte数据，具体的bytes位数长度根据uint的大小而改变如上面的列表所示

```
abi.encodePacked() => bytes
```
不会为每个参数添加额外的长度信息或对齐填充，因此生成的数据更加紧凑，但这也意味着解码时需要非常小心，因为缺少了长度信息和对齐可能会导致数据解析错误。

```
abi.encodeWithSelector() => bytes
```
通过函数选择器以及指定的函数参数生成calldata，通常使用this.function.selector获取函数选择器。

```
abi.encodeWithSelector() => bytes
```
获取签名编码等价于=>`abi.encodeWithSelector(bytes4(keccak256(signature), ...)`

# 转账方法

在合约中转账方法有很多，可以通过send、transfer、call方法来进行以太坊的转账。虽然他们的功能都是转账但是功能和特性都
不尽相同。

首先在使用call方法进行转账时，你可以指定要转发给接收方的 Gas 数量。这意味着你可以传递所有剩余的 Gas 或者一个自定义的 Gas 限额。这为调用者提供了更大的灵活性，尤其是在你希望确保接收方有足够的 Gas 来执行某些操作时。call 返回一个布尔值表示调用是否成功，以及一个包含返回数据的 bytes 类型。你可以检查这个布尔值来确定转账是否成功，并根据结果采取相应的措施。例如，你可以记录错误或尝试其他逻辑。所以它的主要作用可以指定 Gas 限额和提供完整的 ABI 编码数据，call 提供了更高的灵活性。你可以通过 call 来调用接收方的特定函数，甚至可以在同一笔交易中执行多个操作。这对于复杂的交互场景非常有用

其次transfer 和 send：这两个方法会向接收方发送固定数量的 2300 Gas。这是为了防止重入攻击（reentrancy attack），但也意味着如果接收方需要更多的 Gas 来完成其逻辑，那么这些额外的操作将无法被执行，并且可能会导致交易失败。transfer 会在转账失败时抛出异常并回滚整个交易，而 send 则返回一个布尔值表示转账是否成功。然而，send 不会抛出异常，因此你需要显式地检查返回值并处理可能的失败情况。由于这两个方法只能简单地发送以太币，不能调用接收方的任何函数。如果你需要与接收方进行更复杂的交互，那么 transfer 和 send 就显得不够灵活了。

在安全性方面call方法的安全性是最差的，会有重入的风险，所以在使用的时候需要考虑到加重入锁，而后俩者固定了只有2300的gas，这些gas能做的操作及其有限，不大肯可能做出太多复杂的操作所以transfer是转账当中较为安全的方法。