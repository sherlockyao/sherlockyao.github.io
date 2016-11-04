---
layout: post
title: "更轻松地页面跳转，Wireframe框架（下）"
date: 2016-11-04 16:04:37 +0800
comments: true
categories: [iOS, Swift, 框架, Route, 跳转, navigation, wireframe]
---

上篇我解释了Wireframe框架的大体思路和跳转逻辑，这次我具体说明一下里面的builder, navigator和transition的概念。

### Builder

Builder顾名思义就是用来实例化目标View Controller的工厂。在Wireframe中实例化View Controller有两种方法：1）通过storyboard初始化 2）通过代码。下图是配置文件中相应的配置内容：

![File Inspector]({{ root_url }} /images/20161104/decodes.png")

可见，如果View Controller被画在了storyboard中，那么只要提供对应的storyboard文件名称，以及对应的view controller id，那么wireframe会自动通过storyboard来实例化。如果要使用代码实例化(包括从xib文件加载页面)，那么就需要通过配置builder来实现。首先需要在配置文件中配置对应的builder code，然后通过调用wireframe实例的register方法去注册对应的builder即可，示例代码如下：

```swift

register(builderName: "product") { (params) -> UIViewController in
    let productViewController = ProductViewController()
    productViewController.productName = params?[WireframeParam.name.rawValue] as? String
    return productViewController;
}

register(builderName: "seller") { (params) -> UIViewController in
    return SellerViewController(nibName: "SellerViewController", bundle: nil)
}

```

具体的实例化方式，是根据你具体的业务逻辑需求来的，方法所传递的params参数，是页面跳转时所传的额外参数(optional)，你可以根据具体需要来定制目标view controller的初始化过程。目前框架只提供了一个简单的UIAlertController的默认builder。


### Navigator

Navigator则是负责具体跳转的接口。它和Builder的使用方法类似，在配置文件中配置对应的name/code，然后通过register方法注册对应的navigator实现到wireframe中去。框架默认提供了7个常用的跳转方式，比如：present(Animated or Not Animated)，push(Animated or Not Animated)，如果app的跳转方式比较简单，那么默认的navigators就已经足够满足需求了。下面的代码演示了如果注册一个navigator，它的作用就是给目标view controller包上一个navigation controller再做跳转：

```swift

register(navigatorName: "navigation-wrap") { (sourceViewController, destinationViewController, completion) in
    let navigationController = UINavigationController(rootViewController: destinationViewController)
    sourceViewController.present(navigationController, animated: false, completion: completion)
}

```

### Transition

有些时候，我们需要给跳转过程加特定的动画效果，这就需要我们用到类似UIViewControllerAnimatedTransitioning这样的技术，wireframe也提供了配置的方式，每个Wireframe实例有一个optional的transition接口变量，你可以设置成自己实现的transition接口，每当wireframe完成目标view controller的初始化后，就会调用对应的接口方法来配置需要的transition，你可以通过判断soure和destination的类型来选择对应的动画效果。

目前transition这个配置的方式还不是很完美，会有很多类型判断的代码在里面，如果大家有好的建议欢迎给我留言：）

最后要说的是，对于大型的项目而言，一个项目中可以有多个wireframe存在，每个wireframe负责某个模块内部的调度，然后再由一个总的wireframe来负责模块间的调度，这样可以使结构更清晰，可维护性更强。这个是框架的[Github链接](https://github.com/sherlockyao/SYWireframe)，欢迎大家提供宝贵意见和建议。


