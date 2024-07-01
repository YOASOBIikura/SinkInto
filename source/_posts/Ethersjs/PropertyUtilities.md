---
title: PropertyUtilities
date: 2024-06-30 11:28:47
categories:
  - ethers
tags:
  - Property Utilities
---

`ethers.utils.checkProperties(obj, check) => void`
检查对象是否包含check中的属性，不包含则抛出错误

`ethers.utils.deepCopy(obj) => void`
获得一个对obj对象的递归副本

`ethers.utils.defineReadOnly(obj, name, value) => void`
对obj对象上的属性设置为只读

`ethers.utils.getStatic(Constructor, key) => any`
从初始化方法中以及继承链上递归的检查静态方法的key

`ethers.utils.resolveProperties(obj) => any`
解析obj里的所有属性

`ethers.utils.shallowCopy(obj) => any`
返回一个obj的浅拷贝