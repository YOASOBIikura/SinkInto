---
title: React-08
date: 2024-12-31 15:40:57
categories:
  - React
tags:
  - 组件通信方案
---

## 父子通信
1. 父组件想把信息传递给儿子组件 => 基于属性Props
2. 儿子想传递信息或修改父组件属性 => 基于父亲传递方法props，儿子组件调用该属性方法
3. 父亲组件想把一些HTML结构传递给儿子 => 基于属性中的children(插槽)
4. 父亲组件调用儿子组件的方法的时候，可以给儿子组件设置ref，通过获取儿子组件的实例拿到儿子组件中暴露的数据与方法

## 上下文传递通信
基于上下文对象中，提供的Provider组件来存储信息
1. 向上下文中存储信息: value属性指定的值就是要存储的信息
2. 当祖先组件更新，render重新执行，会把最新的状态值再次存储到上下文对象中

后代组件中获取上下文信息
1. 导入创建的上下文对象
2. 给类组件设置静态私有属性contextType
3. 从this.context中获取需要的信息即可
或者
```
<XxxContext.Consumer>
{context => {
  let { Xxx } = context
  rerutn 渲染的视图
}}
</<XxxContext.Consumer>
```
函数组件中通过使用useContext钩子函数来获取上下文中的信息