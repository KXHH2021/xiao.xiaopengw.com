---
title: 【OpenCV】在Mac OS 上使用 EmguCV
date: 2024-01-05 21:00:00
categories:
  - OpenCV
tags:
  - Python
  - Java
  - Visual Studio Code
  - .NET 6.0
description: OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库
cover: https://s2.loli.net/2024/01/06/PuUQyF8ASjD43ik.png
---
## 【OpenCV】在Mac OS 上使用 EmguCV
## 前言
OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库，它具有C++，Python，Java和MATLAB接口，并支持Windows，Linux，Android和Mac OS。 Emgu CV是OpenCV图像处理库的跨平台 .Net 包装器。允许从 .NET 兼容语言调用OpenCV函数。但是网上目前关于在Mac OS上使用EmguCV的教程较少，而我后续推出的OpenVINO C# API项目将支持Mac OS系统，为了大家后续能够使用，特出一期教程来演示一下Mac OS上使用EmguCV。

## 1. 项目环境
- **编码环境：Visual Studio Code**
- **程序框架：.NET 6.0**

目前在Mac OS上使用C#语言官方提供了编译Visual Studio for Mac，但是根据官方发布的通知后续将不再支持该软件更新，后续将全部转移到Visual Studio Code平台，所以在此处我们演示使用Visual Studio Code进行演示。而代码的运行与配置使用dotnet指令实现。

关于Visual Studio Code以及.NET的安装方式可以参考一下官方教程：

在 macOS 上安装 .NET、Visual Studio Code on macOS。

## 2. 创建控制台项目

此处使用dotnet指令创建新项目，在Visual Studio Code的终端中输入一下指令：
```undefined
dotnet new console --framework net6.0 --use-program-main -o test_emgucv
```
如下图所示，在终端中输入以下指令后，会自动创建新的项目以及项目文件夹。

![image.png](https://s2.loli.net/2024/01/06/3gKH6EFevfPZ17A.png)

在创建好项目后，我们进行一下项目测试，依次输入以下指令，最后输出如下图所示：
```undefined
cd test_emgucv
dotnet run
```
![image.png](https://s2.loli.net/2024/01/06/i9WHTx2GqwEOjc5.png)
## 3. 添加 Nuget Package 程序包
Emgu CV是一个可以跨平台使用的程序包，并且官方也提供了编译好的程序包，用户可以根据自己的平台进行安装。在Mac OS上，主要需要安装一下两个包，分别是Emgu.CV的官方程序包以及Emgu.CV的运行依赖包。
```undefined
dotnet add package Emgu.CV
dotnet add package Emgu.CV.runtime.mini.macos
```
安装完上面两个安装包后，项目的配置的文件中会增加下面两个配置。
```undefined
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Emgu.CV" Version="4.8.1.5350" />
    <PackageReference Include="Emgu.CV.runtime.mini.macos" Version="4.8.1.5350" />
  </ItemGroup>

</Project>
```
接下来运行dotnet run，检验项目中是否包含所需要的配置文件：Emgu.CV.dll、runtimes/osx/native/libcvextern.dylib。打开项目运行生成的文件夹bin/{build_config}/{dotnet_version}/，在本项目中是bin/Debug/net6.0/文件夹，如下图所示：

![image.png](https://s2.loli.net/2024/01/06/dH4jVmEkW3RnrcO.png)

通过该图可以看出，在本项目中只有Emgu.CV.dll文件，并没有runtimes/osx/native/libcvextern.dylib文件，因该文件需要我们自行配置。首先是需要找到该文件，该文件主要是在Emgu.CV.runtime.mini.macos程序包中，如下图所示：

![image.png](https://s2.loli.net/2024/01/06/YnyTiLM7UlVFIKt.png)

接下来就是创建runtimes/osx/native/文件夹，然后将该文件放在该文件夹下即可。如下图所示：

![image.png](https://s2.loli.net/2024/01/06/6BWErUp3nZ9Cc1X.png)

## 3. 测试应用
  最后我们编写项目代码进行测试，如下面代码所示：
```undefined
using System;
using Emgu.CV;
using Emgu.Util;
using Emgu.CV.Structure;
using Emgu.CV.CvEnum;
namespace test_emgucv 
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Mat image = CvInvoke.Imread("image.jpg");
            Mat image2=new Mat();
            if (!image.IsEmpty)
            {
                Console.WriteLine("srcImg is OK!");
            }
            Console.WriteLine("图像的宽度是：{0}",image.Rows);
            Console.WriteLine("图像的高度是：{0}", image.Cols);
            Console.WriteLine("图像的通道数是：{0}", image.NumberOfChannels);
            CvInvoke.Imshow("src", image);
            CvInvoke.CvtColor(image, image2, ColorConversion.Bgr2Gray);//转为灰度图像
            CvInvoke.Imshow("src1", image2);
            CvInvoke.WaitKey(0);
            CvInvoke.DestroyAllWindows();//销毁所有窗口
        }
    }
}
```
  项目代码运行后，最后呈现效果如下图所示：

![image](https://github.com/KXHH2021/xiao.xiaopengw.com/assets/88917933/9052b442-4df9-4ebf-90a4-f482a1817f41)


## 4.总结
  在本次项目中，我们成功实现了在Mac OS上使用EmguCV，并成功配置了EmguCV依赖库，实现了在.NET 6.0环境下使用C#语言调用EmguCV库，实现的图片数据的读取以及图像色彩转换，并进行了图像展示。
