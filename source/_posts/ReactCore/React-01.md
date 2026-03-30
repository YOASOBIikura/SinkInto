---
title: React-01
date: 2026-01-24 12:37:25
categories:
  - ReactCore
tags:
  - Fiber设计
---

## fiberDOM结构属性

child: 第一个子节点
sibing: 下一个兄弟节点
return: 父节点

## fiber基础属性

tag: fiber类型，host(dom), hostComponent(5)->原生DOM， hostText(6)->文本节点。显示都是数值类型

type: 为tag下的更为细分的类型
      对于函数组件->函数本身
      DOM元素->h1、span等

elementType: 大多数情况下等同于type, 但在memo、lazy高阶组件的情况下不太一样

stateNode: 存储DOM元素->DOM节点
               类组件->实例
               HostRoot->FiberRoot对象->一种新的FiberRoot结构，为了优化性能而设计

ref: 不一定存在，存在的话存储的就是DOM节点

pendingProps: 存储ReactElement的props

所有ReactElement转过来的都是ChildFiber，所有原生生成的叫HostRootFiber


