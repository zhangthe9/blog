---
layout: post
title: vmware mac os 使用
description: vmware mac os 使用
categories:
- vmware
- mac
tags:
- mac
---
###xcode
安装xcode & xcode tools，为homebrew的依赖
4.6.2
http://blog.csdn.net/cielpy/article/details/8557283

###homwbrew
http://mxcl.github.io/homebrew/
`ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"`


	su
	cd /usr/bin
	mv git git-old
	brew install git
	git --version

###替换shell为zsh
mac 自带了zsh 4.x版本
brew 的zsh是 5.x
####oh-my-zsh
curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh
或者替代的
####prezto
git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"

zsh

setopt EXTENDED_GLOB 
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N);
do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done 

参考
https://github.com/sorin-ionescu/prezto
http://blog.jr0cket.co.uk/2013/03/hey-prezto-zsh-for-command-line-heaven.html
http://www.yangzhiping.com/tech/mac-dev.html
Mac开发者2013年新机设置参考
###清理
mac 10.8
/System/Library/Frameworks/ScreenSaver.Framework /Versions/A/Resources/Default Collections/