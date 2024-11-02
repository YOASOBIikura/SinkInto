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

