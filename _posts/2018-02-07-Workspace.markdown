---
layout:     post
title:      "iOS使用Workspace来管理自己的项目和Framework"
subtitle:   "本文记录在实际开发中使用Workspace来管理自己的项目和Framework。"
date:       2018-02-07
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---

### 前言
本文记录在实际开发中使用Workspace来管理自己的项目和Framework
</br>本文Demo[Demo地址](https://github.com/minsol/WorkSpace)
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/framework.png)

***


### 概念

>1. Framework是资源的集合，将静态库和其头文件包含到一个结构中:.a文件不能直接使用，至少要有.h文件配合，.framework文件可以直接使用。`.a + .h + sourceFile = .framework。`
>2. 静态库：链接时，静态库会被完整地复制到可执行文件中，被多次使用就有多份冗余拷贝
>3. 动态库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存
>4. 注意理解：无论是.a静态库还.framework静态库，我们需要的都是二进制文件+.h+其它资源文件的形式，不同的是，.a本身就是二进制文件，需要我们自己配上.h和其它文件才能使用，而.framework本身已经包含了.h和其它文件，可以直接使用。
>5. 图片资源的处理：两种静态库，一般都是把图片文件单独的放在一个.bundle文件中，一般.bundle的名字和.a或.framework的名字相同。.bundle文件很好弄，新建一个文件夹，把它改名为.bundle就可以了，右键，显示包内容可以向其中添加图片资源。
>6. category是我们实际开发项目中经常用到的，把category打成静态库是没有问题的，但是在用这个静态库的工程中，调用category中的方法时会有找不到该方法的运行时错误（selector not recognized），解决办法是：在使用静态库的工程中配置other linker flags的值为-ObjC。
>7. 如果一个静态库很复杂，需要暴露的.h比较多的话，就可以在静态库的内部创建一个.h文件（一般这个.h文件的名字和静态库的名字相同），然后把所有需要暴露出来的.h文件都集中放在这个.h文件中，而那些原本需要暴露的.h都不需要再暴露了，只需要把.h暴露出来就可以了。



### 一：准备工作
#### 1.创建一个工作空间。
File->New->Workspace,命名为workPlace,存放在文件夹WorkPlace中
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/workPlace.png)
#### 2.创建一个主APP工程
将创建的工程放入WorkPlace与之前创建的工作工程同级
***
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/cocoaTouch.png)
#### 3.创建一个Cocoa Touch Static Library的静态库
将创建的工程放入WorkPlace与之前创建的工作工程同级
#### 4.创建一个Cocoa Touch FrameWork的静态库
将创建的工程放入WorkPlace与之前创建的工作工程同级
#### 5.目前工程的目录结构
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/Contents.png)


### 二：将三个工程添加到Workspace
1. 点开workPlace.xcworkspace，然后添加对于的.xcodeproj
</br>![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/addFile.png)

### 三：将两个静态库添加到主工程
1. 设置两个静态库的Edit Scheme在Run的时候为Release
</br>![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/editScheme.png)
2. 在主工程里面添加Header Search Paths
</br>在主项目的Build Settings 里找到Header Search Paths，添加一项`$(SRCROOT)/../Framework`
</br>![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/HeaderSearch.png)
3. 静态库添加到主工程里
</br>在主项目的Build Phases 里找到Link Binary With Libraries添加对接的静态库
</br>![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/LinkBinary.png)
</br>
<font color=red>注意：swift里面的Framework需要在target -> General ->Embedded Binaries 中添加这个framework</font>（详情参见之前文章[swift-----打包自己的framework](https://minsol.github.io)）
</br>![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/workSpace/Embedded.png)
4. 为.a的静态库添加swift和OC的桥接文件
</br>（详情参见之前文章[Bridging-Header-----手动创建桥接文件](https://minsol.github.io)）
5. 在主工程中导入头文件Build
















