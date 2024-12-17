---
title: vite3
date: 2024-10-29 22:14:38
categories:
  - vite
tags:
  - 静态资源配置
---

## vite加载静态资源

```
resolve:{
  alias:{
    "@":path.resolve(__dirname, "./src")
  }
}
```
设置文件访问路径别名，之后就可以用@来代替src这个目录

## build构建

- rollupOptions: 配置rollup的一些构建策略,在rollup里面，hash代表将你的文件名和文件内容进行组合计算得来的结果

- assetInlineLimit: 配置图片的最大容量，超过则打包成base64字符

- outDir: 输入路径文件夹

- assetsDir: 静态资源路径文件夹
                  