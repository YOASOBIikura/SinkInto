---
title: vue1
date: 2024-11-14 17:35:21
categories:
  - vue
tags:
  - 源头
---

# 响应式

函数 <=> 函数在运行过程中用到的标记数据 建立对应关系

1. 监听数据的读取和修改 => 才能知道函数何时重新执行
2. 如何知晓数据对应的函数

所谓响应式就是第一点的意思，现在较常用的实现方式就是proxy
```
const proxy = reactive(data)

function reactive(target){
  return new proxy(target, {
    get(target, key){
      // 依赖搜集
      track(target, key)
      return target[key]; //返回对象的相应属性值
    },
    set(target, key, value){
      // 派发更新
      trigger(target, key)
      return Reflect.set(target, key, value) //设值对象属性
    }
  })
}
```

第二点找到对应的数据对应的函数，实际上就是依赖搜集


# WeakMap Map的区别

1. WeakMap的键只能是对象，但是Map的键可以示任意类型
2. Map的键是强引用，而WeakMap的键是弱引用如果没有依赖则会被垃圾回收掉
3. Map是可以迭代的，但是WeakMap不行
