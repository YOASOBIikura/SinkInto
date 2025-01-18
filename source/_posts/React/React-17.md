---
title: React-17
date: 2025-01-15 11:32:38
categories:
  - React
tags: 
  - Dva
---

## Dva启动写法
```
// init
const app = dva({
    // option
    // 指定路由模式，默认是哈希路由
    history: createHistory(),
    // 扩展其他的中间件 redux-logger/redux-persist.....
    extraEnhancers: []
})

// Plugins
app.use({})

// Model 
// 一加载页面就会注册相关Model使用 
app.model(XxxxModel)

// Router
app.router(require('./router').default)

// Start
app.start('#root')
```

### 路由懒加载
在Dva中通过ddynamic进行路由懒加载

```
const demo = ({history, app})=>{

    // history：包含路由跳转方法的history对象
    // app: 基于dva创建的应用
    // 基于dva/dynamic实现懒加载组件以及对应的Models
    const LazyDemo = dynamic({
        app,
        models: () => [import(xxx)],
        component: () => import(xxx)
    })

    return <>
    // 子路由
    <Router history = {history}>
        <Switch>
            <Route path="/demo" component={LazyDemo}>
        </Switch>
    </Router>
    </>
}
```

## routerRedux跳转方案(Dva)
在Dva手脚架中除了使用原生的跳转方案还能使用Dva中routerRedux进行跳转
routerRedux是react-router-redux中提供的对象，此对象中包含了路由跳转的方法
- go/goBack/goFoward
- push/replace
相比较于props.history来说，它不仅可以在组件中跳转还可以在redux操作中进行跳转，非常方便
它本身就是redux和router的结合操作
    在redux内部
    yield put(...)
    在redux外部(或组件中)
    dispatch(
        routerRedux.push(...)
    )

在真是项目中Model的处理
1. 所有模块的Model, 我们一般都放在src/models目录下
2. 关于Model的加载问题
    a: 全部打包到主JS，页面一加载就把Models都准备好
        - 一加载页面，所有模块的Model都已经准备好了，可随时取用
        - 如果模块较多，对应的Model也就会很多，那么进行整合的主js就会很大，会降低页面第一次的渲染速度
    
    b: 我们可以把不需要在第一次加载完就处理的Model，只有进入相关组件才需要处理可以配合dynamic路由懒加载实现Model的懒加载处理

## Model
```
// Template
export default {
    namespace: 'Xxx', // 命名空间，集成后再组件使用时的标识
    state: {}, // 模块所管理的公共状态
    reducers: {
        // 一个纯函数修改本模块的状态
        fn(state, {payload}){
            state = {...state}
            ....
            return state
        }

    }, // 一个个方法的模式，完成派发标识的判别以及状态的更改
    effects: {
        // redux-saga中我们基于take/takeLatest/takeEvery等方式创建监听器，此时写成一个个的"Generator函数"即可
        // 默认基于takeEvery的方式创建的监听器
        // 方法名就是监听器的名字，由于每一次的派发，reducer和effects中的方法都会去匹配，所以要避免于reducer中的方法重名
        // 方法就是派发的任务被监听后，执行的working方法
        *fn({payload}, effect){ 
            // 基于yield effect.select()可以获取所有模块的公共状态

        }
        // 如果想要设置不同的类型的监听器
        fn: [
            function* ({payload}, effect){
                ....
            },
            // 数组第二项中指定监听器的类型
            {type: 'takeLatest'}
            // 或者 {type: 'throttle', ms: 500}
        ]
    }, // 基于redux-saga实现的异步派发
    subscriptions: {
        // 方法会有两个实参 
        fn({history, dispatch}){

            let unlisten = history.listen((location)=>{
                // 在Model没有懒加载的情况下，可以让函数hfn在第一次加载中久订阅到事件池中，并且通知执行。并且通过
                // listrn创建监听器，除了第一次会执行，以后每次路由切换也会执行
                ...
                unlisten() //通过该返回的函数停止监听
            })
        }
    } // 在这里订阅的方法，会在页面一加载的时候就会被通知执行(并且以后切换路由不再执行)所以
      //  我们可以把页面一加载需要去做的事情(与redux相关)放在这里处理
      //  在这里可以基于history.listen做事件监听，保证进入那个组件再进行处理
}
```
在组件中则通过dva中的connect的高阶组件获取props中的属性进行Model状态的操作方法
```
export default connect(
    state => state.demo
)(Xxx)
```

组件方法中执行派发
```
<button
onClick={
    dispatch({
        type:'demo/increate',
        payload: 5
    })
}
>...</button>
```

