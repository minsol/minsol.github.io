---
layout:     post
title:      "CocoaPods搭建自己的私有库"
subtitle:   ""
date:       2018-10-26
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---

### CocoaPods安装
1. 查看原来的Ruby源
    > ```gem sources -l```
2. 删除原来的源
   >```gem sources --remove https://rubygems.org/```
3.  将Ruby的软件源替换成国内的(注意最近由.org换成了.com)
    > ```gem sources -a https://gems.ruby-china.com/```
4. 直接安装
    >```sudo gem install cocoapods```
5. 安装可能遇到的错误在以后总结

### 准备工作
#### 创建一个专门用来放podspec的私有库

![](https://github.com/minsol/MarkdownPhotos/blob/master/Images/CocoaPods/CocoaPodsSpec.png?raw=true)

1. 将这个远程的私有版本仓库添加到本地
    >pod repo add MinsolSpec xxxx(你刚创建的仓库地址 )<br>
     
    这时候可以在```~/.cocoapods/repos```里面看到多了一个MinsolSpec的本地库文件夹

![](https://github.com/minsol/MarkdownPhotos/blob/master/Images/CocoaPods/CocoaPodsSpec1.png?raw=true)
#### 创建一个私有库用来放源文件

![](https://github.com/minsol/MarkdownPhotos/blob/master/Images/CocoaPods/CocoaPodsSpec2.png?raw=true)

1. 这个仓库用于放置所有私有库的源文件(现在直接搭建在github以备日后好查看)

### 配置私有库

1. 创建.podspec文件
    >```pod spec create TestLib```
2. 编辑.podspec文件
```
  Pod::Spec.new do |s|
      s.name         = "TestLib"# 项目名称
      s.version      = "0.0.2"# 版本号 与 你仓库的 标签号 对应
      s.summary      = "项目简介 项目简介 项目简介 项目简介 项目简介 项目简介 项目简介."# 项目简介
      s.homepage     = "https://github.com/minsol/MinsolSpec.git"# 仓库的主页
      s.license      = { :type => 'MIT', :file => 'LICENSE'  }# 开源证书
      s.author             = { "wanjian" => "wanjian@a8sport.com" } #作者信息
      s.source       = { :git => "https://github.com/minsol/MinsolPrivate.git", :tag => "#{s.version}" }#你的仓库地址，不能用SSH地址
      s.source_files  =  'TestLib/Classes/Person.h'# 你代码的位置， Classes/*.{h,m} 表示 Classes 文件夹下所有的.h和.m文件
      s.public_header_files = 'TestLib/Classes/Person.h'# 头文件的位置
      s.ios.deployment_target = '8.0'  #平台及支持的最低版本

      # 下面的根据自己的需求进行配置---------------------------------------------------
      #s.requires_arc = true # 是否启用ARC

      #s.frameworks = "Foundation","UIKit"#支持系统的框架
      #s.library = 'z'#项目依赖的库文件(这个是系统的库文件),不需要后缀名,比如sqlite,libz等.以lib开头的需要省略掉lib这三个字母.例如:libz需要简写为z否则报错.

      #s.ios.vendored_library = 'libGDTMobSDK.a'#静态库
      #s.ios.vendored_frameworks = 'Frameworks/MyFramework.framework'#动态库

      # s.dependency "JSONKit", "~> 1.4"# 依赖库
      # s.dependency "Masonry" #多个依赖
      # s.dependency "OpenUDID"

      # 子库  如果有需要-------------------------------------------------------------
      # 表示建立一个名字为 xxx 的子文件夹，可以将文件分类
      s.subspec 'GDT_iOS_SDK_4.8.0' do |ss|
          ss.ios.vendored_library = 'GDT_iOS_SDK_4.8.0/libGDTMobSDK.a'
          ss.source_files = 'GDT_iOS_SDK_4.8.0/*.h'
          ss.frameworks = "AdSupport","CoreLocation","QuartzCore","SystemConfiguration","CoreTelephony","Security","StoreKit","AVFoundation"
          ss.weak_frameworks = "WebKit"
          ss.libraries = "z","xml2"
      end
      # 注意：不同子文件夹下 source_files 中的文件是单独编译的，如果文件中引入了别的子文件夹下的代码是编译不通过的。
  end
```

   > 文件匹配
   > - *匹配所有文件
   > - c*匹配以名字C开头的文件
   > - *c匹配以名字c结尾的文件
   > - *c*匹配所有名字包含c的文件
   > - **文件夹以及递归子文件夹
   > - ?任意一个字符(注意是一个字符)
   > - [set] 匹配多个字符,支持取反
   > - {p,q} 匹配名字包括p 或者 q的文件

3. 验证.podspec的有效性

    不需要联网
    >```pod lib lint TestLib.podspec --allow-warnings --use-libraries```
   
    会联网检查sepc repo,并且关联tag
    >```pod spec lint TestLib.podspec --allow-warnings--use-libraries```

    出现TestLib passed validation.表示验证通过（可以先本地测试）

4. 将准备好的的源文件推送到git上面的私有库中，并打上对应于podspec中version的标签。
    - 注意此时修改 ```source_files``` 和 ```vendored_libraries``` 的绝对路径，仓库很本地的路径不一样。
5. 将.podspec推送到本地.cocoapods/repos/MinsolSpec及之前关联的远程MinsolSpec仓库
    >```pod repo push MinsolSpec TestLib.podspec --allow-warnings --use-libraries```

    MinsolSpec:本地私有库文件夹名字。<br>
    此时会发现本地MinsolSpec文件夹下多了TestLib文件夹，并且已经同步到自己的远程仓库。

![](https://github.com/minsol/MarkdownPhotos/blob/master/Images/CocoaPods/CocoaPodsSpec4.png?raw=true)

![](https://github.com/minsol/MarkdownPhotos/blob/master/Images/CocoaPods/CocoaPodsSpec3.png?raw=true)

6.  ```每次更新完源代码后，更改podspec的version。然后继续验证步骤4和步骤5.(注意文件路径)```
### 使用
1. 新建工程在工程的根目录创建Podfile文件（other－>empty名字叫Podfile）
```
source 'https://github.com/CocoaPods/Specs.git'  # 官方库
source 'https://github.com/minsol/MinsolSpec.git'   # 自己私有库
#注意是专门用来放podspec的私有库，不是放源文件的库。

target 'demo' do

  # Pods for demo
    pod 'TestLib'
    pod 'YDTAppPrivate'
    pod 'YDTAdvertising', :path => "/Users/minsol/Desktop/YDTAdvertising"#本地验证
end
```
2. 安装
    >```pod install --verbose --no-repo-update```
    
3. 更新
    >```pod update --verbose --no-repo-update```

### 备注
#### .podspec文件参数解释
[.podspec文件参数解释](https://guides.cocoapods.org/syntax/podspec.html#vendored_libraries)
#### pod常用命令
```
//查看本地repos路径
~/.cocoapods/repos

//更新本地仓库
pod repo update
//查看索引库
pod repo 
//移除单个库
pod repo remove TestLib 

//关联本地与远程
pod repo add TestLib xxxx(你刚创建的仓库地址 )

//验证podspec文件
pod spec lint TestLib.podspec --allow-warnings --use-libraries
pod lib lint TestLib.podspec --allow-warnings --use-libraries

* pod lib lint 和 pod spec lint 有什么区别
pod lib lint 不需要联网
pod spec lint 会联网检查sepc repo,并且关联tag


//提交到库  (TestLib就是你们的私有库名 后面还有一个git的私有地址)
pod repo push TestLib TestLib.podspec --allow-warnings --use-libraries

//清空~/.cocoapods/repos下的目录（不建议）
pod repo remove master //清理目录(不建议)
pod setup //重置（不建议）


//获取已经缓存的列表
pod cache list
//清除指定的缓存
pod cache clean xxx(缓存的名字,上一步可获取)

git update --no-repo-update
git tag '0.2.0'
git push --tags
```
