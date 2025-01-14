---
title: React-14
date: 2025-01-08 10:29:36
categories:
  - React
tags: 
  - react-router-dom(V6)
---

在react-router-dom V6版本中，移除了:
- Switch
- Redirect => 代替方案: Navigate
- withRouter => 代替方案: 自己写一个withRouter

## 使用规则
所有路由匹配规则放在<Routes>中，每条规则的匹配还是基于<Route>
- 路由匹配成功，不再基于component/render控制渲染的组件，而是基于element,语法给是<Component />
- 不再需要Switch，默认就是一个匹配成功，就不在匹配下面的路由了
- 不再需要exact，默认每一项都是精准匹配
原有的<Redirect>操作，被<Navigate to="/" />代替
- 遇到<Navigate/>组件，路由就会跳转到to指定的路由地址
- 设置replace属性，则不会新增立即记录，而是替换现有记录
- <Navigate to={{....}}/> to的值可以是一个对象: pathname表示需要跳转的地址、search问号传参信息
```
<Routes>
    <Route path="/" element={<Navigate to="/a">} />
    <Route path="/a" element={<A />}>
        // V6版本中所有路由(包括多级路由)，不分散到各个组件中编写，而是统一都写在一起进行处理
        <Route path="/a" elemrnt={<Naviget to="/a/a1"/>} />
        <Route path="/a/a1" element={<A1/>}/>
    <Route/>
    <Route path="/b" element={<B/>}/>
    // 都不匹配时就重定向到A
    <Route path="*" element={<Navigate to={{
        pathname: '/a',
        search: '?from=404'
    }}>}/>
</Routes>
``` 
子路由组件通过`<Outlet/>`来进行展示

## 跳转传参

在react-router-dom V6中，即便当前组件是基于<Router>匹配渲染的，也不会基于属性把history/location/match传递给组件！想获取相关信息，我们只能基于Hook函数进行处理：
1. 首先确保需要使用的"路由Hook"的组件，是在Router(HashRouter或BrowserRouter)内部包着的，否则使用这些Hook会报错！！
2. 只要<Router>内部包裹的组件，不论是否是基于<Router>匹配渲染的:
    默认都不能再基于props获取相关信息了
    只能基于"路由Hook"去获取

为了类组件中也可以获取路由的相关信息:
1. 稍后我们构建路由表的时候，我们会想办法: 继续让基于<Route>匹配渲染的组件，可以基于属性获取需要的信息
2. 不是基于<Route>匹配渲染的组件，需要自己重现withRouter，让其和基于<Route>匹配渲染的组件具备相同的属性

实现跳转方式主要有以下几种:
- <Link/NavLink to="/a" /> 点击条状路由
- <Navigate to="/a" /> 遇到这个组件就会跳转
- 编程式导航: 取消了history对象，基于navigate函数实现路由跳转
```
import {useNaviget} from 'react-router-dom'

const navigate = useNavigate();
navigate('/c');
navigate('/c', {replace: true})
navigate({
    ....
})
```

### 传参方式
1. 问号路径传参
```
navigate({
    pathname: '/c',
    search:qs.stringify({
        id: 100,
        name: 'zhufeng'
    })
})
```
组件接收
```
const location = useLocation()

const usp = new URLSearchParams(location.search)
console.log(usp.get(xxx))

let [usp] = useSearchParams();
console.log(usp.get(xxx))
```
2. 路径参数作为路径的一部分
```
navigate(`/c/100/xxx`);
```
组件接收
```
const location = useLocation()
const params = useParams();
```
3. 隐式传参
```
navigate('/c', {
    state: {
        ....
    }
})
```
组件接收
```
const location = useLocation()
console.log(location.state)
```
这种方式在router5中，组件只要刷新，传递的值就会消失；但是在router6版本中隐式传递的信息即使刷新页面之后依然会被保留下来