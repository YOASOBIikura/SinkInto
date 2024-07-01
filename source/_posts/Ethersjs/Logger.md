---
title: Logger
date: 2024-07-01 21:10:26
categories:
  - ethers
tags:
  - Logger
---

## Log

`ethers.utils.Logger(version)`
创建一个新的Logger

`Logger.globalLogger() => Logger`
返回单例全局的logger

`logger.debug(..args) => void`
打印相关的debug信息

`logger.info(...args) => void`
打印通用的信息

`logger.warn(...args) => void`
打印warnings信息

## Errors

`logger.makeError(message, code, params) => Error`
创建一个指定了消息、标志code和附加参数的错误对象。利用其来拒绝错误。

`logger.throwError(message, code, param) => never`
抛出一个指定了消息、标志code和附加参数的错误对象。

`logger.throwArgumentError(message, name, value) => never`
抛出一个指定name和value的参数异常错误

## Usage Validation

`logger.checkAbstract(target, kind) => void`
检查target是否是kind，用来检查抽象类没有被实例化

`logger.checkArgumentCount(count, expectedCount, message) => void`
检查参数的个数是否符合函数的标准

`logger.checkNormalize(message) => void`
用来检查环境是否具有String.normalize的功能

`logger.checkSafeUint53(value, message) => void`
检查指定的value作为一个js number是否是安全的

`Logger.setCensorship(censor, permanent) => void`
设置错误审查，它会让调试变得困难，实际开发中很少用到

`Logger.setLogLevel(logLevel) => void`
用来设置日志的级别

