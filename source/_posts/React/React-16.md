---
title: React-16
date: 2025-01-14 11:43:57
categories:
  - React
tags: 
  - redux-saga
---

##  redux-thunk的不足
- 异步操作代码分散到各个actionCreator中，且不能以集中/统一的方式进行管理
- 不方便做单元测试

使用Saga
```
const sagaMiddleware = createSagaMiddleware();

const store = createStore(
    reducer,
    applyMiddleware(sagaMiddleware)
)

// 开启监听器对象执行
sagaMiddleware.run(saga)

```
创建监听器
```
import {
    take，
    takeEvery,takeLatest,
    throttle,
    debounce,
    call,
    apply,
    fork,
    delay,
    put
} from 'redux-saga/effects'

// 创建执行函数，在任务被监听后，去做异步操作
const workingCount = function* workingCount(action){
    // action 组件派发时候传递的action对象
    yield put({ // 派发任务通知reducer执行
        tyep: XXX,
        payload: action.paylpad
    })
}

// 创建监听器，监听派发的任务(Generator函数)
const saga = function *saga(){
    // 当前任务被监听到后才会往下走
    while(true){ // 多次监听
      let action = yield take(`Xxxx@SAGA@`)
      yield workingCount(action)  
    }
}
```
由于每次基于dispathch进行派发的时候，一定会吧reducer执行一遍，再去saga中间件重判断此任务是否被监听
所以如果打算进行"同步派发"
    则需要再派发行为标识需要和reducer中做判断的标识保持一致
    并且在saga中，不要再对这个标识进行监听，该表示可以在action-type中进行统一标记管理
如果是打算进行"异步派发"
    则派发的标识一定不能和reducer中的判断的标识一样
    需要saga中对这个表示进行监听!监听到派发后，进行异步的操作处理
    可以指定一个规则，如在正常表示后加一个标记表示是异步的派发
    当异步操作结束，我们基于yield put进行派发的时候，设置的派发表示要和reducer中做判断的标识保持一样，进行真正的派发操作
其中主要通过yield take(异步标识)来进行标识监听，只有监听到了之后程序才会往下走
    只是如果只写yield take这样只会被监听一次，我们需要加一个死循环进行多次监听

yield takeEvery(异步标识，要执行的方法)
等价于上述死循环的写法

yield takeLatest(异步标识，要执行的方法)
和takeEvery一样，每一次异步派发都会被检测到，都会把要执行的方法进行执行
只不过在执行方法前，会把当前任务之前正在运行的操作都结束掉，只保留当前最新执行的任务(也就是最新一次，有点类似于防抖函数)

yield throttle(ms, 异步标识，要执行的方法)
对异步派发进行节流处理：组件中频繁进行派发操作，我们控制一定的触发频率
它不是对执行的方法做节流，而是对异步任务的监听做节流:第一次异步任务被检测到派发后，下一次检测到需要过设定的ms这么长时间后才能再次被监听到

yield debounce(ms, 异步标识，要执行的方法)
和takeLatest一样，也是做防抖处理
它是对异步任务的检测做防抖处理

------
yield delay(2000) 设置延迟操作，只有延迟时候到达后才会继续向下执行

yield put(action) 派发任务到reducer，等价于dispatch

yield select(state => state.Xxx) 获取状态属性 

yield call(fn, arg1, arg2, ...)  基于call方法，可以吧指定的函数执行
                                 项目中一般基于call方法从服务器获取数据

yield apply(this, fn, [arg1, arg2, ...])  作用与call相同但是如果需要指定this则使用apply

yield fork(fn, ...args) 以非阻塞的形式函数

yield all({}) 并行执行方法
```
let {a, b} = yield all({
    a: call(fna, ...args),
    b: call(fnb, ...args)
})
```