---
title: DevFront-01
date: 2024-12-28 20:23:50
categories:
  - front
tags:
  - Vue与React的渲染方式
---

## Vue的渲染方式

Vue在渲染模版的过程中主要会经历编译时和运行时两个过程，会先经过编译时将模版编译为render函数后通过运行时执行render函数生成虚拟DOM后将根据此渲染成真实DOM的一个过程。

首先在编译时:
1. vue会通过自动状态机模块将模版HTML转换为AST(抽象语法树)
2. 在拿到生成的AST后，vue会将其转换为javascriptAST，由于这个AST会比原有的AST多一个codegenNode节点，这个节点描述了如何生成render函数的详细细节
3. 最后vue通过这个节点生成对应模版的render函数然后进入运行时的流程

然后进入运行时:
1. vue会解析渲染器对象所提供的render渲染函数
2. 解析之后执行该渲染函数时会判断之前的vnode是否存在
3. 如果不存在则将vnode挂载到容器中；如果存在则执行patch更新操作
这就是vue的一个大概的渲染过程，其中挂载操作是根据vnode type通过document.createElement生成相应的DOM元素，然后将vnode的children加入到生成DOM的innerText中，最后通过容器的appendChild方法进行挂载

## React的渲染方式

由于React中没有使用模版而是通过JSX进行页面创建，也就是对JSX的底层处理
1. 将编写的jsx代码编译为虚拟的DOM对象，也就是通过React中的一个插件(babel-preset-react-app)将JSX编译为React.createElement()这样的函数
2. 然后执行该函数构建出虚拟的DOM对象，并缓存一个OldVirtualDom
3. 最后把构建好的虚拟DOM渲染为真实DOM，后期视图更新的时候同样会经过一个DOM-DIFF的比对计算出一个PATCH更新包然后进行视图更新
