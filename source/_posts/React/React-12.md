---
title: React-12
date: 2025-01-07 10:57:51
categories:
  - React
tags: 
  - mobx
---

## mobx 
mobox是redux的代替方法，使用比redux更加简单
1. 可以先创建一个容器
```
import { observable, action } from 'mobx'
import { observer } from 'mobx-react'

// 创建容器
class Store{
    @observable //监听状态
    num = 10
    @action // 修改公共状态的方法
    change(){
        this.num++
    }
}
```
2. 在组件中直接引用即可
```
// 类组件
@observer
class Xxx{}

// 函数组件
observer(
    Xxx
)
```
3. 在组件中的回调
首先会立即执行一次，自动建立起依赖监测(监测用到的状态);当依赖的状态值发生改变时，callback会重新执行!
```
import {autorun} from 'mobx'

autorun(()=>{
    ......
})
```
4. 属性监听执行回调
创建监听器，对对象进行监听，当对象中的某个成员发生改变，触发回调函数执行(前提是对象是基于observable修饰的把其变为可监听的)
```
import {observe} from 'mobx'

observe(obj, change => {
    ....
})
```
5. 计算缓存效果
与Vue3中的computed类似
```
import {computed} from 'mobx'

class Store{
    @observable
    x = 10
    @observable
    y = 10
    @computed 
    get total(){
        return this.x * this.y
    }
}
```
6. 细粒化监测
与autorun一样，都是监听器，但是提供更细粒化的状态监测(默认不会执行)
```
import {reaction} from 'mobx'

reaction(
    ()=>[store.a, store.b],
    ()=>{
        .....
    }
)
```
7. 全局配置项
强制进行一些全局的限制，例如强制使用action方法的模式，去修改状态；不允许单独基于实例修改状态
```
import {configure} from 'mobx'

// mobx的全局配置
configure({
    enforceActions: 'observed'
})
```