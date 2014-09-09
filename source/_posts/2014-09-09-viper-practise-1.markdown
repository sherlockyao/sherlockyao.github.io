---
layout: post
title: "VIPER实践（上）"
date: 2014-09-09 14:35
comments: true
categories: [iOS, Objective-C, VIPER, architecture]
---

好的代码质量对于开发效率的提升可谓是指数级别的，好的代码架构则像是地基一样，是一切的前提。在iOS代码架构中讨论最多的对象就是ViewController了，传统的开发很容易产生其臃肿冗长的结果，我们很多人都为如何给它减肥伤透脑筋，无数前辈们探索出了各种不同的方案都值得我们学习参考。今天要说的VIPER架构就是较新的一次尝试，我是通过objc.io的文章第一次知道到它，具体链接[这里](http://www.objc.io/issue-13/viper.html)，虽然文章没有非常深入这个主题，给出的示例代码个人认为还存在些缺陷，但它整个理念正好吻合我之前一直在构思的方案，而且给了我一个非常系统的思路，于是我直接把它应用到了一个新的项目中去，在实践中去深入体会其优缺点，同时不断根据自己的理解改进，下面就通过一个假想的App作为示例来说明一下我自己的理解。

这个假想的App功能非常简单，类似一个简化版的Siri，用户在界面上输入一些文本后提交，应用就会返回一些回复的信息给你。首先在开始前我想说明一点，VIPER架构的核心思想就是把传统庞大的应用结构解构成View, interactor, Presenter, Entity, Routing五个部分，既然是做了解耦合，那各个部分直接应该做到相应的灵活和可重用，这点非常重要，我们在设计View，Presenter和Interactor时要时刻记住这一点。在objc.io原文章的示例代码中，这点做的就不够好，而在个人的实践中就更容易陷在这里，你会发现你给一个特定的View写了一个Presenter然后这个Presenter用到了一个特定的Interactor，这三个文件所做的就是应用中的一个特定的业务逻辑，所以也没办法被其他地方重用，其结果就是你只是简单的把原来属于一个ViewController的代码分散放到三个类中了，本质上却没有解耦合。请大家在看下面内容的时候时刻牢记这一点。同时示例我只提供了设计思路，并不提供实现的代码，因为示例的目的就是为了帮助理解VIPER的设计思路。

首先我们从View入手，界面上其实很简单：用户输入文本 -> 提交 -> 显示回应文本。我们想要的presenter对view的操作会有 1)显示/隐藏等待框 2)显示回应的文本两个主要的部分，接下来我们就要解耦合了。等待框和文本显示其实是不相关的两部分逻辑，所以我们的View Interface也应该要分开来，如下所示：

```objc
    // 用于显示和隐藏等待框
    @protocol ProgressViewInterface <NSObject>

    - (void)beginProgress;
    - (void)endProgress;

    @end

    // 用于显示回应文本
    @protocol MessageBoardViewInterface <NSObject>

    - (void)showMessage:(NSString *)message;

    @end
```

Prsenter是“VIP”结构中的中间件，起着承上启下的作用，所以我们放到下一篇文章里写，接下来我们来设计Interactor。Interactor在这里就只有一个功能，就是一个智能AI的角色。一般来说，interactor是一部分具体业务逻辑的实现者，虽然我们提倡对接口编程，但是这些实现一般一个应用就只会有一份，同时像Kiwi这种测试框架已经提供了良好的mock和stub支持，所以我们就不再对interactor设计接口，而是直接实现，其.h文件大致如下：

```objc
    @interface AIRobotInteractor : NSObject

    @property (nonatomic, weak) id<CAIRobotInteractorDelegate> delegate;

    - (void)askQuestion:(NSString *)question;

    @end


    @protocol AIRobotInteractorDelegate <NSObject>

    @optional
    - (void)aiRobotInteractor:(AIRobotInteractor *)aiRobotInteractor didResponseToQuestion:(NSString *)question response:(NSString *)response;

    @end
```

注意，这里我们使用了Delegate作为和presenter的交互方式，但不是一定要用，具体可以通过实际需求来设计交互形式。同时在类的命名上我故意使用了一些广泛的名字，目的就是为了突出解耦这个概念，要知道这些view interface和interactor可以在任何需要的地方使用而不是只局限在当前这个view controller。

好了，下一篇我将具体说Presenter和Routing。
