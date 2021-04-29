---
title: 在GNU/linux上安装并配置i3wm
date: 2016-11-13 16:36:15
categories: tech
tags: [linux,i3wm]
---

## 简介
随着linux发展，现在已经有越来越多的GNU/linux使用者。虽然这值得庆贺(Congratulations!!!^ _ ^)，但是很多用户都将windows的使用习惯带入GNU/linux类系统的使用中，这是一个不太好的习惯。   
    
很多追求效率(爱装 X)的GNU/linux使用者需要尽可能的提高自己的工作效率(让显示界面更好看)，于是诞生了一系列平铺窗口管理器。关于窗口管理器，这里有一个详细的介绍：[fq用户：Window_manager - wikipedia](https://en.wikipedia.org/wiki/Window_manager) 以及 [未fq用户：窗口管理器 - 百度百科](http://baike.baidu.com/view/2092693.htm)。<!-- more -->   
    
## 安装
直接从ubuntu 16.04的官方库中安装：   
```
sudo apt install i3
```
   
## 配置
其中[i3wm](http://i3wm.org/)(以下简称i3)是其中一个用户比较多的window manager，关于i3的介绍官网很详细，并且每一个配置也十分详细。一般来说，第一次进入i3桌面，需要让你设置$Mod键，根据个人习惯选择super(windows图标，mac键盘不清楚- _ -!)或者alt键；并且会生成默认的配置文件。你可以使用如下命令创建一个自己的配置文件：   
```
sudo cp /etc/i3/config ~/.config/i3/config  #拷贝默认配置文件至自己的i3配置目录中
sudo chmod 666 ~/.config/i3/config  #使配置文件可编辑
```
然后可以编辑配置文件，主要根据[官网文档](http://i3wm.org/docs/userguide.html)或者github上的很多配置自己按需查看。
### 配置锁屏shell
在i3中，锁屏默认是i3lock命令，然而默认的命令界面并不是十分好看，我介绍一种可以模糊当前窗口的锁屏效果，该方式需要安装scrot软件`sudo apt install scrot`。在~/.config/i3/config中加入下面一行代码：
```
bindsym $mod+l exec --no-startup-id blurlock    #其中的键位可以自己设定，blurlock是一个命令
```
现在我们创建一个blurlock命令：在包含在path变量中的目录中创建一个blurlock文件，并修改权限可执行：
```
sudo touch /usr/bin/blurlock  #其中路径可自定义
sudo chmod 777 /usr/bin/blurlock    #权限不一定非得是777，但是要可执行
```
下面是blurlock文件的内容：
```
scrot /tmp/screenshot.png
convert /tmp/screenshot.png -blur 0x20 /tmp/screenshotblur.png
rm /tmp/screenshot.png
i3lock -i /tmp/screenshotblur.png
```
### 其他配置
当然，你可以使用feh配置背景图，nm-applet进行网络设置，volumeicon进行声音设置，xfce4-power-manager进行电源管理，使用compton设置背景透明度。   
以上的配置基本上都可以从github或者其他网站找到，推荐[j4tools](https://www.j4tools.org/)，i3的大多数配置都有效果不错的源码。   
最后，上一张我配置好的图，我的配置文件地址：[https://github.com/jadegong/dotfiles](https://github.com/jadegong/dotfiles)   
   
![](https://raw.githubusercontent.com/jadegong/dotfiles/master/screenFetch-2020-12-11_13-57-07.png)

