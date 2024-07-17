---
title: Animation
date: 2024-07-17 15:54:07
categories:
  - css
tags:
  - Transition
---

## 动画

`animation-name` 指定动画关键帧的名字(动画内属性)

`animation-duration` 指定动画变换的时间，负值无效(动画内属性)

`animation-timing-function` 指定动画变换的效果，确定是线性过渡还是平滑过渡
                            等模式(动画内属性)

`animation-direction` 动画执行的方向，方向是关键帧内部指定的属性同时也会反转
                      animation-timing-function属性

`animation-delay` 动画开始前需要等待的时间(动画外属性)

`animation-iteration-count` 动画执行的次数(只作用于动画内的属性)

`animation-fill-mode` 控制元素在动画外的状态

`animation-play-state` 定义动画的执行和暂停

## 关键帧

百分比的属性，其实代表的是时间的快慢，如果在相同的时间所执行动画的效果很多，其执行的速度就会被加快