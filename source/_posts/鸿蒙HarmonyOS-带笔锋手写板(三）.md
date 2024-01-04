---
title: 鸿蒙HarmonyOS-带笔锋手写板(三）
date: 2024-1-4 16:53:00
categories:
  - HarmonyOS
tags:
  - HarmonyOS
---

###### 一、效果图

![](https://raw.githubusercontent.com/KXHH2021/seveimg/main/img/202401041651779.png)

###### 二、实现方法

参考文章：

[支持笔锋效果的手写签字控件\_android 写字板如何兼容笔峰-CSDN博客](https://blog.csdn.net/gs12software/article/details/86008400 "支持笔锋效果的手写签字控件_android 写字板如何兼容笔峰-CSDN博客")

[安卓画笔笔锋的实现探索（一） - 简书](https://www.jianshu.com/p/6746d68ef2c3 "安卓画笔笔锋的实现探索（一） - 简书")

主要代码：

        核心思想在于通过插值，在两点之间逐渐绘制多个椭圆，从而呈现出笔锋的效果。

  `drawLine` 方法是一段用于在2D渲染画布上绘制线条并赋予其笔锋效果的代码。

        在代码中，`curDis` 用于计算起始点和结束点之间的欧几里德距离。`steps` 根据距离计算出线条上需要绘制的点的数量。`deltaX`, `deltaY`, `deltaW` 分别表示 x 坐标、y 坐标和宽度每一步的增量。
    
        通过 `for` 循环，在两点之间进行插值，绘制多个椭圆，以模拟笔锋效果。每一步循环中，创建一个椭圆对象 (`oval`)，并设置其位置调用 `oval` 方法绘制椭圆。
    
        最后，更新坐标和宽度的增量，为绘制下一个椭圆做准备。

```undefined
private drawLine(canvas: CanvasRenderingContext2D, x0: number, y0: number, w0: number, x1: number, y1: number, w1: number): void {

const curDis: number = Math.hypot(x0 - x1, y0 - y1);

let steps: number;

steps = 1 + Math.floor(curDis / 2);

let deltaX: number = (x1 - x0) / steps;

let deltaY: number = (y1 - y0) / steps;

let deltaW: number = (w1 - w0) / steps;

let x: number = x0;

let y: number = y0;

let w: number = w0;

for (let i = 0; i < steps; i++) {

const oval: MyRect = new MyRect();

const top: number = y - w / 2.0;

const left: number = x - w / 4.0;

const right: number = x + w / 4.0;

const bottom: number = y + w / 2.0;

oval.set(left, top, right, bottom);

this.oval(canvas, oval);

x += deltaX;

y += deltaY;

w += deltaW;

}

}
```

        绘制椭圆的方法，在安卓中可以用canvas.drawOval()方法，在HarmonyOS中需要通过canvas.ellipse()方法来实现：

```undefined
private oval(canvas: CanvasRenderingContext2D, roundedCircleBox: MyRect): void {

canvas.beginPath();

canvas.ellipse(

roundedCircleBox.left + (roundedCircleBox.right - roundedCircleBox.left) / 2,

roundedCircleBox.top + (roundedCircleBox.bottom - roundedCircleBox.top) / 2,

(roundedCircleBox.right - roundedCircleBox.left) / 2,

(roundedCircleBox.bottom - roundedCircleBox.top) / 2,

0,

0,

2 * Math.PI);

canvas.fill();

canvas.closePath();

}
```

###### 三、开源地址

[NotePad: HarmonyOS ArkTS带笔锋手写板应用](https://gitee.com/liu-haikang/note-pad "NotePad: HarmonyOS ArkTS带笔锋手写板应用")