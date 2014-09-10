---
layout: post
title: "VIPER实践（下）"
date: 2014-09-10 16:01
comments: true
categories: [iOS, Objective-C, VIPER, architecture]
---

上一篇文章我们设计好了View和Interactor，现在我们来做Presenter。

Presenter在这个用例里面的主要作用就是接受View提交过来的“提问”请求，然后向Interactor获取“答案”，最后把结果更新到View上，以下是实现部分的代码片段：

```objc

    @interface HumanComputerCommunicatePresenter : NSObject <AIRobotInteractorDelegate>

    @property (nonatomic, weak) id<ProgressViewInterface> progressView;
    @property (nonatomic, weak) id<MessageBoardViewInterface> messageBoard;
    @property (nonatomic, strong) AIRobotInteractor *aiRobotInteractor;

    - (void)wantSendMessage:(NSString *)message;

    @end


    @implementation HumanComputerCommunicatePresenter

    - (void)wantSendMessage:(NSString *)message {
      [self.progressView beginProgress];
      [self.aiRobotInteractor askQuestion:message]
    }

    - (void)aiRobotInteractor:(AIRobotInteractor *)aiRobotInteractor didResponseToQuestion:(NSString *)question response:(NSString *)response {
      [self.progressView endProgress];
      [self.messageBoard showMessage:response];
    }

    @end
```

可以发现，Presenter就是承担了一个中间者的角色，它才是传统意义上MVC机构中的C(Controller)。其中有一点要注意的是，对应ViewInterface的reference，我们采用了weak的定义来防止Strong reference cycle ，在我自己的设计当中，我偏向于如下的reference关系：

 - View --strong-> Presenter
 - Presenter --strong-> Interactor
 - Presenter --weak-> View

最后我们来看一下Routing，objc.io那片文章中的Routing做的很散，通过多个wireframe来实现，我个人的设计中偏向用一个大的wireframe结合Assembling Factory来实现Routing，废话不多说看代码比较直观，以下只是我个人目前的一种实现方案，仅供参考：

```objc
    @implementation AssemblingFactory

    + (UIViewController *)assembleChatroomView {
      ChatroomViewController *viewController = [[UIStoryboard genericStoryboard] instantiateViewControllerWithIdentifier:ChatroomViewIdentifier];
      HumanComputerCommunicatePresenter *presenter = [HumanComputerCommunicatePresenter new];
      AIRobotInteractor *interactor = [AIRobotInteractor new];
      
      viewController.presenter = presenter;
      presenter.progressView = viewController;
      presneter.messageBoard = viewController
      presenter.interactor = interactor;
      interactor.delegate = presenter;

      return viewController;
    }

    @end


    @implementation Wireframe

    + (void)moveToNextPageOfViewController:(UIViewController *)viewController messenger:(PageMessenger *)messenger {
      SEL selector = [self selectorOfClass:[viewController class] messengerName:[messenger name]];
      IMP imp = [[self class] methodForSelector:selector];
      void (*func)(id, SEL, UIViewController*, NSDictionary*) = (void *)imp;
      func([self class], selector, viewController, [messenger params]);
    }

    + (SEL)selectorOfClass:(Class)class messengerName:(NSString *)messengerName {
      static NSDictionary *selectorMap = nil;
      if (!selectorMap) {
        
        selectorMap = @{
                        @"SplashViewControllerDefault" : [NSValue valueWithPointer:@selector(moveToChatroomViewController:params:)]
                        // add more nav configuration here...
                        };
      }
      NSValue *value = [selectorMap valueForKey:[[class description] conj:messengerName]];
      return value ? [value pointerValue] : @selector(emptyMove:params:);
    }

    + (void)moveToChatroomViewController:(UIViewController *)viewController params:(NSDictionary *)params {
      UIViewController *viewController = [AssemblingFactory assembleChatroomView];
      [(BaseViewController *)viewController setParams:params]; // I defined a base view controller to allow pass params
      [viewController.navigationController pushViewController:viewController animated:YES];
    }

    @end
```

以上大致就是我个人目前实践VIPER的一些心得，肯定非常不成熟，有很多地方可以改进，只想抛砖引玉让大家广思集益共同改进我们代码的架构方案，从而最终帮助提高开发效率。
