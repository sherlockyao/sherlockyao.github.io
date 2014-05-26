---
layout: post
title: "Making a CocoaPod"
date: 2014-05-26 13:59
comments: true
categories: [CocoaPod, iOS]
---

最近想把自己正在做的iOS项目里的一个手势控制代码单独提出来以library的形式分享出去，本打算只是简单地在Github上建个public库然后把代码文件放上去，但作为一个“合格”的程序员，要时刻保持探索精神，所以打算做成时下流行的Cocopod公布出去。虽然要分享的代码作为一个pod还远不够格，但即使是菜鸟也应该为开源世界做点贡献，以下是随pod创建过程记录的一些notes，供大家参考。

首先给出官方指导链接[Making a CocoaPod](http://guides.cocoapods.org/making/making-a-cocoapod.html)，我基本就是参考这篇文章来操作的。

- 第一步就是新建目录结构`pod lib create SYDemoPod`[^pod-version]（我是采用完全从头开始创建的方法，所以对于如何把已经放在github上的文件变为pod的过程，这篇文章不会涵盖），执行完后pod就会自动在当前文件夹创建一个SYDemoPod的文件夹，里面会生成一些默认的文件和目录。


- 下一步就是把我需要发布的文件放到Classes目录下，比方说是文件SYPod.h和SYPod.m，因为是从我的一个iOS项目里提出来的，暂时就只让它作为iOS的库好了，所以就把他们拷贝到Classes的ios目录下，同时修改.podspec文件，把里面的信息改成自己需要的（如果不知道怎么写，可以找个其他的pod照着写就行），这里有一个tip，在source file的配置项里，加上`'Classes', 'Classes/**/*.{h,m}'`，这样就可以把Classes下面所有的文件加进去了，不然是会加根目录下的文件。


- 接下来修改Example目里下的Podfile，配置pod依赖：

```bash
  platform :ios, '6.0'
   
  pod "SYDemoPod", :path => "../"
```

  现在我们就可以创建一个Example项目到那个文件夹里面，作为给其他用户演示如何使用我的这个库的示例项目。用XCode新建完项目，然后命令行`pod install`[^pod-version2]安装依赖，接着打开.workspace文件后你就可以coding你的示例代码了。


- 把该写的代码都完成后，就万事俱备，只欠发布了。我们先来验证一下podspec文件，在Pod目录下运行`pod lib lint`命令，当验证没问题后，我们就可以把所有文件都提交并push到自己的github上去，接下来我们要实际测试一下这个pod是否有效。

    在本地新建一个项目，同时加一个Podfile，加上如下配置：
  
```bash
  # 这里的url就是对应你自己的pod的github链接
  pod 'SYDemoPod', :git => 'https://example.com/URL/to/repo/SYDemoPod.git'
```

  配置好后运行`pod install`或`pod update`，然后看看是否能正确添加依赖，如果没有任何问题，那就可以发布啦，依次使用以下命令即可：

```bash
  rake release
  
  pod trunk push SYDemoPod.podspec
```

另：如果执行`pod trunk push`命令时提示你要创建一个session，那就先`pod trunk register`先注册一下就可以了

[^pod-version]: 我用的pod版本是0.29.0
[^pod-version2]: 笔者此时运行pod命令时，发现pod不是最新版本，所以升级到了0.32.1
