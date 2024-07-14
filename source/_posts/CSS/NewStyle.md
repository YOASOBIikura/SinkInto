---
title: NewStyle
date: 2024-07-13 14:28:05
categories:
  - css
tags:
  - NewStyle
---

## front

`opacity` 控制文字的透明度

`rgba(0,0,0,.num)` 控制颜色以及透明度

有了这两个属性可以更简单的实现背景透明或文字不透明，文字透明背景不透明的效果

`text-shadow` 用来个文字添加阴影，并且可以添加多层阴影，阴影之间用逗号隔开

`direction` 控制文字的排版方向并且通常需要配合`unicode-bibi`属性来形成镜像显示

`text-overflow` 文字超过显示自动省略(使用前提条件是盒子不能靠内容撑开)其中配套使用的属性有
                `white-space: nowrap` 超过不自动换行
                `overflow: hidden` 溢出自动省略 


---

## box

`box-shadow` 一共有五个属性 
             offset-x 右偏移
             offset-y 下偏移，设为负值则为相反的方向
             模糊度
             阴影大小
             反置属性，设置之后阴影会在容器内部展现

`resize` 可以设置盒子的大小拖动，且必须配`overflow: auto`使用，但会破坏页面一般不用

`border-size: boder-box` 可以实现padding等盒子属性在盒子内部撑开，而不会扩大盒子的长宽

`border-radius` 通过百分比来定义盒子的圆角，实际是是通过给定基于长度或宽度百分比的半径在盒子内部画圆，并在盒子内部扩大拉伸并在刚好与盒子相切
                的时候抹除其外角，设置的顺序分别是盒子的上右下左的设置顺序

`border-image-source` 通过图片代替默认边框样式，配和`border`属性一起使用

`border-image-slice` 通过对图片进行切割来对border的九宫格进行填充

`border-image-repeat` 通过slice的切割之后对图片在九宫格中进行平铺等其他布局操作

`border-image-width` 不设置默认就是border的宽度，设置后会改变source中设置图片的大小，但实际border宽度不变

`border-image-outset` 设置边框外扩，设置负值无效

---

## background

`background-origin` 由于背景图片是从padding box开始平铺，截取到border；所以可以设置背景设置的渲染位置

`background-clip` 设置开始剪图片的位置，可以是padding-box、border-box和content-box

`background-size` 设置背景图片的大小

---

## 线性渐变

渐变是图片

`background-image: linear-gradient(to (where), color, color)` 控颜色渐变需要两个以上的颜色参数，并在第一个to参数中设定渐变方向，也可以
                                                              在to参数上控制渐变角度，在前面加上`repeating`则可以设置重复渐变

`background-image: radial-gradient` 径向渐变，像球一样发散性的均匀渐变    