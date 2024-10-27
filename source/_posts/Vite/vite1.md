---
title: vite1
date: 2024-10-26 11:59:20
categories:
  - vite
tags:
  - 基础构建
---

# 依赖预构建

首先vite会找到对应的依赖，然后调用esbuild，将其他导入规范全部转换为esmodule规范，然后放到当前目录下面的node_modules/.vite/deps，同时对esmodule规范的各个模块进行统一的集成。所有的依赖路径会直接索引到这路径
下面去寻找。

这样就解决了三个问题:
1. 不同的第三方包会有不同的导出格式，这样重新集成就重新统一了导出格式
2. 对路径的处理上可以直接使用.vite/deps，方便导入路径的重写
3. 解决了网络多包传输的性能问题(也是原生esmodule规范不敢支持node_modules的原因之一)，有了依赖预构建之后
不管它有多少的额外export和import，vite都会尽可能的将它们进行集成最后只生成一个或几个模块

# 配置文件

```

第一种常用写法
import {defineConfig} from "vite"

export default defineConfig({
    .....
})

第二种常用写法
/**@type import ("vite").UserConfig*/
const viteConfig = {
   ......
}
```
通常这样写才会出现默认配置提示，而defineConfig是一个函数，所以才会提示默认配置项

## 环境配置

在vite中可以提前配置好base、dev、prod等不同的vite配置，而在vite主配置中则可以通过command传参来决定要启动那个配置项

## 环境变量配置

```
 const env = loadEnv(mode, process.cwd(), "");
 // 加载配置文件 process.cwd()会返回当前的工作目录
```

.env: 表示所有环境都需要的环境变量
.env.development: 开发环境
.env.production: 生产环境

当我们调用loadEnv的时候，它会做以下几件事
1. 直接找到.env文件不解释，并解析其中的环境变量并放进一个对象中
2. 会将传进来的mode这个变量直接对.env进行拼接，并根据提供的目录找寻相应的配置文件进行解析，然后放进一个对象中，并对基础的env文件中相同的变量进行覆盖

如果是在客户端中，也就是普通的js脚本当中,可以直接从import.meta.env中获取环境变量
其次vite还为我们做了一层拦截，为了区分普通变量和隐私变量，它会将VITE开头的环境变量送进import.meta.env中
如果想要修改该前缀可以使用envPrefix进行修改，可以在vite.config.js中进行修改