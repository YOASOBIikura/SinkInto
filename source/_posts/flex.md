---
title: flex
date: 2024-07-19 10:34:48
categories:
  - css
tags:
  - flex
---

## 容器

`flex-wrap` 控制容器是单行/单列还是多行/多列

`align-content` 侧轴方向有多余空间时，如何排布每一行或列

`flex-flow` flex-direction和flex-wrap的简写属性

`order` 控制子组件在容器中的排列顺序，往往给定值越小越靠前

`align-self` 控制单个组件在容器中侧轴的位置

`flex-grow` 控制容器组件内部的拉伸因子
            可用空间 = 容器大小 - 所有相邻组件的flex-basis的总和
            可扩展空间 = 可用空间 / 所有相邻组件的flex-grow的总和
            每项伸缩大小 = 伸缩基准值 + (可扩展空间 * flex-grow值)

`flex-shrink` 控制容器内部组件的收缩因子(flex-wrap默认值一行的情况下才会有作用)
              计算收缩因子和基准值乘的总和
                    每个组件的flex-shrink和flex-basis的总和
              计算收缩因数
                    收缩因数 = (组件设置的收缩因子 * 组件基准值) / 第一步计算总和
              移除空间计算
                    移除空间 = 项目收缩因数 * 负溢出空间