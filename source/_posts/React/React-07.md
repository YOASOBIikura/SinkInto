---
title: React-07
date: 2024-12-30 10:20:57
categories:
  - React
tags:
  - Hooks
---

## useState
使函数组件使用状态，并且后期基于状态的修改可以让组件更新
`let xxx = useState(initialValue)`
- 执行useStatee，传递initialValue是初始的状态值
- 执行这个方法，返回结果是一个数组：[状态值， 修改状态方法] 
- 每一次修改值的时候都会拿最新修改的值和之前的状态值做比较(Object.is)如果发现两次的值是一样的则不会去修改状态使视图更新
- 如果传递的初始值非常复杂那就直接传入一个惰性函数来取值

函数组件和Hooks组件不是类组件，没有实例的概念，调用组件不再是创建类的实例而是把函数执行，产生一个私有的上下文而已，进而在函数组件中不涉及this的处理！

函数组件的每一次渲染或者更新都是把函数重新执行，然后产生一个全新的"私有上下文"
- 内部的代码也需要重新执行
- 涉及的函数需要重新的构建=>这些函数的作用域函数执行的上下文是每一次执行产生的闭包
- 每一次useState也会重新执行但是:
    执行useState,只有第一次设置的初始值会生效，其余以后再执行获取的状态都是最新的状态值而不是初始值
    返回修改状态的方法，每一次都是返回一个新的

执行一次usetState:把需要的状态信息都放在对象中统一管理
    - 执行setState的时候传递的是啥值，就把状态整体改为啥值
    - 只传单个状态变量，其他成员就会消失，不能只进行部分对象属性的更新


## useEffect
useEffect(callback): 
1. 第一次渲染完毕后，执行callback，等价于componentDidMount
2. 在组件每一次更新完毕后，也会执行callback，等价于componentDidUpdate

useEffect(callback, []):
1. 只有第一次渲染完毕后才会执行callback，之后更新就不再执行，类似于componentDidMount

useEffect(callback, [依赖的状态(可以是多个状态)]):
1. 第一次渲染完毕后，执行callback
2. 依赖的值发生改变，也会触发callback执行

useEffect(()=>{
    return ()=>{}
})
1. 组件更新，上一次组件释放时会吧这个返回的小函数执行


## useLayoutEffect
useEffect是发生在浏览器绘制和渲染之后，而useLayoutEffect是发生在render方法已经创建出DOM对象之后执行的，然后再进行浏览器渲染真实的DOM，也就是真实DOM只渲染一次，不会出现内容/样式的闪烁。 
- useLayoutEffect要优先于useEffect去执行
- 在两者设置的callback中，依然可以获取DOM元素，因为真实的DOM对象已经创建了，区别是在于浏览器是否已经渲染


## useRef
通过Ref获取一个DOM对象
```
let Xxx = useRef(null)

<... ref={Xxx}><.../>
```
 1. 在每一次组件更新的时候再次执行useRef不会创建新的REF对象，而createRef在每一次组件更新的时候都会创建一个全新的REF对象处理，比较浪费性能

 
 ## useImperativeHandle
 通过函数父子组件的React.forwardRef()返回子组件中的状态或者方法
 ```
 useImperativeHandle(ref, ()=>{
  return {
    ...,
    func...
  }
 })
 ```


 ## useMemo
`let Xxx = useMemo(()=>{}, [])`
1. 第一次渲染的时候callback会执行，后期只有依赖发生改变才会去执行，把结果返回给变量
2. useMemo具备缓存的效果，在依赖的状态值没有发生改变，callback没有触发的时候变量就是上一次计算出来的结果(类似于Vue中的Computed属性)


## useCallback
`const handle = useCallback(()=>{},[])`
1. 组件后续每一次更新，判断依赖是否发生改变，如果改变则会重新创建新的函数堆，并赋值给变量属性；但如果依赖没有更新或者没有设置依赖，则组件更新不会继续创建新的函数出来，而是一直使用第一次创建的函数堆

应用场景: 父组件嵌套子组件，父组件要把一个内部的函数，基于属性传递给子组件，此时传递的这个方法，我们基于useCallback处理一下会更好。函数组件通过React.memo来判断父组件传递的属性是否变化判断是否需要重新渲染子组件