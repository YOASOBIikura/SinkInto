---
title: React-01
date: 2024-12-25 13:21:38
categories:
  - React
tags:
  - 基本配置
---

## 基本开发命令

```start``` 开发环境启动项目

```build``` 生产环境打包部署，打包项目并将打包内容输入到dest目录中

```test``` 单元测试

```eject``` 暴露webpack配置规则（应为我想修改规则）

## webpack配置

path.js 打包中需要的一些路径管理

webpack.config.js 脚手架默认的webpack打包规则的配置

webpackDevServer.config.js webpack-dev-server的配置

scripts 启动命令的入口文件如yarn start等

@react-app-polyfill 对@babel/polyfill的重写

ES6内置API做兼容处理
```
import 'react-app-polyfill/ie9'
import 'react-app-polyfill/ie11'
import 'react-app-polyfill/stable'
```

React、Vue、Angular => 数据驱动页面思想
                       不会去操作DOM 转而去操作数据(修改了数据，框架按照相关的数据重新渲染)构建了一套 虚拟DOM => 真是DOM的渲染体系 => 有效避免了DOM的重排/重绘 => 开发效率更高并且最后的性能也最好

Vue中不仅有 数据驱动页面，它还能反过来通过页面来驱动数据再驱动页面


原生js => 直接操作Dom(非常消耗性能 => 可能会导致DOM重排(回流)/重绘)
          操作非常麻烦  

React框架采用的是MVC体系;Vue框架采用的MVVM体系

MVC体系: Model数据层 + View视图层 + Controller控制层
1.  页面: React基于Jsx来构建视图
2.  数据: 在页面中需要动态处理的(不论样式还是内容),我们都要有对应的数据模型
3.  控制：我们在视图中进行的某些操作都是去修改相关数据，然后React会按照最新的数据重新渲染视图

数据驱动视图渲染。视图中的表单内容改变想要修改数据需要开发者自己去写代码实现！
"单向驱动"

MVVM体系：Model数据层 + View视图层 + ViewModel 数据/视图监听层
1.  数据驱动视图的渲染：监听数据的更新，让视图重新渲染
2.  视图驱动数据的更改：监听页面表单中元素内容的改变，自动去修改相关数据
"双向驱动"