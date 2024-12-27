---
title: React-02
date: 2024-12-25 19:23:22
categories:
  - React
tags:
  - JSX基础
---

JSX: javascript and xml(html) 把JS和HTML混合在一起

大括号允许的js表达式: 变量、数学运算、判断、循环(数组的迭代方法)，一个jsx只能有一个根节点，根容器也不能指定body标签

{}在渲染值的时候如果value是boolean/null/undefined/Symbol/BigInt: 则渲染出来的东西都为空，而普通对象则不支持渲染，数组的话则把每一项都拿出来渲染，并不是盖层字符串渲染中间没有逗号

其中行内css样式需要通过双大括号，来写行内样式用小驼峰命名

设置样式类名需要通过把class替换为className

## JSX底层处理机制
1. 把我们编写jsx代码编译成虚拟的DOM对象(框架内部构建的一套对象体系，根据这些属性描述构建视图的DOM节点的相关特征)
    a.  基于babel-preset-react-app把JSX编译为React.createElement(...)这种格式
    {
        ele： 元素标签名(组件名)
        props: 元素的属性集合，没有设置则为null
        children: 当前元素的子节点
    }
    b. 再把createElement方法执行，创建出虚拟DOM对象(JSX元素、JSX对象)
2. 把构建的虚拟DOM渲染为真实的DOM,并且缓存成一个OldVirtrualDOM
    v16
    ReactDom.render(
        <>...</>,
        document.getElementById('root')
    )
    v18
    const root = ReactDom.createRoot(document.getElementById('root'))
    root.render(
        <>...</>
    )
3. 后期视图更新的时候需要经过一个DOM-DIFF的对比，计算出补丁包PATCH=>也就是两次视图差异的部分，然后再把PATCH补丁包进行渲染=>Page视图更新

render函数在渲染的时候，如果type是:
字符串: 创建一个标签
普通函数: 把函数执行(创建一个新的上下文)，并把props传递函数
对象(构造函数): 把构造函数执行基于new执行=>创建一个类的实例,会把props解析出来传递过去
    - 每次调用类组件都会创建一个单独的实例
    - 并执行render函数，把返回的JSX当做组件视图进行渲染