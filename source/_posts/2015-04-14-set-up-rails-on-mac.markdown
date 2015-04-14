---
layout: post
title: "Mac上安装Ruby On Rails"
date: 2015-04-14 19:48
comments: true
categories: [rails, ruby, homebrew, rbenv, bundle, ror]
---

Mac加了SSD硬盘，重装了系统，所以需要重新安装Rails，找到一篇文章一步步照着来做，顺带记录下步骤分享给需要的人。本人Mac系统版本10.10.3。([原文链接](https://gorails.com/setup/osx/10.10-yosemite))

- 安装Homebrew

Homebrew能帮我们更便捷地从源代码编译安装其他软件，在Terminal里运行以下命令:

```bash
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- 安装Ruby

原先是用rvm来安装ruby的，这里用的是rbenv，据同事说这个比rvm更好用，总之照着下面的命令来吧：

```bash
	brew install rbenv ruby-build

	# Add rbenv to bash so that it loads every time you open a terminal
	echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
	source ~/.bash_profile

	# Install Ruby
	rbenv install 2.2.1
	rbenv global 2.2.1
	ruby -v
```

- 安装Rails

Terminal里运行以下命令：

```bash
	gem install rails -v 4.2.0

	#在这个过程中本人遇到安装失败，提示运行以下命令
	xcode-select --install
```
装完rails，为了能让rails成为可执行命令，还需要运行以下命令：

```bash
rbenv rehash
```

OK，大功告成，最后分享一条Tip：
平时国内网络下安装gem包总是会连接不上，淘宝提供了mirror，所以可以用下面的命令把地址转到淘宝的链接：

```bash
bundle config mirror.https://rubygems.org http://ruby.taobao.org
```