---
title: React-03
date: 2024-12-26 11:15:40
categories:
  - React
tags:
  - React组件化开发
---

## 静态组件(函数组件)

在JSX文件中创建一个函数返回一个JSX视图(VirtualDOM)，这就是创建了一个函数组件。然后通过ES6Module导入导出使用即可！
组件的命名一般都采用大驼峰命名法 

函数组件属于静态组件，第一次渲染组件把函数执行
- 产生一个私有上下文
- 把传递的Props传递进来
- 对函数返回的JSX元素也就是虚拟DOM进行渲染
- 对上下文的私有变量进行改变并不会去触发页面的重新渲染组件中内容不会改变了所以称为静态组件 

### 渲染机制
1. 基于babel-preset-react-app把调用的组件转换为createElement格式的方法
2. 把该createElement方法执行，创建出一个virtualDOM对象
3. 基于root.render把virtualDOM变为真实的DOM
    type值不再是一个字符串，而是一个函数时=>把该函数执行->把virtualDOM中的props作为实参传递给该函数
    然后接收函数执行的返回结果(也就是当前组件的virtualDOM对象)
    最后基于render把组件返回的虚拟DOM变为真实DOM插入到root容器中

### 属性props的处理
1. 传递进来的属性是只读的(props对象被冻结了)
2. 父组件调用子组件的时候可以通过props传递给子组件，通过传递不同的属性呈现不同的效果让组件的复用性更强
3. 虽然对于传递进来的属性不能直接修改但是可以做基础的规则校验
    通过函数组件的defaultProps = {}来设置默认值或者通过一个官方的插件prop-types来进行更复杂的规则校验-> 组件名.propTypes = {
        title: PropTypes.string.isRequired,
        x: PropTypes.number
    }
    传递进来的属性，首先都会经历规则校验，不管校验成功还是失败最后都会吧属性传给形参props，只不过如果不符合设定的规则，控制台会抛出警告错误。 


## 动态组件(类组件、HooKs组件)

1. 创建一个类集成React中Component/PureComponent这个类
2. 必须给当前类设置一个render方法，并且其render方法返回需要渲染的视图
3. 并且给当前的类设置四个私有属性: props/context/refs/updater
4. 如果自己设置了constructor,则内部第一句话一定执行super()

### 从调用类组件开始，类组件内部发生的事情
1. 首先会进行初始化属性和规则校验在构造函数中对props等属性进行挂载，即便我们自己不在构造函数中处理，React内部也会把传递的props挂载到实例上；所以在方法中可以直接通过this.props上获取传递的属性，且同样是冻结的。
  并且在类中通过:
  ```
  static defaultProps = {
    ....
  }

  static propTypes = {
    ....
  }
  ```
  来进行属性的规则校验
2. 初始化状态(后期修改状态可以触发视图更新)
需要手段初始化，如果我们没有去做相关的处理，则默认会向实例上挂载一个state，初始值为null => this.state = null
```
state = {
  ... 
}
```
并且想直接更新状态是不会触发更新的，只能通过this中自带的setState方法进行更改状态才能触发视图更新

此外还可以通过this.forceUpdate()去触发页面强制更新
3. 触发componentWillMount周期函数(钩子函数):组件第一次渲染之前
此函数目前不安全未来可能会被移除
4. 触发render周期函数=>渲染
5. 触发componentDidMount周期函数: 第一次渲染完毕
已经把virtrualDom渲染为了真实的DOM，所以我们还在这个钩子函数内部获取真实的DOM了

### 组件内部更新的逻辑
1. 触发shouldComponentUpdate(nextProps, nextState)周期函数:是否允许更新,形参表示修改前的新的props和state
2. 触发componentWillUpdate(nextProps, nextState)周期函数: 更新前，在这个阶段状态还没有修改
3. 修改状态值/属性值
4. 触发render周期函数: 组件更新
  按照最新的状态和属性，把返回的JSX编译为virtualDom并和之前的virtualDOM进行对比(DOM-DIFF)并把差异部分进行渲染为真实的DOM
5. 触发componentDidUpdate周期函数：组件更新完毕

如果是基于this.forceUpdate()方法强制更新视图，会跳过触shouldComponentUpdate周期函数的校验，直接从WillUpdate开始进行更新 => 视图一定会触发更新

### 父组件更新触发子组件更新
1. 触发componentWillReceiveProps周期函数: 接搜最新属性之前
2. 触发shouldComponentUpdate周期函数
之后就是组件内部更新的逻辑