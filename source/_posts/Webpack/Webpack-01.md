---
title: Webpack-01
date: 2025-01-18 12:51:33
categories:
  - Webpack
tags: 
  - 基础配置
---

## 基础配置
webpack时期是一个平台，在平台中我们可以融入/配置各种需要处理的规则
- mode: 打包模式(开发环境和生产环境)
- entry: 入口(webpack就是从入口开始，根据导入的依赖分析模块之间的关系，从而按照依赖关系进行打包)
- output: 出口(打包完成文件所在的路径)
- loader: 加载器(一般都是用于实现代码的编译的，但是想编译啥代码需要安装对应的加载器，并且完成相关的规则配置)
- plugin: 插件(处理的需求比较多，如压缩、编译HTML、清空打包等等)
- resolve: 解析器
- optimization: 优化项
- devServer: 配合webpack-dev-server，在本地启动Web服务，实现项目预览以及跨域处理....
webpack是基于Node.js进行打包的
支持CommonJS规范，同时也支持ES6Module规范，并且在webpack中进行混用。

-----

webpack支持零配置打包：需要写任何配置使用默认配置进行打包
- 默认去找src/index.js，将其作为打包的入口，进行打包!! 打包后的内容输出到dist/main.js中！！

## 自定义打包规则
```
const HtmlWebPackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
    // production生产环境，打包后JS会自动压缩
    // development开发环境则不会
    mode: 'dev'
    // 指定入口
    entry: './ssrc/index.js',
    // 多入口情况
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    // 指定出口
    output: {
        // 打包后文件的名字
        // 通过添加hash,代码变化后打包文件也会跟着变化，有助于强缓存
        filename: 'main.[hash].js'
        // 设置打包路径
        path: path.resolve(__dirname, './dist')
    },
    // 使用插件
    plugins: [
        // 打包编译HTML的插件，自动导入需要的css以及js文件
        new HtmlWebPackPlugin({
            // 指定页面模版
            template: './public/index.html',
            // 打包后名字
            filename: 'index.html',
            // 是否压缩
            minify:true,
            // 指定导入的资源名称
            chunks: ["index"]
        }),
        // 删除之前打包的文件
        new CleanWebpackPlugin()
    ]
}
```

