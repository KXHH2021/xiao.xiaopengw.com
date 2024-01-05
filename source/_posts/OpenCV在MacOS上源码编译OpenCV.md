---
title: 在MacOS上源码编译OpenCV 
date: 2024-01-04 20:00:00
categories:
  - OpenCV
tags:
  - opencv_contril
  - Apache2.0
  - C++
  - Python
  - Java
  - MATLAB
  - Mac OS
cover: https://s2.loli.net/2024/01/05/JtUAnOgPxj8bopr.png
---

## 前言
在做视觉任务时，我们经常会用到开源视觉库OpenCV，OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库，它具有C++，Python，Java和MATLAB接口，并支持Windows，Linux，Android和Mac OS。 最近在项目中，我遇到了在MacOS上使用OpenCV需求，目前OpenCV官网上并没有提供OpenCV现成的安装包，因此在此处我们需要自己进行编译，所以在此处我们将结合``opencv_4.8.0``、``opencv_contril_4.8.0``，演示如何源码编译并使用

## 1. 下载项目源码
  首先下载项目源码，这里我们下载的是4.8.0，大家可以根据自己的需求进行下载，不过要尽量保证opencv、opencv_contril源码版本一致。通过下面代码我们进行源码下载：
```undefined
wget https://github.com/opencv/opencv/archive/4.8.0.zip
wget https://github.com/opencv/opencv_contrib/archive/refs/tags/4.8.0.zip
```
  下载完代码后，将代码文件解压到当前文件中，如下图所示：
![image.png](https://s2.loli.net/2024/01/05/9c5WJ4jmxpHh3tb.png)

## 2. 创建CMake编译文件
  OpenCV支持CMake编译，所以此处需要安装CMake，安装方式此处不做讲解。输入一下指令，打开并创建编译文件夹：
```undefined
cd opencv-4.8.0
mkdir build && cd build
```
  接下来输入CMake指令，进行CMake编译，此处需要注意三个路径：

- CMAKE_INSTALL_PREFIX=<install path>：<install path>表示编译好的OpenCV安装路径，可以指定到系统路径，也可以是自定义路径，此处设置为：/Users/ygj/3lib/opencv_4.8.0/include/opencv4/opencv2，注意这个路径，后续编译C++项目时会用到。

- OPENCV_EXTRA_MODULES_PATH=<model path>：<model path>表示扩展模块的路径，就是上文我们下载的opencv_contril_4.8.0文件，在此处设置为/Users/ygj/3lib/opencv_build/opencv_contrib-4.8.0/modules。

- PYTHON3_EXECUTABLE=<python path>：<python path>表示本计算机Python的安装路径，此处也可以不设置，主要就是设置要不要生成Python依赖库。如果设置了，需要开启BUILD_opencv_python2=ON或者BUILD_opencv_python3=ON，具体按照你的电脑中安装的Python版本决定。

设定好上面三个路径后，就可以在终端输入以下指令，进行CMake编译：
```undefined
cmake -DCMAKE_SYSTEM_PROCESSOR=arm64 -DCMAKE_OSX_ARCHITECTURES=arm64 -DWITH_OPENJPEG=OFF -DWITH_IPP=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=<install path> -D OPENCV_EXTRA_MODULES_PATH=<model path> -D PYTHON3_EXECUTABLE=<python path> -D BUILD_opencv_python2=OFF -D BUILD_opencv_python3=ON -D INSTALL_PYTHON_EXAMPLES=ON -D INSTALL_C_EXAMPLES=OFF -D OPENCV_ENABLE_NONFREE=ON -D BUILD_EXAMPLES=ON ..
```
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/795aed4e-b05c-4fe5-94b9-08fad2a3fc2d)
  编译完成后如下图所示，不过此处要注意一点，在编译时会下载相关的第三方库，要保证网络通畅，防止下载失败。
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/9208b2c8-1773-4498-9c32-e7ea1e3bffcb)

## 3. 编译安装
  上一步完成CMake编译后，就可以进行make编译了，只需要输入一下指令即可，-j8表示用8个核心进行编译，具体设置可以根据你的电脑进行设置，数值越大编译越快。
```undefined
make -j8
```
  编译完成后，如下图所示：

![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/9d31ebef-55d9-49d0-872a-75483532f0ea)



  接下来就是进行安装，只需要一下指令就可：
```undefined
make install
```
  安装完成后，会在你上文设置的安装路径下生成依赖文件，如下图所示：
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/3019482d-06e3-42e6-8cd3-943b45bfe265)


## 4. 案例测试
  首先创建一个新的C++文件main.cpp文件，在文件中添加以下代码：
```undefined
#include "opencv2/opencv.hpp"

int main(){
    std::cout<<"hello opencv!"<<std::endl;
    cv::Mat image = cv::imread("image.jpg");
    if (!image.empty())
    {
        std::cout << "image is OK!" << std::endl;
    }
    std::cout << "图像的宽度是：" << image.rows << std::endl;
    std::cout << "图像的高度是：" <<image.cols << std::endl;
    std::cout << "图像的通道数是：" << image.channels() << std::endl;
    cv::Mat image1;
    cv::cvtColor(image,image1,cv::COLOR_RGB2GRAY);
    cv::imshow("image",image);
    cv::imshow("image1",image1);
    cv::waitKey(0);
    std::cout<<"hello opencv!"<<std::endl;
    return 0;
}
```
  这一段代码主要是读取本地图片文件，获取并输出图片的基本信息，然后使用窗口将图片展示出来。

  此处编译方式采用CMake编译方进行编译，定义的CMakeLists.txt文件如下所示：
```undefined
cmake_minimum_required(VERSION 3.28)
project(opencv)
set(OpenCV_DIR /Users/ygj/3lib/opencv_4.8.0/lib/cmake/opencv4)
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV_DIR = ${OpenCV_DIR}")
message(STATUS "OpenCV_INCLUDE_DIRS = ${OpenCV_INCLUDE_DIRS}")
message(STATUS "OpenCV_LIBS = ${OpenCV_LIBS}")
include_directories(
    ${OpenCV_INCLUDE_DIRS}
)
add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} )
```
  在CMakeLists文件中，我们通过find_package(OpenCV REQUIRED)查找本计算机安装的OpenCV依赖库，但是需要在之前指定OpenCV的安装路径。写完Cmake文件后，在命令行中输入cmake .进行运行，输出结果如下图所示：
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/c447a8da-4235-49e4-90d1-101afd360258)


  可以看出，CMake已经成功找到了本计算机安装的OpenCV路径，并获取了项目编译所需要的所有信息。
如果CMake没有任何问题，接下来就进行项目编译，只需要输入make指令即可，输出如下所示：
![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/36425893-81a0-4297-84e1-aa2b92db6316)


  make之后，会在项目文件夹中生成一个main文件，接下来直接运行该文件，斌可以的到如下图所示的输出：

![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/0e89f5eb-edb5-448a-b300-2b6e8c04db50)


## 5. 总结
  在本项目中，我们实现了在MacOS系统上源码编译OpenCV，并在VS Code上使用OpenCV做了项目测试，最后成功实现了在MacOS系统上使用我们源码编译OpenCV的链接库，进行了图片处理。
