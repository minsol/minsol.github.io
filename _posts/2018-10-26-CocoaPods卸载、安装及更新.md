---
layout:     post
title:      "CocoaPods卸载、安装及更新"
subtitle:   ""
date:       2018-07-14
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - iOS开发记录
---
### CocoaPods介绍
#### CocoaPods 是开发 OS X 和 iOS 应用程序的一个第三方库的依赖管理工具，而其本身是利用 ruby 的依赖管理 gem 进行构建的。

### CocoaPods卸载
  1. 打开终端,输入命令```which pod```然后回车.我们就看到一个地址,这个地址就是我们安装pod 的地址
  2. 卸载 cocoaPods： ```sudo rm -rf /usr/local/bin/pod```
  3. 查看gems中本地Cocoapods程序包：```gem list ```.
  4. 卸载gems中本地Cocoapods程序包```sudo gem uninstall cocoapods -v 0.39.0```（注：后面的版本号要和上面列表中的版本号对应）
  5. 卸载 rvm：```rvm implode```(RVM:Ruby Version Manager,Ruby版本管理器包括Ruby的版本管理和Gem库管理(gemset))
  6. 卸载 HomeBrew:```ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"```
  7. 验证  直接输入```pod```:   ```command not found```

### CocoaPods安装
  - Mac电脑自带Ruby环境
  1. 查看原来的Ruby源```gem sources -l```
  2. 删除原来的源```gem sources --remove https://rubygems.org/```
  3. 将Ruby的软件源替换成国内的(注意最近由.org换成了.com) ```gem sources -a https://gems.ruby-china.com/```
  4. 安装RVM```curl -L get.rvm.io | bash -s stable ```
  5. 等待一段时间后就可以成功安装好RVM后输入两个命令 ```source ~/.bashrc``` ----```source ~/.bash_profile```
  6. 查看rvm  ```rvm -v ```
  7. 直接安装```sudo gem install cocoapods```
  8. 安装可能遇到的错误<br>
      > 1. While executing gem ... (Errno::EPERM) Operation not permitted - /usr/bin/fuzzy_match 错误<br>
        解决方案 ：```执行sudo gem install -n /usr/local/bin cocoapods```语句。然后提示```gems installed```即可。
      > 2. Error installing pods:active support requires Ruby version >= 2.2.2<br>
        解决方案：使用rvm升级ruby<br>
          1. ```rvm list known```<br>
          2. ```rvm install 2.2.2```如果直接成功请绕过homebrew的卸载安装<br>
          3. 如果出现```Requirements installation failed with status: 1.```,先卸载homebrew再执行```rvm install 2.2.2```
          4. 继续```sudo gem install cocoapods```
      > 3. gem安装时出现 ```undefined method `size' for nil:NilClass (NoMethodError)``` <br>
          解决办法:将其下的cache目录删除，再次执行gem安装的时候就不会出错了<br>
                    # gem env  得到gem的PATH路径，比如<br>
                      - GEM PATHS:<br>
                              - /usr/local/ruby/lib/ruby/gems/2.1.0<br>
                              - /home/vagrant/.gem/ruby/2.1.0<br>
      > 4. 等待很长时间```[!] Unable to find a specification for `FCUUID (~> 1.3.0)```<br>
            1. ```pod repo remove master```<br>
            2. ```pod setup```
  9. 验证```pod search AFNetworking```



### CocoaPods使用
  1. 新建工程在工程的根目录创建Podfile文件（other－>empty名字叫Podfile）
  2. 编辑Podfile文件
        ```
        platform :ios, '8.0'
      #use_frameworks!个别需要用到它，比如reactiveCocoa

      =begin
      多行注释
      =end
      target 'HuiMaBaoAPP' do
        #pod 'AFNetworking', '~> 2.6'
      end
        ```
  3. 安装：```pod install --verbose --no-repo-update```
  4. 更新：```pod update --verbose --no-repo-update```

  - 每次执行pod install和pod update的时候，cocoapods都会默认更新一次spec仓库。这是一个比较耗时的操作。在确认spec版本库不需要更新时，给这两个命令加一个参数跳过spec版本库更新,可以明显提高这两个命令的执行速度。
