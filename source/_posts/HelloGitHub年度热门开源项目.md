---
title: 年度热门开源项目：HelloGitHub
date: 2024-02-12 12:00:00
categories:
  - HelloGitHub
tags:
  - HelloGitHub
  - selenuium
  - 
description: HelloGitHub 年度盘点，为了满足不同读者的需求，下了“大力气”将内容分为 Top10 和 精选 两部分
cover: https://s2.loli.net/2024/02/02/P623YoUdqTXL1pf.png
---

## HelloGitHub成为2023年度热门开源项目


在过去的一年里，「HelloGitHub 月刊」一共分享了 520 个开源项目。始终秉持着分享 GitHub 上有趣、入门级开源项目的初心，一直在路上，不断探索、发现和分享着那些令人惊叹的开源项目。

## 一、Top10

这里是 HelloGitHub 上最受欢迎的 10 个开源项目，筛选和排序是综合了用户的浏览、点赞、收藏和评论等数据，100% 来自用户的选择。

HelloGitHub 开源社区 现已支持点赞、收藏、评分、评论开源项目等功能，快去给你喜欢的开源项目点赞吧！

# 1、类似 selenuium 的网页自动化工具
![image.png](https://s2.loli.net/2024/02/02/xt5aLOv96Cp2rIu.png)

这是一款基于 Python 的网页自动化工具，支持 Chrome 和 Edge 等 Chromium 内核的浏览器。它将控制浏览器和收发请求两大功能合二为一，并提供了统一、简洁的接口，简单易用十分容易上手。该项目 v3.x 版本推出了 WebPage 摆脱对 selenium 的依赖，重新开发了底层逻辑，具有速度快、不易被网站识别、无需为不同版本浏览下载驱动等特点。

**用户评价**：非常好用，15 分钟的工作量 10 秒搞定。
```
from DrissionPage import SessionPage
from re import search

# 以s模式创建页面对象
page = SessionPage()
# 访问目标网页
page.get('https://www.starbucks.com.cn/menu/')

# 获取所有class属性为preview circle的元素
divs = page.eles('.preview circle')
# 遍历这些元素
for div in divs:
    # 用相对定位获取当前div元素后一个兄弟元素，并获取其文本
    name = div.next().text

    # 在div元素的style属性中提取图片网址并进行拼接
    img_url = div.attr('style')
    img_url = search(r'"(.*)"', img_url).group(1)
    img_url = f'https://www.starbucks.com.cn{img_url}'

    # 执行下载
    page.download(img_url, r'.\imgs', rename=name)
```
GitHub 地址→→→→https://github.com/g1879/DrissionPage


# 2、用玩 RPG 游戏的方式养成好习惯
![image.png](https://s2.loli.net/2024/02/02/wE17ORLS2AqoYW3.png)

这是一款养成类 RPG 游戏，当你完成一个现实中的待办事项后，会获得相应的经验和金币。随着你的等级提升，将会开启更多的玩法，比如购买装备、孵化宠物、职业、专属技能、组队打副本等。

**用户评价**：用了一会，感觉还挺不错的。不充值不影响正常使用，这点很难得。

![image.png](https://s2.loli.net/2024/02/02/XRZyaeTEL69tmoi.png)

GitHub 地址→→→→https://github.com/HabitRPG/habitica


# 3、全面的 Leetcode 算法解题指南

![image.png](https://s2.loli.net/2024/02/02/UqhTAiRN53WbrEa.png)
该项目包含 LeetCode、《剑指 Offer》、《程序员面试金典》等题目的相关题解，题解有 Java、Python、C++、Go、TypeScript、Rust 等多种编程语言实现。

**用户评价**：这个确实不错，很早就看这个然后面进了微软。

![image.png](https://s2.loli.net/2024/02/02/eOEHd4vfFhLKVqw.png)

GitHub 地址→→→→https://github.com/doocs/leetcode


# 4、从 0 到 1 数据库内核实战教程
![image.png](https://s2.loli.net/2024/02/02/KO9LzbYvugn8fQh.png)

该项目是 OceanBase 团队基于华中科技大学数据库课程原型，联合多所高校重新开发的、从零上手数据库的学习项目。它结构简单、代码简洁，不仅有文字讲解和视频教程，还有由浅入深的题目。通过理论+实战的方式，帮忙初学者迅速掌握内核模块功能和协同关系，提高工程编码能力，有助于在面试和工作中脱颖而出。

**用户评价**：这个项目确实挺赞的, 很好的帮我们从理论到实践来学习数据库。

![image.png](https://s2.loli.net/2024/02/02/T6ZsltVx5y8WLdC.png)

GitHub 地址→→→→https://github.com/oceanbase/miniob

# 5、AirDrop 的开源替代方案
![image.png](https://s2.loli.net/2024/02/02/dRnFANTZVO3XQ1h.png)

这个项目可以通过本地网络与附近的设备，安全地共享文件和消息，此过程不需要互联网，不需要外部服务器，支持 Windows、Linux、macOS、Android、iOS 设备。

**用户评价**：这个非常棒，为数不多的性能好，操作方便。

![image.png](https://s2.loli.net/2024/02/02/mF91eba2p5vG4rC.png)

GitHub 地址→→→→https://github.com/localsend/localsend

# 6、一款 Rust 写的像素风 RPG 游戏
![image.png](https://s2.loli.net/2024/02/02/ViZgpGwRTJYrtOB.png)

它的灵感来自《塞尔达传说：旷野之息》、《矮人要塞》和《我的世界》等游戏。虽然这款游戏的画质低，但拥有广阔的开放世界，玩家在游戏里可以打造道具、合成物品、战斗、升级、驯养宠物，还可以探索地牢洞穴、在空中滑翔、与 NPC 交易。

**用户评价**：CudeWorld 开源版。

GitHub 地址→→→→https://github.com/veloren/veloren

# 7、适合所有阶段开发者的 Docker 教程
![image.png](https://s2.loli.net/2024/02/02/yvnEslGAtq1DwWM.png)

该教程的内容分为初、中、高三个级别，适合所有阶段的 Docker。内含 500 个动手实验，以及 Docker 和 Docker Compose 小抄，这一切全部开源且分文不取。

![image.png](https://s2.loli.net/2024/02/02/v6BCEMT9sOF3RzY.png)

GitHub 地址→→→→https://github.com/collabnix/dockerlabs

# 8、一款强大的 Nginx 可视化管理平台
![image.png](https://s2.loli.net/2024/02/02/GFXTqbLhfJNQv2a.png)

它开箱即用支持 Docker 一键部署，可以让用户通过 Web 界面在线配置、管理 Nginx 服务，支持转发、重定向、SSL 证书、高级配置等功能。

**用户评价**：很好用的，非常棒！

![image.png](https://s2.loli.net/2024/02/02/EVna9ibkyeYcLr1.png)

GitHub 地址→→→→https://github.com/NginxProxyManager/nginx-proxy-manager

# 9、实时直播和视频 AI 换脸程序
![image.png](https://s2.loli.net/2024/02/02/5qoSbHYQwRexOGA.png)

该项目可以对摄像头和本地视频文件中的人物，进行实时 AI 换脸，可用于 PC 直播、视频等场景。

GitHub 地址→→→→https://github.com/iperov/DeepFaceLive

# 10、可视化全球天气实况的项目
![image.png](https://s2.loli.net/2024/02/02/bn8yBzqjDTps2cl.png)

该项目以可视化的方式展示了全球的天气情况，提供了风、温度、相对湿度等多种天气数据，以及风、洋流和波浪的动画效果。

**用户评价**：wow！这个项目很酷啊。

![image.png](https://s2.loli.net/2024/02/02/3Xe2NEtwLSa7nlq.png)

GitHub 地址→→→→https://github.com/cambecc/earth



## 最后
能看到这里的都是真爱粉了，再次感谢大家过去一年的陪伴，我们一起见证了 HelloGitHub 的成长。

2023 年，我和 HelloGitHub 遇到了很多挑战，还好都挺过来了，没有断更、没有滥竽充数、没有忘记初心。

我知道这很难、也很慢，但有你们我想试试。

新的一年，我将用实际行动帮助更多的人了解开源、走近开源、爱上开源！HelloGitHub 愿做大家开源之旅上的朋友，让我们一路相伴。

