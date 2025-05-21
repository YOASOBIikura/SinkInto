---
title: standerd
date: 2025-05-20 18:08:01
categories:
  - assembly
tags:
  - 汇编标准
---


## 整理一些内联汇编的一些标准写法

`mstore(0x40, and(add(数据结束位置, 31), not(31)))`
重置自由内存指针, 从当前数据结束位置开始向上取整到最近的32位地址


