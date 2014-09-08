---
layout: post
title: "如何更新Storyboard的多国语言strings文件"
date: 2014-09-08 18:47
comments: true
categories: [iOS, storyboard, localize]
---

目前XCode已经提供了很好的国际化集成，使你很容易就可以让应用支持国际化，就连Storyboard里面的文字也可以自动提取出来让工程师配置对应的多国文字。今天要讲的是当一个Storyboard文件已经本地化了以后，又有了新的需求而进行了改动，这时该如何去更新多国语言文件。

首先，如果你还不清楚如何国际化storyboard文件，那请点击[这里](http://www.raywenderlich.com/64401/internationalization-tutorial-for-ios-2014)。

当一个storyboard文件被localize了以后，在XCode右边的文件查看窗口就可以看到对应的语言列表，它们前面有一个checkbox，如果勾选的话就表示那种语言有对应的strings文件。因为XCode只支持一次性生成所有的storyboard的localize文件，所以当每次更新storyboard后想更新语言文件就变得非常麻烦。

![File Inspector]({{ root_url }} /images/20140908/storyboard.png")

我这里提供一个实践中学到的比较好的办法，就是首先你先备份一份你想要更新的语言文件，然后在右边的文件窗口把对应的那个勾选项取消掉，接着再选上，这个时候XCode就会重新生成一份新的文件。我们要做的就是把老文件里已经写好的对应关系复制到新文件上，再把新的元素的文字补上就可以了。
