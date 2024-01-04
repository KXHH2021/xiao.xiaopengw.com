---
title: HarmonyOS应用开发-手写板
date: 2024-1-4 17:03:53
categories:
  - HarmonyOS
tags:
  - HarmonyOS
  - 标签2
---

一、先上效果图：

![](https://img-blog.csdnimg.cn/direct/bd4a127a65bb4dbc949ba877c2512f07.gif)

二、上代码

```undefined
@Entry

@Component

struct Index {

@State pathCommands: string = '';

build() {

Column() {

Button("清空")

.onClick(() => {

this.pathCommands = '';

})

Flex() {

if (this.pathCommands != '') {

Path().commands(this.pathCommands).strokeWidth(5).fill('none').stroke(Color.Blue)

}

}.onTouch((event: TouchEvent) => {

this.onTouchEvent(event)

}).width('100%').height('100%')

}

}

onTouchEvent(event: TouchEvent) {

let x = vp2px(event.touches[0].x);

let y = vp2px(event.touches[0].y);

switch (event.type) {

case TouchType.Down:

this.pathCommands += 'M' + x + ' ' + y;

break;

case TouchType.Move:

this.pathCommands += 'L' + x + ' ' + y;

break;

default:

break;

}

}

}
```

在这个代码中，我们构建了一个手势[绘图应用](https://so.csdn.net/so/search?q=%E7%BB%98%E5%9B%BE%E5%BA%94%E7%94%A8&spm=1001.2101.3001.7020)。以下是关键部分的解释：

1.  **@Entry和@Component注解：** 这两个注解用于标识这个类是一个入口点并且是一个组件。在HarmonyOS中，这是定义页面的标准方式。
    
2.  **@State注解：** 在HarmonyOS中，@State注解同样用于声明状态。在这里，我们声明了一个[字符串类型](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B&spm=1001.2101.3001.7020)的`pathCommands`，用于存储手势绘制的路径。
    
3.  **build()函数：**这个函数定义了HarmonyOS页面的结构，包括清空按钮和用于展示绘图路径的组件。
    
4.  **onTouchEvent函数：** 这个函数处理触摸事件，根据手指按下和移动的位置，将相应的绘制命令添加到路径中，实现了手势绘制的功能。