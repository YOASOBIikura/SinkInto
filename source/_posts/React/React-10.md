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
    在reducer中去实现状态的更新 => 1. 利于逻辑的复用 2. 利于维护和管理

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

redux在设计上也存在一些不好的地方
1. 我们基于的getState获取的是公共状态，是直接和redux中的公共状态共用相同的堆地址，这样导致
可以拿到公共状态后直接进行修改，导致修改逻辑混乱不好维护
2. 我们会吧让组件更新的方法，放在事件池中，让公共状态改变的时候会通知事件池中所有的方法执行
。此操作在放置方法的时候没有办法去设置状态的依赖，后期不论修改的状态是不是组件所需要的，事件池中
所有的方法都会执行，进而导致相关的组件进行更新！
3. 所有reducer的合并，并不是真正意义上的代码层面上的合并，而是创建一个总的reducer出来每一次派发
都是让总的reducer进行执行，这样就会导 致每个模块的reuducer都会完整switch执行一遍，导致效率降低

## redux的工程化开发
1. 按照模块，把reducer进行单独管理，每个模块都有自己的reducer；最后，我们还要把所有的reducer进行合并，合并为一个然后赋值给创建的store
各模块下reducer的模版:
```
const initial = {
  ...
}

export default function XxxReducer(state = initial, action){
  state = clone(state) // 深拷贝
  switch(action.type){
    ....
  }
  return state
}
```
```
export default const reducer = combineReducers(
  {
    xxx: XxxReducer
  }
)
```
2. 每一次dispatch派发的时候，都会去每个模块的reducer中找一遍，把所有和派发行为的标识匹配的逻辑执行一遍
可能会有标识冲突的问题，所以我们派发的行为标识必须是唯一的，解决办法就是宏管理(统一管理标识问题)
3. 创建actionCreator对象，按模块管理我们需要派发的行为对象，通过将各个reducer的dispatch方法进行封装成函数
返回相应的dispatch对象最后进行集合导出
```
export default const xxxAction = {
  fn1(){
    return {
      type: TYPE.XXX
    }
  },
  fn2(){
    return {
      type: TYPE.YYY
    }
  }
}
```
```
export default action = {
  xxx: xxxAction
}
```

## react-redux 简化redux在组件中的应用
1. 提供Provider组件，可以自己在内部创建上下文对象，把store放在根组件上下文中
```
<Provider store = {store}>
  <XXX/>
</Provider>
```
2. 提供的connect函数，在函数内部可以获取上下文中的store，然后快速的把公共状态以及dispatch
操作基于属性传递给组件
```
connect(mapStateToProps, mapDispatchToProps)(渲染的组件)
```
```
connect(state => state.XXX)(XXX)
connect(null, action.XXX)(XXX)
```


## redux中间件
- redux-logger 每一次派发，在控制台输出派发日志，方便对redux的操作进行调试！！
(输出内容: 派发之前的状态、派发的行为、派发后的状态)
- redux-thunk/redux-promise 实现异步派发(每一次派发的时候，需要传递给reducer
的action对象中的内容，说是需要异步获取的)
等等其他一系列的中间件

加入中间件:
```
const store = createStore(
  reducer,
  applyMiddleware(reducerLogger) // 中间件
) 

export default store
```

redux-thunk 执行异步action
```
fn(){
  return async (dispatch) => {
    await delay();
    dispatch( {
      type: XXX
    })
  }
}
```
1. 首先方法执行返回一个函数(也算是一个对象)， 内部给函数设置一个type属性，属性值
不会和reducer中的逻辑进行匹配！！
=> 第一次派发 type不会和reducer中的任何逻辑进行匹配，所以没有修改任何状态！
2. 把返回的函数执行，把派发的方法dispatch传递给函数
=> 接下来在函数中，完成异步操作，操作完成之后中间件自己再基于手动dispatch进行派发！
总共会派发两次

异步操作中间件还可以用redux-promise,只用在action方法前面加async即可更加方便
1. 此dispatch也是被重写的，传递进来的是promise实例:
- 没有type属性，但是不报错
- 也不会通知reducer执行

2. 但是他会监听执行方法的返回值(promise实例)等待实例为成功的时候，它内部会自动再派发一次任务
- 用到的是store.dispatch派发(会通知reducer执行)
- 把实例变为成功的结果(也就是标准的action对象)传递给reducer，实现状态的改变