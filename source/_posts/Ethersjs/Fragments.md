---
title: Fragments
date: 2024-06-24 21:28:00
categories:
  - ethers
tags:
  - Fragments
---

## Output Formats

`ethers.utils.FormatTypes.full => string`
返回所有ABI属性，主要特点是提高可读性

`ethers.utils.FormatTypes.minimal => string`
类似于full，但删除不必要的空格和参数名，对存储最小的字符串非常有用

`ethers.utils.FormatTypes.json => string`
通过json格式返回ABI

`ethers.utils.FormatTypes.sighash => string`
输出的最小格式，可通过计算一个函数选择器，可以在计算签名哈希和主题事件的哈希时使用

## Fragment
ABI中的每一个数据块单位(错误，事件，函数，构造函数)就是一个Fragment,多个Fragment集合就是一个合约的ABI

### properties

`fragment.name => string`
一个数据块单位的名称，可以是函数名或事件名

`fragment.type => string`
显示数据块属于的那种类型例如event,function

`fragment.inouts => array`
数据块的参数类型的数组

### ConstructorFragment

`fragment.gas => Bignumber`
显示部署期间应该使用的gas

`fragment.payable => boolean`
显示该构造函数在部署期间是否可以接收ether

`fragment.stateMutability => string`
显示构造函数的state mutability,显示是否是可支付的

`ethers.ConstructorFragment.from(objOrStr) => ConstructorFragment`
从指定的object或string创建一个新的ConstructorFragment

`ethers.ConstructorFragment.isConstructorFragment(obj) => ConstructorFragment`
判断该Obj是否是ConstructorFragment

### ErrorFragment

`ethers.utils.ErrorFragment.from(objOrStr) => ErrorFragment`
从指定的Obj或者Str中创建ErrorFragment

`ethers.utils.ErrorFragment.isErrorFragment(obj) => ErrorFragment`
判断该Obj是否是ErrorFragment

### EventFragment

`fragment.anonymous => boolean`
显示事件是否匿名事件

`ethers.utils.EventFragment.from(objOrStr) => EventFragment`
从指定的Obj或者Str中创建EventFragment

`ethers.utils.EventFragment.isEventFragment(obj) => boolean`
判断该Obj是否是EventFragment

### FunctionFragment

`fragment.constant => boolean`
显示该函数是否是只读或者既不读也不写的

`fragment.stateMutability => string`
显示函数的状态可变性

`fragment.outputs => array`
返回返回函数的参数类型

`ethers.utils.FunctionFragment.from(objOrStr) => FunctionFragment`
从指定的Obj或者Str中创建FunctionFragment

`ethers.utils.FunctionFragment.isFunctionFragment(obj) => boolean`
判断该Obj是否是FunctionFragment

### ParamType

`paramType.name => string`
返回参数名

`paramType.type => string`
返回参数的完整类型

`paramType.baseType => string`
对于基类型同上，会显示复合类型的基础类型

`paramType.indexed => boolean`
显示该参数是否被标记为索引

`paramType.arrayChildren => paramType`
显示数组中的基类型

`paramType.arrayLength => string`
返回参数数组的长度

`paramType.components => array`
返回元组的组成部分

`paramType.format(outputType) => string`
通过指定的outputType formats创建Fragment参数的描述

`ethers.utils.ParamType.from(objOrstr) => ParamType`
返回指定Obj和str的新的ParamType

`ethers.utils.ParamType.isParamType(obj) => boolean`
判断obj是否是一个ParamType
