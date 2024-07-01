---
title: FixedNumber
date: 2024-06-29 14:24:21
categories:
  - ethers
tags:
  - FixedNumber
---

## FixedNumber
通常是一个内部有十进制除数固定宽度的number,可以用来表示十进制小数部分,其实就是跟安全的浮点类型

`FixedNumber.from(value, format="fixed") => FixedNumber`
通过指定值和format返回一个FixedNumber

`FixedNumber.fromBytes(BytesLike, format="fixed") => FixedNumber`
通过给定的bytes返回一个FixedNumber

`FixedNumber.fromString(value, format="fixed") => FixedNumber`
通过给定的字符串返回一个FixedNumber

`FixedNumber.fromValue(value, decimals, format="fixed") => FixedNumber`
通过给定的值和精度返回FixedNumber

`fixednumber.addUnsafe(otherValue) => FixedNumber`
浮点类型的非安全的相加

`fixednumber.subUnsafe(otherValue) => FixedNumber`
浮点类型的非安全的相减

`fixednumber.mulUnsafe(otherValue) => FixedNumber`
浮点类型的非安全的相乘

`fixednumber.divUnsafe(otherValue) => FixedNumber`
浮点类型的非安全的相除

`fixednumber.round(decimals) => FixedNumber`
返回一个新值并按照精度进行四舍五入

`fixednumber.isZero() => boolean`
判断是否为0

`fixednumber.toFormat(format) => FixedNumber`
通过指定的format格式输出一个新的FixedNumber

`fixednumber.toHexString() => FixedNumber`
返回值的十六进制字符串格式

`fixednumber.toString() => FixedNumber`
返回值的字符串格式

`fixednumber.toUnsafeFloat() => float`
返回一个js和float类型值，会根据js number类型进行四舍五入

`FixednNmber.isFixedNumber(value) => boolean`
判断是否为fixedNumber类型

