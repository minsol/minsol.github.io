---
layout:     post
title:      "iOS备忘录1-委托和Block的回调"
subtitle:   "在本文中只记录在实际应用中委托和Block回调的简单使用，做以备忘。"
date:       2017-01-10
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---



### 前言
在本文中只记录在实际应用中委托和Block回调的简单使用，做以备忘。

### 委托（Delegate）



##### 首先明确两点概念：

> **委托（Delegate）：委托是Objective-C中常用的一种设计模式（也叫代理设计模式），由代理对象、委托者、协议三部分组成。**

> **协议 (Protocol)：就是遵守了某个协议后就要按照这个协议来办事，通常@protocol与@end在 声明协议时是成对出现。**

> _注1：其实**Protocol**和**Delegate**完全不是一回事，只是因为代理的实现通常与协议是密不可分的，所以我们经常在同一个头文件里看到这两个单词才将这两个放在一起说。_



##### 委托的使用流程

> 委托方：1.指定原则（协议）2.声明一个代理人的属性（delegate）3.在适当的时机，通知代理人执行代理方法

> 代理方： 1.遵守协议 2.设置自己为委托方的代理人 3.实现 协议方法

###### ·我是委托方

```
委托方的 .h 文件中实现上诉1、2两点：指定原则（协议）、声明一个代理人的属性（delegate）
#import <UIKit/UIKit.h>
@protocol SJSAddUserDelegate <NSObject>
//代理方必须实现的方法
@required
//代理方可选实现的方法
@optional
- (void)updataUser;//可以带参
@end

@interface SJSAddUserViewController : UIViewController
//关键属性委托代理人，为了避免造成循环引用，代理一般需使用弱引用
@property (nonatomic, weak) id<SJSAddUserDelegate> delegate;
@end

委托方的 .m 文件中实现上诉第三点：在适当的时机，通知代理人执行代理方法
if ([self.delegate respondsToSelector:@selector(updataUser)]){
	[self.delegate updataUser];//在适当的时候调用代理方法
}
```

###### ·我是代理方

```
代理方的.m文件中实现上诉第一点：遵守协议
@interface SJSUserController ()<SJSAddUserDelegate>
第二点：设置自己为委托方的代理人
addVc.delegate = self;//在创建委托方的时候指定直接为代理人
第三点：实现 协议方法
-(void)updataUser{
    //做代理人应该做的事
}
```




### Block
Block的详细介绍请参照：[iOS-Block的详解](http://www.jianshu.com/p/007632bbb01d)具体进行了解。

##### 场景一：回调

```
委托方.h
//声明一个Block类型的别名
typedef void(^AddCityFinish_Block)(NSString*,NSString*);

//声明block属性
@property(nonatomic,copy) AddCityFinish_Block block;
//用于直接传Block：方便愉快的撸代码
- (void)returnText:(AddCityFinish_Block)block;

委托方.m
/**
 *  对Block进行赋值  ，在代理方就直接调用方法就可以了
 *  这个就是相当于一个setter方法，对自己的block属性进行赋值
 *  @param block
 */
- (void)returnText:(AddCityFinish_Block)block{
    self.block = _block;
}

//适当的时机 执行block所代表的匿名函数：并将数据作为参数传回去
self.block(self.cityNameField.text, self.populationField.text);
```

```
代理方.m文件中
//在创建写一个界面的时候，设置下个界面的block属性为这里的匿名函数

/** Block方法一 ： 写Block的时候没有提示 */
addCity.block = ^(NSString *name, NSString *population) {
//做代理人应该做的事
};

/** Block方法二 ： 写Block的时候有提示 （方便快捷）*/
[addCity returnText:^(NSString * name , NSString *population ) {
//做代理人应该做的事
}];
```

##### 场景二：数据请求（二次封装AFNetworking）：未完待续...
