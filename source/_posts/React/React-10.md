---
title: React-10
date: 2025-01-03 11:28:16
categories:
  - React
tags:
  - Redux
---

## Redux
1. 在创建的storee容器中，存储两部分内容
    公共状态: 各组件需要共享/通信的信息
    事件池：存放一些方法[ 让组件更新的方法 ]
特点: 当公共状态一但发生改变，会默认立即通知事件池中的方法执行!!
这些方法的执行主要是让指定的组件进行更新，而一更新就可以获取最新的公共状态信息进行渲染!!
 2. 修改公共容器中的状态不能直接修改
    基于dispatch派发，通知reducer执行
    在reducer中去实现状态的更新

使用步骤
1. 创建全局公共容器用来存储各组件需要的公共信息
`const store = createStore([reducer])`
2. 在组件内部，获取公共状态信息，然后渲染
`store.getState() => {supNum:..., oppNum:...,...}`
3. 把让组件可以更新的方法放在公共容器的事件池中
`store.subscribe(函数)`
后期公共状态改了，事件池中的方法会按照顺序，依次执行！也就是让对应的组件也更新！！
组件只要更新，就可以从store容器中获取最喜欢的状态渲染！！
4. 创建容器的时候需要传递reducer
```
let initial = {...}; // 初始状态值
const reducer = function reducer(state=initial, action){
  // state容器中的状态变量
  // action派发的行为对象(必须具备type属性)
  switch(action.type){
    // 根据传递的type值的不同，修改不同的状态信息
    ....
  }
  return state; // 返回的信息会替换store容器中的公共状态
}
```
5. 派发任务，通知reducer执行修改状态
store.dispatch({
  type:xxx,
  ....
})

总结: redux具体的代码编写顺序
1. 创建store，规划出reducer(当中的业务处理逻辑可以后续不断完善，但是最开始reducer的框架需要先写出来)
2. 在入口中，基于上下文对象，把store放入到上下文中；需要用到store的组件，从上下文中获取!!!
3. 组件中基于store，完成公共状态的获取、任务的派发
使用的公共状态组件，必须想store的事件池中加入让组件更新的办法；只有这样，才可以确保公共状态改变，可以让组件更新，才可以获取最新的状态进行绑定！！