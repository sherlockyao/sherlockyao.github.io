---
layout: post
title: "UIScrollView在Autolayout下的实践经验"
date: 2014-09-01 13:14
comments: true
categories: iOS 
---

平常在我们使用UIScrollView的场景中，经常会有需求给它增加多个子元素，同时又需要有对应的布局。Autolayout对布局灵活性支持的提升是显而易见的，所以我们当然希望在Scroll View里也能使用约束(Constraint)来进行布局，不过在实践中Autolayout的引入往往会带来一些意外的问题，今天我就介绍一种比较好的实现方法来规避一些不必要的麻烦。

首先我们在Scroll View下面加一层额外的UIView(以下称为View A)，同时确保这个UIView是ScrollView的唯一子View，所以我们需要的其他页面元素都应该放在A下面，这个A的作用就是更方便我们来控制ScrollView的内容长度。

接下来我们就要添加A和ScrollView之间的约束了:

- 我们需要给A添加Width和Height的约束，用一个写死的值(或者用placeholder约束，然后在代码里动态添加约束，这样可以做到动态控制内容的大小)，这样做的目的就是确定A的内容大小。

- 同时我们也需要添加A到Scroll View的top,right,bottom和left的约束，这些约束的目的是为了能让Scroll View计算出它的content size，这些约束不会改变A的大小，其实Scroll View就是通过A的大小和这些周边的距离值来确定其内容大小的。

做完上面两步后，我们就可以像平常一样设计我们的布局，添加逻辑代码了。当然这不能规避所有问题，但是通过这个方案可以从概念上分离内容大小和显示区域大小，从而简化我们的工作。

想深入了解的话可以点这里[参考文章](http://spin.atomicobject.com/2014/03/05/uiscrollview-autolayout-ios/)
