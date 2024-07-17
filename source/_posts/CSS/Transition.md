---
title: Transition
date: 2024-07-15 09:58:34
categories:
  - css
tags:
  - Transition
---

## Transition动画

注意：1. transition在元素首次渲染还没有结束的情况下是不会被触发的
     2. 在绝大部分样式切换时，如果变换函数的位置个数不相同也不会触发过渡(位置需要一样)

`transition-property` 默认值为all,表示所有可被动画的属性都表现出过渡动画，
                      同时也可以指定属性显示动画效果

`transition-duration` 动画效果的显示时间(以s或ms为时间单位)，也就是指定过渡
                      的时间，时间也可以通过列表进行指定，少于指定属性就复制之前指定的时间列表，多余就裁剪余下的时间列表

`transition-timing-function` 用于指定过渡的行为模式，是均匀时间过渡还是先
                             加速在减速等模式的指定

`transition-delay` 指定过渡动画开始前的的延迟时间

top,height 百分比参照与包含块的高度
left,margin,padding,with 百分比参照与包含块的宽度
translate(-50%, -50%) 百分比参照与自身的宽高
background-position 百分比参照与(图片区域-图片的位图像素值)
