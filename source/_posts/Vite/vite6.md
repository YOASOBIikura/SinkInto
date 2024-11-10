---
title: vite6
date: 2024-11-10 15:34:12
categories:
  - vite
tags:
  - 分包策略
---

## 问题来源

浏览器的缓存策略

静态资源 => 名字没有变化，那么他就不会重新去拿xxx.js

在vite中每次更行dest中都会出现新的hash名字的js文件，但是我们业务代码虽然会经常变化但引用的第三方库的方法是不会经常变化的，这会导致第三方库文件的代码随着业务代码的更新被平繁更新获取影响性能！

分包就是把一些不会常规更新的文件进行单独的打包处理独立与业务代码之外

```
"build": {
    "rollupOptions": {
        "output":{
            "manualChunks": (id: string) => {
                if (id.includes("node_modules")){
                    return "vendor";
                }
            }
        }
    }
}
```