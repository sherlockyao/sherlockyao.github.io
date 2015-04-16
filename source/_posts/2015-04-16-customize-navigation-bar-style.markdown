---
layout: post
title: "自定义Navigation Bar样式"
date: 2015-04-16 15:51
comments: true
categories: [ios, navigation, 导航条, NavigationBar, navigationItem, appearance]
---

在开发iOS应用时，自定义Navigation Bar样式是非常普遍的一个需求，所以这里特别整理了一些常见的情况，以供快速查阅。照旧先附上参考的原文[链接](http://www.appcoda.com/customize-navigation-status-bar-ios-7/)。

### > 更改导航栏背景色

这个应该是最常见的需求了，同时也只需一行代码就可以实现：

```objc

[[UINavigationBar appearance] setBarTintColor:[UIColor yellowColor]];

```

### > 更改导航栏标题的样式

有时我们需要修改标题成特殊的字体样式以达到设计上得效果，可以通过设置titleTextAttributes的属性来实现，以下是一些常用的key，同时附上实现代码：

  * UITextAttributeFont – Key to the font
  * UITextAttributeTextColor – Key to the text color
  * UITextAttributeTextShadowColor – Key to the text shadow color
  * UITextAttributeTextShadowOffset – Key to the offset used for the text shadow

```objc

NSShadow *shadow = [NSShadow new];
shadow.shadowColor = [UIColor grayColor];
shadow.shadowOffset = CGSizeMake(0, 1);
[[UINavigationBar appearance] setTitleTextAttributes: @{
  NSForegroundColorAttributeName : [UIColor whiteColor],
  NSShadowAttributeName : shadow,
  NSFontAttributeName : [UIFont fontWithName:@"HelveticaNeue" size:21.0]
}];

```

有些设计会要求使用logo或其他更复杂的内容代替文字显示在标题位置，这个时候我们可以通过设置navigationItem的titleView来实现这些需求，这里以简单地替换成图片为例子：

```objc

// 这里的viewController就是对应的要替换标题的那个view controller，而不是navigation controller本身
viewController.navigationItem.titleView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"title"]];

```

### > 修改导航栏上按钮的颜色

我们可以通过设置tintColor来实现，要注意的是，这个颜色会同时影响到所有按钮上的文字和图片：

```objc

[[UINavigationBar appearance] setTintColor:[UIColor whiteColor]];

```

### > 添加更多的导航栏按钮

这种情况下，我们可以通过设置leftBarButtonItems和rightBarButtonItems来实现，直接给出一段实例代码，大家一看就懂：

```objc

UIBarButtonItem *markItem = [UIBarButtonItem barItemWithImage:[UIImage imageNamed:@"mark"] selectedImage:nil size:CGSizeMake(40, 40) target:self action:@selector(markButtonClicked:)];
UIBarButtonItem *starItem = [UIBarButtonItem barItemWithImage:[UIImage imageNamed:@"star"] selectedImage:[UIImage imageNamed:@"star_selected"] size:CGSizeMake(30, 40) target:self action:@selector(starButtonClicked:)];
self.navigationItem.rightBarButtonItems = @[starItem, markItem];

```
