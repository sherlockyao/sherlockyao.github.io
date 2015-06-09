---
layout: post
title: "为 UIActivityViewController 添加自定义的 UIActivity 选项"
date: 2015-06-09 15:13
comments: true
categories: [iOS, UIActivityViewController, UIActivity, Share, Custom, 分享, 自定义, UI]
---

在现今开发应用的过程中，为其添加分享功能几乎成了一个不可或缺的需求。而 iOS 里的 UIActivityViewController 为我们提供了一种非常简单、快速的实现方案，因此往往成为初期快速迭代中的第一选择。但是，其自带的 Activity Type 毕竟有限，所以有时我们需要添加一些自定义的类型来满足需求。这里我就简单说明一下如何通过继承 UIActivity 来添加自定义选项。

首先，让我们再简单认识一下UIActivityViewController，它到底是用来做什么的呢？（UIActivityViewController自从 iOS6 开始便被加入到 API 中，要具体了解请看[此文](http://nshipster.com/uiactivityviewcontroller/)）总得来说，它主要做两件事：
	
- 它从你的应用接收各种对象数据，可以是 NSString, NSAttributedString, NSURL, UIImage 等等，它们被称为 Activity Items。

- 它管理着所有的 Activities，它把接收到的数据传递给每个 Activity，同时展示给用户；这些 Activities 包括系统自带的，用户自定义的，以及来自 Share and Action Extensions 的。
	
它展示的界面效果是这样的：

![File Inspector]({{ root_url }} /images/20150609/uiavc.png")

它把 Activities 分成了上下两部分，上面的都是属于 Share 类别的，而下面的则是 Action 类别。UIActivityViewController 的 UI 界面是系统生成的，不支持自定义，所以如果对界面有特殊要求的话，恐怕只能自己去实现相应的功能了（这里有一个[开源库](https://github.com/overshare/overshare-kit)可以参考），但是你也失去了其他应用实现的 Share Extensions 和 Action Extensions 的支持了。

言归正传，我们来看一下如何添加一个自定义的分享选项。其实过程非常简单，你只需要新建一个类，把它作为 UIActivity 的子类，然后重载以下这些方法：

- ``` + (UIActivityCategory)activityCategory ``` 这个方法就是告诉系统这个 Activity 是 Action 类型还是 Share 类型，默认是 Action，所以我们这里要返回 UIActivityCategoryShare。

- ``` - (NSString *)activityType ``` 用来区分不同 activity 的字符串，用你的 bundle id 做前缀就会避免冲突

- ``` - (NSString *)activityTitle ``` 显示在选项图标下的文字

- ``` - (UIImage *)activityImage ``` 图标素材，这里要注意的是，目前只有 iOS 8 才支持显示彩色的图标，在这之前，你所提供的素材其实是作为 mask 来使用的，显示的则是黑白效果

- ``` - (BOOL)canPerformWithActivityItems:(NSArray *)activityItems ``` 这里就是你通过 items 来判断当前 Activity 是否支持，如果不支持（返回No），则当前选项不会在界面中显示出来

- ``` - (void)prepareWithActivityItems:(NSArray *)activityItems ``` 为分享做准备，你必须在这里把这些 items 保存下来，然后做一些适当的准备工作

- ``` - (void)performActivity ``` 真正执行 share 动作的地方，这里要注意的是，不管分享成功与否，都要在结束后调用 ``` - (void)activityDidFinish:(BOOL)completed ``` 这个方法来通知系统分享结束了

实现好我们自定义的选项后，使用起来就非常简单了：

```objc
NSArray* activities = @[ [CustomActivity1 new], [CustomActivity2 new] ];
[[UIActivityViewController alloc] initWithActivityItems:activityItems applicationActivities:activities]
```

更多详细内容请点击这篇[参考文章](http://getnotebox.com/developer/uiactivityviewcontroller-ios-8/)继续阅读