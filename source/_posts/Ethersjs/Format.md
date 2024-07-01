---
title: Format
date: 2024-06-29 10:24:08
categories:
  - ethers
tags:
  - format
---

`ehters.uitls.commify(value) => string`
返回三位格式化后值字符串

`ethers.utils.formatUnits(value, nuit) => string`
根据指定nuit转换为相对应单位的eth单位表示字符串，value必须是Bignumber类型

`ethers.utils.formatEther(value) => string`
转换为以ETH为单位的值，等价于调用formatUnits(value, "ether")

`ethers.utils.parseUnits(value, unit) => string`
于formatUnits方法正好相反，是把指定的value转换为对应nuit的ETH单位的字符串

`ethers.utils.parseEther(value) => BigNumber`
等价与parseUnits(value, 'Ether')
