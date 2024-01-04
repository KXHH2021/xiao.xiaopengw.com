---
title: HarmonyOS应用开发-手写板(二）
date: 2024-1-4 16:55:38
categories:
  - HarmonyOS
tags:
  - HarmonyOS
---

一、先上效果图：

![](https://img-blog.csdnimg.cn/direct/accc292bcf124f3eb310a3e62701eb5a.gif)

二、上代码

```undefined
import picker from '@ohos.file.picker';

import fs from '@ohos.file.fs';

import buffer from '@ohos.buffer';

@Entry

@Component

struct CanvasPage {

@State pathCommands: string = '';

canvas: CanvasRenderingContext2D = new CanvasRenderingContext2D();

path2D: Path2D = new Path2D();

build() {

Column() {

Row() {

Button("清空")

.margin(10)

.onClick(() => {

this.path2D = new Path2D();

this.canvas.clearRect(0, 0, this.canvas.width, this.canvas.height);

})

Button("保存")

.onClick(() => {

this.saveImage();

})

}

Canvas(this.canvas)

.width('100%')

.height('100%')

.onTouch((e) => {

this.onTouchEvent(e);

})

}

}

onTouchEvent(event: TouchEvent) {

let x = (event.touches[0].x);

let y = (event.touches[0].y);

switch (event.type) {

case TouchType.Down:

this.path2D.moveTo(x, y);

break;

case TouchType.Move:

this.path2D.lineTo(x, y);

this.canvas.strokeStyle = "#0000ff";

this.canvas.lineWidth = 5;

this.canvas.stroke(this.path2D);

break;

default:

break;

}

}

saveImage() {

let imageStr = this.canvas.toDataURL().split(',')[1];

let uri = '';

try {

let PhotoSaveOptions = new picker.PhotoSaveOptions();

PhotoSaveOptions.newFileNames = ['11111.png'];

let photoPicker = new picker.PhotoViewPicker();

photoPicker.save(PhotoSaveOptions).then((PhotoSaveResult) => {

uri = PhotoSaveResult[0];

let file = fs.openSync(uri, fs.OpenMode.READ_WRITE);

const decodeBuffer = buffer.from(imageStr, 'base64').buffer;

fs.writeSync(file.fd, decodeBuffer);

fs.closeSync(file);

}).catch((err: Error) => {

console.error(err + '');

})

} catch (e) {

console.error(e);

}

}

}
```

在这段代码中，根据功能划分，主要涵盖了三个关键操作：绘制路径、清空画布和保存画布。

#### 一、绘制路径

        在绘制路径方面，代码通过Canvas执行同时借助Path2D定义了具体的绘制路径。手写路径的生成通过记录手指按下和移动的位置实现。具体操作包括：

```undefined
this.path2D.moveTo(x, y)

this.path2D.lineTo(x, y)
```

#### 二、清空画布

清空画布的操作分为两步：

1.将路径置空

```undefined
this.path2D = new Path2D();
```

2.清空canvas

```undefined
this.canvas.clearRect(0, 0, this.canvas.width, this.canvas.height);
```

#### 三、保存画布

        保存画布的过程主要由`saveImage`方法完成，依赖于`@ohos.file.picker`组件，调用系统的图片保存功能。具体步骤包括：

1.  通过`PhotoViewPicker`的`save`方法获取用户选择的保存文件路径。
2.  利用Canvas的`toDataURL()`方法将Canvas转换为base64字符串形式的图片。
3.  通过`@ohos.buffer`将base64字符串转换为buffer。
4.  最终，通过`@ohos.file.fs`将buffer写入文件，文件的路径为之前获取的保存路径。

        这一系列步骤成功实现了将绘制的图像保存为一个完整的图片文件。整体而言，代码清晰地展示了绘制路径、清空画布和保存画布的功能实现。