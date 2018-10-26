---
layout:     post
title:      "iOS_Swift中桥接头文件建立的两种方法"
subtitle:   "本文记录在实际开发中桥接头文件建立的两种方法。"
date:       2018-02-07
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - iOS开发记录
---

### 前言
本文记录在实际开发中桥接头文件建立的两种方法
</br>备注：swift环境
***

### 方式一：在swift环境下的工程里面还没有加入任何Objective-C的文件时
#### 直接创建一个Objective-C的文件
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/Bridging-Header/Bridging-Header1.png)

### 方式二：自己手动创建
#### 1.创建一个Header File
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/Bridging-Header/Bridging-Header2.png)
#### 2.路径选择`TARGETS———BuildSetting———搜索header------$(PROJECT_DIR)/项目名字/Bridging-Header.h`
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/Bridging-Header/Bridging-Header3.png)
#### 3.编译通过



### 顺便记录一下Objective-C下的PCH的创建
#### 先说一下pch的作用：
预编译头文件(一般扩展名为.PCH),是把一个工程中较稳定的代码预先编译好放在一个文件(.PCH)里,它们在整个工程中是较为稳定的,即在工程开发过程中不会经常被修改的代码。在我的理解里是把一些宏定义（kScreenW等），一些要在多个类中使用的头文件在此文件中书写。简单来说，在.pch文件中定义的宏定义会作用到项目中的所有文件。
>1. 存放一些全局的宏(整个项目中都用得上的宏)；
2. 用来包含一些全部的头文件(整个项目中都用得上的头文件)；
3. 能自动打开或者关闭日志输出功能。

#### 实际操作：
##### 1.创建一个Header File
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/Bridging-Header/Bridging-Header4.png)
##### 2.路径选择`TARGETS———BuildSetting———搜索pch------$(PROJECT_DIR)/项目名字/Bridging-Header.h`
![image](https://raw.githubusercontent.com/minsol/MarkdownPhotos/master/Images/Bridging-Header/Bridging-Header5.png)
#### 3.编译通过



