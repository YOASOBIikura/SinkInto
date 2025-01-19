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
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CssMinimizerWebpackPlugin = require('css-minimizer-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')

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
        new CleanWebpackPlugin(),
        new MiniCssExtractPlugin(
            // 打包后css文件的名字
            filename: 'main.[hash:8].css'
        ),
    ],
    // dev-server
    devServer: {
        host: '127.0.0.1',
        port: 3000,
        open: true, // 自动开发浏览器
        hot: true, // 热更新
        compress: true, // 开启服务器端的GZIP压缩
        // 跨域代理配置
        proxy: {
           "/Xxx": {
                target: "", // 代理的真正的服务器地址
                pathRewritr: { // 地址重写 因为真正的地址是没有/XXX字符串的
                    "^/Xxx": ""
                },
                changeOrigin: true, // 修改请求头中的origin信息
                ws: true, // 支持websoket通信机制
           } 
        } 
    },
    // loader加载器 顺序 从上到下 从右到左
    module: {
        rules: [{
            test:  /\(.css|less)$/, // 给予正则匹配那些文件需要处理
            use: [
                MiniCssExtractPlugin.loader, //抽离css代码
                "style-loader", // 把css以内嵌式导入到页面
                "css-loader", // 处理特殊语法
                {
                    loader: "postcss-loader",
                    options: {
                        postcssOptions: {
                            plugins: {
                                require("autoprefixer")
                            }
                        }
                    }
                }, // 配合autoprefixer&browserlist给css3属性设置前缀
                "less-loader", // 把less编译为css
            ],
        },{
            test: '/\.js$/',
            use:[
                "babel-loader"
            ],
            // 设置编译时忽略的文件和指定的编译目录
            include: path.resolve(__dirname, 'src')
            exclude: /node_modules/
        },{
            test: /\.(png|jpe?g|gif)$/i,
            type: 'javascript/auto', // webpack5 需要
            use: [{
                loader: 'url-loader',
                options: {
                    // 把指定大小内的图片转换为base64(小于200Kb则需要Base64)
                    limit: 200 * 1024,
                    esModule: true,
                    // 编译后没有base64的图片，编译输出的路径和名称
                    name: 'images/[name].[hash].[ext]'
                }
            }]
        }]
    },
    // 设置打包的最大资源大小
    performance: {
        maxAssetSize: 100 * 1024 * 1024,
        maxEntrypointSize: 100 * 1024 *1024
    }
    // 优化项
    optimization: {
        // 设置压缩方式
        new CssMinimizerWebpackPlugin()
        new TerserPlugin()
    },
    // 解析器
    resolve: {
        // 别名
        alias: {
            '@': path.resolve(__dirname, './src')
        }
    }
}
```

webpack-dev-server 项目开发流程
1. 基于Node在客户端本地启动一个web服务器，帮助开发者预览开发作品
2. 项目打包后放在虚拟内存中
3. 启动web服务器从虚拟内存中获取打包的内容进行实时预览
4. 热更新(代码修改后会实时进行打包编译自动刷新浏览器，渲染最新的效果)
5. 启动web服务器，可以作为数据跨域请求的代理服务器，也就是可以实现Proxy跨域代理



