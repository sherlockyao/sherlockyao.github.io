---
layout: post
title: "Hello Blogging"
date: 2014-05-23 15:30
comments: true
categories: 
---

很久以来都没有认真写过技术博客，今天在学习Clojure的时候又用起了Vim，却突然发现好多以前熟知的操作因为疏于练习而忘记了，意识到平时记录些Notes的重要性，这才翻出了早先就配置好但一直荒废的Git Pages，写下这些文字。作为一个懒惰的人，我可能写完这篇便就半途而废了，但仍抱有对持久写作的一丝希望，作为开篇，就简单说明一下如何建立自己的github.io blog。

搭建的文档官方已经很完善了，这里就只给出链接，另外我用的是Octopress，所以文档是它家的，至于Markdown的话找了一篇简洁明了的：

- [Github Pages](https://pages.github.com/)
- [Octopress](http://octopress.org/docs/)
- [Markdown](http://jianshu.io/p/q81RER)

这里再赘述一下创建一篇博文的简单流程，以作备忘用：

1. 用下面的命令生成一篇新文章  
  `rake new_post["Create A Post With Given Title"]`

2. 书写完毕后，可以用以下命令预览  
  `rake preview` or `rake watch`

3. 确定要发布的时候就使用下面命令，它会将页面生成到source目录下同时push到master分支  
  `rake generate` and `rake deploy`

4. 最后别忘了把source分支的修改提交并push到git上  
  `git commit -m'msg'` and `git push origin source`
