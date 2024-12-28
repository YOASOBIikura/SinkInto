---
title: React-04
date: 2024-12-27 14:35:10
categories:
  - React
tags:
  - Ref获取DOM
---

## Ref获取DOM的常用方法

1. 需要给元素设置ref="xxx",后面在函数中可以根据this.refs.xxx来获取DOM元素(已不推荐使用)
2. 把ref设置为一个函数，如:
```
ref = {x => this.xxx = x}
```
后面需要获取即通过this.xxx获取元素
3. 通过xxx = React.createRef()方法创建一个ref对象(reef={this.xxx}) => this.xxx = {current:null},后面通过this.xxx.current进行获取DOM元素

如果给类组件设置ref，获取的是调用类组件的实例，后续可以根据此获取其中的属性、状态、子组件等
如果给函数组件设置ref会直接报错，无法获取。但是我们可以通过Reac.forwardRef实现Ref转发，获取函数组件内部的某个组件