---
title: Bignumber
date: 2024-06-26 21:38:13
categories:
  - ethers
tags:
  - Bignumber
---

## Bignumber
由于以太坊中需要操作的数字都是超过了js中安全范围值的数字，所以需要对数字包一层Bignumber避免计算精度丢失

`ethers.BigNumber.from(obj) => BigNumber`
接受的obj可以是string、BytesLike、BigNumber、number、BigInt等，返回一个BigNumber实例

### 数学运算相关

`BigNumber.add(value) => BigNumber`
相加

`BigNumber.sub(value) => BigNumber`
相减

`BigNumber.mul(value) => BigNumber`
相乘

`BigNumber.div(value) => BigNumber`
相除

`BigNumber.mod(value) => BigNumber`
求余

`BigNumber.abs() => BigNumber`
取绝对值

`BigNumber.mask(bitcount) => BigNumber`
对超出指定位数的低位置0

`BigNumber.fromTwos(bitwidth) => BigNumber`
返回指定带宽位的二进制补码转换而来的值，使用较少

`BigNumber.toTwos(bitwidth) => BigNumber`
将Bignumber转换为指定带宽位的二进制补码，使用较少

`BigNumber.eq(value) => boolean`
比较相等

`BigNumber.lt(value) => boolean`
是否小于

`BigNumber.lte(value) => boolean`
是否小于等于

`BigNumber.gt(value) => boolean`
是否大于

`BigNumber.gte(value) => boolean`
是否大于等于

`BigNumber.isZero() => boolean`
是否为0

`BigNumber.toBigInt() => bigInt`
转换为js支持的Bigint

`BigNumber.toNumber() => number`
转换为js中的number值，超出最大最小则会报错

`BigNumber.toString() => string`
以十进制字符串的形式返回BigNumber的值

`BigNumber.toHexString() => string`
以字符串的形式返回十六进制的值

`ethers.BigNumber.isBigNumber(obj) => boolean`
判断是否是BigNumber对象

