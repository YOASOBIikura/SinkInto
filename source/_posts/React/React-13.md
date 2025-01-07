---
title: React-13
date: 2025-01-07 15:14:50
categories:
  - React
tags: 
  - react-router-dom
---

基于`<HashRouter>`把所有要渲染的内容包起来，开启HASH路由
- 后续用到的`<Router>、<Link>`等都需要在HashRouter中使用
- 开启后，整个页面地址默认会设置一个#/哈希值

## <switch>
确保匹配一个之后就不用继续向下匹配了，此外还可以加入exact进行精准匹配

## <Redirect>
- from: 从那个地址来
- to: 重定向地址
- exact: 精准匹配

配置路由表
- redirect: true 此配置是重定向
- from: 来源地址
- to: 重定向地址
- exact: 是否精准匹配
- path: 匹配路由
- component: 渲染的组件
- name: 路由名称(命名路由)
- meta: {} 路由元信息 => 包含当前路由的一些信息，当路由匹配后可以获取该信息做一些事情...
- children:[] 子路由配置项

路由懒加载
```
import {lazy} from 'react'

component: lazy(()=>{
    return import(组件路径)
})
```

在react-router-dom V5中，基于Router路由匹配渲染的组件，路由会默认给每个组件传递三个属性:
- history
- location
- match
后期在组件中基于props/this.props获取传递的属性值，此外还可以通过回调函数获取这三个对象
useHistory() => history
useLocation() => location
useRouteMatch() => match

路由跳转传参方案
1. 问号传参
- 传递的信息出现在URL地址上: 丑、不安全、长度限制
- 信息是显示的，即便在目标路由内刷新传递的信息也在
```
history.push({
  pathname: '/xxx',
  search: qs.stringify({
    id: ...,
    name:....
  })
})
```
目标组件接收:
```
const location = useLocation()

// 获取传递的问号参数信息
let {id, name} = qs.parse(location.search.substring(1))

// 基于URLSearchParams来处理
let usp = new URLSearchParams(location.search)
```
2. 路径参数
把值作为路径的一部分
组件通过useMatch，useParams两个回调函数获取传参
3. 隐式传参
- 传递的信息安全美观没有限制
- 目标组件内刷新，传递的信息就丢失了
```
history.push({
  pathname: '/xxx',
  state: {
    id: ...,
    name:....
  }
})
```
目标组件通过useLocation回调函数进行接收
```
const localtion = useLocation()

let {id, name} = localtion.state
```

## NavLink VS Link
都是实现路由跳转，语法上几乎一样，区别是:
第一次页面加载或者路由切换完毕，都会拿最新的路由地址和NavLink中的to指定的地址(或者pathname地址)
进行匹配
- 匹配一样会默认设置active选中的样式(我们可以基于activeClassName重新设置选中的样式类名)
- 同样可以设置exact精准匹配
基于此可以实现导航中设置相关选中后的样式