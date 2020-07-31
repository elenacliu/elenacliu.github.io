---
layout:     post
title:      Android-Studio安装记录
subtitle:   Android开发
date:       2020-07-31
author:     cliu
header-img:  img/post-bg-android.jpg
catalog: true
tags:
    - Android
---



# Android Studio安装记录

去年就搞过AS，但是因为完全零基础踩了不少坑：不知道安装SDK的前提是安装JRE/JDK；SDK和gradle下载不知道如何使用镜像站，网速很差，下载的东西根本不全，后来总是出现奇奇怪怪的问题完全不能使用。

今年因为课程需要再次下载，现记录一下步骤备用。

1. [官网](https://developer.android.google.cn/studio) 版本4.0.1

   选择.exe版本下载。

2. 安装完成后，点击运行AS，遇到的第一个错误

   ![error1.png](https://i.loli.net/2020/07/31/sADVZy8TJiMzwGN.png)

   

   原因：电脑没有SDK而且下载的android studio又是不带SDK的；

   按照网上教程的说法，先点击Cancel，进入主页面再在SDK manager工具里面进行下载。（但是实操发现不是这么个步骤）

3. 点击Cancel后就一路Next进入配置步骤。但是不知道为什么又从dl.google官网上下载了一大堆东西。

   当时的配置基本情况如下：

   从显示的信息来看应该是在这一步要主动给我安装SDK，前提是你的电脑环境已经配置好了JDK。

   ```
   Setup Type: Standard
   SDK Folder: C:\Users\*****\AppData\Local\Android\Sdk
   JDK Location: D:\AS\jre (Note: Gradle may be using JAVA_HOME when invoked from command line. More info...)
   Total Download Size: 498 MB
   SDK Components to Download: 
   
   Android Emulator
     
   235 MB
   
   Android SDK Build-Tools 30.0.1
     
   51.3 MB
   
   Android SDK Platform 30
     
   49.9 MB
   
   Android SDK Platform-Tools
     
   8.03 MB
   
   Android SDK Tools
     
   149 MB
   
   Intel x86 Emulator Accelerator (HAXM installer)
     
   2.63 MB
   
   SDK Patch Applier v4
     
   1.74 MB
   ```

4. 因为我的机器从官网上下载这些安装包的速度正常，所以下载很快就成功结束了。

<img src="https://i.loli.net/2020/07/31/aKVMPDjRu38JmSp.png" alt="daohang1.png" style="zoom:50%;" />

​	进入导航界面，先不着急start a new project。点击Configure->SDK Manager，确认自己的SDK安装完成。

<img src="https://i.loli.net/2020/07/31/cnY1ZNSawIErQzX.png" alt="SDK.png" style="zoom:67%;" />

​	SDK Platforms一栏列出了SDK的所有历史版本，其中最新版本已经安装完成。

5. 点击start a new project，选择默认的Empty Activity。配置信息如下：

   <img src="https://i.loli.net/2020/07/31/ErRmYOjiPDMeJBK.png" alt="config.png" style="zoom:67%;" />

6. 进入项目后，开始由AS自动下载Gradle。默认从官网下载，因为网速还可以，所以就直接默认从官网下载了。如果在国内网速不佳，参考[这里](https://blog.csdn.net/zxs0222/article/details/104387956)。由于是首次构建项目，还要在此过程中下载Gradle，即便是网速不错依旧花了将近20分钟。

   ![success.png](https://i.loli.net/2020/07/31/Ga51hBj7kgD9PH8.png)

7. 想要运行项目还需要设置AVD（虚拟设备）。但是听说虚拟设备很吃内存，因此希望找到性能最佳的模拟器，暂时选择了AS内置的模拟器，确实很吃内存。看到网伤有种方式是真机调试+apowermirror投屏到电脑上进行运行，听上去还挺不错的。
