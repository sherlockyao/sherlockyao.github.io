---
layout: post
title: "Core Data 中‘一对一’到‘一对多’关系的轻量级迁移"
date: 2015-06-08 14:14
comments: true
categories: [iOS, Core Data, Tip, Lightweight Migration, Migration, To-One, To-Many, 迁移，一对多，一对一]
---

这次分享一个关于 Core Data 数据迁移的小经验。工作中遇到需求，要将原有数据结构中的一个 To-One 关系改成 To-Many（比方说把一个 Book 对应一个 Author 的关系改成对应多个 Authors）。根据 Core Data 的文档：

> Changing a relationship from a to-one to a to-many, or a non-ordered to-many to ordered (and visa-versa)

轻量级迁移（Lightweight Migration）是完全可以做到的。于是立刻添加了新的 Model Version，然后修改了对应的 relationship，同时也把对应 Managed Object 中的 property 手动改成 NSSet，以及添加了 add 和 remove 的方法（当然也可以用 Xcode 去自动生成新的类文件）。在调整了对应的逻辑代码后运行应用，发现迁移是成功的，数据没有丢失，但是原先那个一对一的关系数据没有迁移到新的一对多关系里（也就是所有 Book 和 Author 的对应关系消失了）。

看这个现象，数据是没有丢失的，所以应该是原来的对应关系没有顺利转化到新的这个多对多关系表中。回想整个迁移过程，我意识到一点，就是我把 Book 中原来的 author 属性改名成了 authors，这在逻辑上讲是理所当然的，但程序可没那么智能，所以应该是这里出了问题。

解决方法：在Xcode中选择 authors 这个关系，然后在右边的 Inspector 中找到 "Renaming ID" 这个选项，在里面填上 "author"，这样程序才能知道这个属性是从原来的 author 属性改名而来，数据当然也能成功迁移了。果然再次把应用回退到老版本后安装新版本，对应关系顺利地展示了出来。


