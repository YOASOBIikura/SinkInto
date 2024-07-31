---
title: 3Dtransform
date: 2024-07-16 16:04:15
categories:
  - css
tags:
  - 2Dtransform
---

`perspective` 景深通常作用于该元素的子元素才有用，让元素有近大远小的效果，并
              且控制灭点的位置，景深越大其图中的灭点越远，元素的变形也就越小
              反之亦然

`perspective-origin` 景深基点，控制的是视角看向的位置

`transform` 3D旋转变换
            `rotateX` 按3维X轴进行旋转
            `rotateY` 按3维Y轴进行旋转
            `rotateZ` 按3维Z轴进行旋转
            `rotate3d(x,y,z,deg)` 按3维空间中的x,y,z确定的点进行旋转

            3D平移变换
            `translateZ` 按Z轴对元素进行平移操作X、Y轴与二维平移相同
            `translate3d` 按各个轴的参数对元素进行平移操作，X、Y轴可以
                          使用百分比但Z轴只能使用px参数进行设置

            3D缩放
            `scaleZ` 在三维空间单独使用是没有作用的，只有在组合变换的时候作
                     为其他变换的因子的时候有用
            `scale3d` 按照X、Y、Z轴设定的数值进行缩放

`transform-style` 用于营造有层级的3d舞台，并且作用于子元素

`backface-visibility` 用于设置元素背面是否显示