---
title: 【OpenCV】在MacOS上使用OpenCvSharp
date: 2024-01-06 22:00:00
categories:
  - OpenCV
tags:
  - MacOS
  - OpenCvSharp
description: OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库，它具有C++，Python，Java和MATLAB接口
cover: https://s2.loli.net/2024/01/06/K3ladXhYPT7SFpz.png
---
## 【OpenCV】在MacOS上使用OpenCvSharp
## 前言
  OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库，它具有C++，Python，Java和MATLAB接口，并支持Windows，Linux，Android和Mac OS。OpenCvSharp是一个OpenCV的 .Net wrapper，应用最新的OpenCV库开发，使用习惯比EmguCV更接近原始的OpenCV，该库采用LGPL发行，对商业应用友好。
## 1. 项目环境
- **编码环境：Visual Studio Code**
- **程序框架：.NET 6.0**

目前在Mac OS上使用C#语言官方提供了编译Visual Studio for Mac，但是根据官方发布的通知后续将不再支持该软件更新，后续将全部转移到Visual Studio Code平台，所以在此处我们演示使用Visual Studio Code进行演示。而代码的运行与配置使用dotnet指令实现。

于Visual Studio Code以及.NET的安装方式可以参考一下官方教程：

在 macOS 上安装 .NET、Visual Studio Code on macOS。

## 2. 创建控制台项目

此处使用dotnet指令创建新项目，在Visual Studio Code的终端中输入一下指令：
```undefined
dotnet new console --framework net6.0 --use-program-main -o test_opencvsharp
```
如下图所示，在终端中输入以下指令后，会自动创建新的项目以及项目文件夹。
![image.png](https://s2.loli.net/2024/01/06/sivnhfkWAZocgup.png)

在创建好项目后，我们进行一下项目测试，依次输入以下指令，最后会得到输出："Hello, World!"：
```undefined
test_opencvsharp
dotnet run
```
## 3. 添加 Nuget Package 程序包
OpenCvSharp4是一个可以跨平台使用的程序包，并且官方也提供了编译好的程序包，用户可以根据自己的平台进行安装。在Mac OS上，主要需要安装一下两个包，分别是OpenCvSharp4的官方程序包以及OpenCvSharp4的运行依赖包。
```undefined
dotnet add package OpenCvSharp4
dotnet add package OpenCvSharp4.runtime.osx_arm64 --prerelease
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
    <PackageReference Include="OpenCvSharp4" Version="4.8.0.20230708" />
    <PackageReference Include="OpenCvSharp4.runtime.osx_arm64" Version="4.8.1-rc" />
  </ItemGroup>

</Project>
```
emsp; 接下来运行dotnet run，检验项目中是否包含所需要的配置文件：OpenCvSharp.dll、runtimes/osx-arm64/native/。打开项目运行生成的文件夹bin/{build_config}/{dotnet_version}/，在本项目中是bin/Debug/net6.0/文件夹，如下图所示：

![image.png](https://s2.loli.net/2024/01/06/JizSvfxebd47mpW.png)

可以看出，在程序运行后，安装的程序包中所有项目都已经加载到当前项目中，如果出现缺失，就需要找到程序包位置，将该文件复制到指定路径。

## 3. 测试应用
  最后我们编写项目代码进行测试，如下面代码所示：
```undefined
using System;
using OpenCvSharp;
namespace test_opencvsharp 
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Mat image = Cv2.ImRead("image.jpg");
            Mat image2=new Mat();
            if (image!=null)
            {
                Console.WriteLine("srcImg is OK!");
            }
            Console.WriteLine("图像的宽度是：{0}",image.Rows);
            Console.WriteLine("图像的高度是：{0}", image.Cols);
            Console.WriteLine("图像的通道数是：{0}", image.Channels());
            Cv2.ImShow("src", image);
            Cv2.CvtColor(image, image2, ColorConversionCodes.RGB2GRAY);//转为灰度图像
            Cv2.ImShow("src1", image2);
            Cv2.WaitKey(0);
            Cv2.DestroyAllWindows();//销毁所有窗口
        }
    }
}
```
项目代码运行后，最后呈现效果如下图所示：

![image.png](https://s2.loli.net/2024/01/06/K3ladXhYPT7SFpz.png)

## 4. 总结
  在本次项目中，我们成功实现了在Mac OS上使用OpenCvSharp，并成功配置了OpenCvSharp依赖库，实现了在.NET 6.0环境下使用C#语言调用OpenCvSharp库，实现的图片数据的读取以及图像色彩转换，并进行了图像展示。
