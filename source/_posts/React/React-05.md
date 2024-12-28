---
title: React-05
date: 2024-12-28 15:23:43
categories:
  - React
tags:
  - setState处理
---

# setState(partialState, callback)
该函数接受两个参数，partialState支持部分状态更改，而callback在状态更改视图更新完毕后触发执行，会在componentDidUpdate周期函数之后执行。
即便我们在shouldComponentUpdate阻止了视图更新了，后续视图触发函数不会执行了但是callback依然会被触发！ 

在React18中，setState在任何地方都是异步执行的, React18中有一套更新队列机制基于异步操作实现状态的批处理(利用了更新队列updater机制来进行处理 => 在当前相同时间内遇到setState会立即放到更新队列中 => 此时状态/视图还未更新 => 所有操作代码执行结束会刷新队列把所有设置状态操作合并在一次只触发一次视图更新的操作)
- 减少了视图的更新次数，降低了渲染消耗的性能
- 让更新的逻辑和流程都更加清晰，有效管理代码的执行逻辑顺序

## React18和React16中，关于setState的同步异步问题
React18中不论在什么地方执行setState它都是异步执行的，都是基于updatter更新队列机制，实现批处理。
React16中如果在合成事件(jsx袁术的中基于onXxx的绑定事件)、周期函数中，setState的操作是异步的，但是如果setState在其他的异步操作中(如:定时器、手动获取DOM元素的事件绑定等)，它将变为同步的操作 => 立即更新状态并触发视图渲染。

flushSync方法可以导致批处理立即执行渲染