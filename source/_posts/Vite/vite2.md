---
title: vite2
date: 2024-10-27 12:35:03
categories:
  - vite
tags:
  - css配置
---


## modules篇

在vite.config.js中我们通过css属性去控制整个vite对于css的处理行为

- localConvention: 修改生成的配置对象的key的展示形式(驼峰还是中划线)

- scopeBehaviour: 配置当前的模块化行为是模块化还是全局化 (有hash就是开启了模块化的标志，使标志唯一，默认是模块化。设置为全局化"global"则会关闭模块化的hash)-> 一般默认设置local

- generateScopedName: 生成类名的规则，订制生成类名的展示格式 -> 一般默认设置

- hashPrefix: 生成的hash会根据你的类名去进行生成，如果想要生成的hash更加独特可以配置这个属性，该属性会参与到最终的hash生成 -> 一般默认设置

- globalModulePaths: 配置不想进行css模块化的文件路径