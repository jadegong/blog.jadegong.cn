---
title: 如何优化你的工作流
date: 2024-06-17 15:52:15
categories: tech
tags: [workflow,vim,emacs]
---

## 简介
本人最近在优化自己的工作流，有一些经验，写这篇文档分享出来，也算是自己的一个总结。本人最近几年使用 [vim](https://www.vim.org/) 进行几乎所有的文本处理工作，具体的原因下面会详细讲述。

## 基本原则
工作流的基本原则我认为有以下几点：<!-- more -->
### 1.符合人体工程学
这一点对于经常打字的编程者来说十分重要，我右手由于经常使用鼠标，已经感受到手腕神经被压迫了。左手键盘，右手鼠标的文本处理方式让我十分难受，使用 vscode 时经常需要右手使用鼠标进行光标定位，文件查找等操作，即使入了人体工程学鼠标，还是只能减轻右手手腕症状。

现在流行的编辑器和IDE大多快捷键很难用，大量使用 ctrl、shift 和 alt按键来操作，有的甚至需要同时按其中两个以上，使得左手十分紧张；另外右手也需要经常移动到方向键来进行选取等操作，离开主键盘区域又需要重新定位到 f 和 j 这两个按键，少量使用还好，大量编辑工作时，就很烦。

本人使用 vim 快捷键方式，并用到了几乎所有使用场景中，包括linux操作系统，vscode中也安装了 vim 插件，浏览器也使用 vim 快捷键。目前只有在浏览器中才会少量使用鼠标进行操作。

### 2.一套快捷键，全部场景
我最近几年感觉记忆力开始下滑，人也变懒了，不想记忆太多快捷键了，windows 有一套快捷键，vscode 有一套快捷键，浏览器有一套快捷键……我只想用一套快捷键在所有场景中工作。之前在 vscode 中安装了 jetbrains 的快捷键插件，但是这套快捷键方式不符合第一条原则，被弃用了。

### 3.快捷键方便记忆
很多快捷键设计根本没有任何含义，完全是乱设计，比如 shift + UpArrow 向上选择，根本没有什么方式来记忆，只能死记硬背。更多多个功能键的组合更是如此了，太难以记忆了。

### 4.完全支持自定义功能
这一点其实非常重要，很多工具非常优秀，但是提供了默认的不可更改的快捷键方式，导致我在使用中无法自定义相应的快捷键。很多工作流的优化都需要这个基本功能来实现，来打造属于适合自己的高效工作流。

## 工具选择
如今我已经不像学生时代那么喜欢折腾了，老喜欢搞些花哨，却没有什么实际意义的功能，我选择工具的优先原则就是能够带来更高的效率，符合以上基本原则。
### 文本编辑器
文本编辑器最重要的是不能有门户之见，很多不用 vim 编辑器和用 vim 编辑器的用户互相敌视，这是十分不理智的。工具的意义在于辅助我们的工作，对任何编辑器一定要十分了解才能知道如何优化，也才知道有什么缺陷。**我自己也使用 vscode 进行编程，只是我觉得 vscode 默认编辑方式很低效，我安装了 vscodevim 的插件，使得工具互相结合，取长补短**。
我最近几年使用 vim 作为主要的文本编辑器，进行文本处理工作。网上很多人说 vim 的快捷键很难学，其实是他们夸大了其难度，只是因为 vim 与我们从小接触的快捷键方式不同，所以觉得很难。官方教程 vimtutor 只需要半个小时就能完整过一遍，学完就会最基本的 vim 操作了。甚至会有人觉得文本编辑器居然要学？试想小时候使用 windows 也需要学习吧，学习难度并不低(至少我本人是这么觉得的)，很多人忘了 windows 也需要学习。

vim 的操作方式完全符合上述基本原则：
1. 双手基本都保持在主键盘区域，快捷键也比较少使用功能键组合，最典型的例子就是自定义 \<leader\> 按键(我将其定义为空格键)，不需要一直按住，只需要快速按两个键，就能实现基本功能(比如在vim启动目录中搜索，先后按下 \<space\>sr,即可使用rg进行搜索)；
2. 很多编辑器都支持插件，基本所有编辑器都有 vim 按键插件，可以随处使用，系统操作方式见下文；
3. vim 快捷键符合基本的语言特定，动词、介词、数词、名词，只要你学过一门语言，就很容易理解其快捷键方式；
4. vim 完全支持自定义功能，包括快捷键；
5. vim 占用内存很小，启动快，我配置的 vim 启动时间在100-200ms(使用 ``vim --startuptime ~/vimstart.log`` 命令启动并记录启动过程日志至 vimstart.log 文件中)。

我没有太大的兴趣也没有资格在这里做 vim 教程，只是提供一条新的道路。网上教程资源十分丰富，也有 [SpaceVim](https://spacevim.org/cn/) 这样开箱即用的 vim 配置集合，这也是开源社区的魅力所在，你可以自由选择，也可以自己对其做任意修改，以符合你的预期。

### 操作系统
操作系统我选择了 [archlinux](https://archlinux.org/)，其实选择任何你喜欢的 linux 操作系统(ubuntu,debian,openSUSE,fedora,mint etc.)就行，我使用了 i3wm 作为窗口管理器，原因有以下几点：
1. linux 系统可高度自定义，系统级别的软件也可以自己任意选择(包括窗口管理器，显示管理器等各种较为底层的系统功能)，提供了优化工作流的可能性；
2. i3wm 完全支持全键盘操作，平铺方式，应用间 focus 十分方便，我自定义的快捷键符合 vim 的操作模式，super + {h,j,k,l} 即可在不同窗口见跳转；
3. i3wm 占用内存很小，启动后系统整体占用内存在几百M；
4. linux 系统提供了大量高效的命令行工具，相互之间完全可以配合起来；
5. archlinux 的包管理器自动解决包之间的依赖关系，软件版本也比较新，可以自由选择软件的版本；
6. 本人并不依赖一些不适配 linux 的软件(例如微信、qq等)。

### 浏览器
浏览器目前也有 vim 按键插件，我使用的 firefox(国内账号没有同步问题，chrome有同步问题) 可以安装 vimium 插件，几乎可以脱离双手日常使用，不过浏览器比较特殊，不能完全脱离键盘，只能尽量少使用鼠标。

## 如何优化

文本操作可以使用数据统计常用的操作，vim 目前没有相关插件，emacs 有插件 [keyfreq](https://github.com/dacap/keyfreq) 统计各个快捷键的使用频率，根据此优化你的文本处理。也可以在日常使用时留意不是很高效的处理，随时关注如何优化文本处理流程。

本人 emacs 的使用统计：
```shell
For all major modes:

  62307    9.39%  evil-forward-word-begin                                                              w
  55557    8.37%  evil-normal-state                                                                    
  45172    6.81%  evil-backward-word-begin                                                             b
  32033    4.83%  evil-delete-backward-char-and-join                                                   
  24040    3.62%  evil-ex-search-next                                                                  n
  19654    2.96%  save-buffer                                                                          SPC f s, M-m f s, C-x C-s, <menu-bar> <file> <save-buffer>
  18872    2.84%  evil-change                                                                          
  18826    2.84%  evil-repeat                                                                          
  17920    2.70%  evil-window-right                                                                    SPC w <right>, SPC w l, C-w C-<right>, C-w C-l, C-w <right>, C-w l, M-m w <right>, M-m w l
  17256    2.60%  evil-window-left                                                                     SPC w <left>, SPC w h, C-w C-<left>, C-w C-h, C-w <left>, C-w h, M-m w <left>, M-m w h
  17213    2.59%  evil-scroll-line-down                                                                C-e
  16042    2.42%  evil-forward-word-end                                                                e
  14987    2.26%  helm-next-line                                                                       
  13888    2.09%  evil-open-below                                                                      
  13081    1.97%  evil-delete                                                                          
  12971    1.95%  exit-minibuffer                                                                      
  11979    1.81%  evil-window-middle                                                                   M
  11108    1.67%  evil-scroll-down                                                                     C-d
  10495    1.58%  spacemacs/evil-mc-paste-after                                                        
  10208    1.54%  evil-yank                                                                            y
  10079    1.52%  evil-append                                                                          
   9980    1.50%  evil-goto-first-line                                                                 g g
   8773    1.32%  recenter-top-bottom                                                                  C-l
   8559    1.29%  evil-end-of-line                                                                     <end>, $
   7765    1.17%  evil-delete-backward-word                                                            
   7064    1.06%  newline-and-indent                                                                   
   6912    1.04%  evil-undo                                                                            
   6841    1.03%  spacemacs/helm-find-files-windows                                                    
   6772    1.02%  company-complete-selection                                                           
   6707    1.01%  lazy-helm/spacemacs/helm-find-files                                                  SPC f f, M-m f f
   6612    1.00%  evil-ex-search-forward                                                               /
   6538    0.99%  evil-scroll-up                                                                       C-u
```
可以看到我大量使用 w 和 b 进行行内移动，这种操作方式比较低效，可以使用 f/F 操作来进行优化，这只是一个简单的示例。

比如使用 zsh_stats 来查看命令行使用频率：
```shell
     1	1844  18.3337%  git
     2	952   9.4651%   gst
     3	664   6.60171%  la
     4	564   5.60748%  cd
     5	531   5.27938%  npm
     6	523   5.19984%  ack
     7	481   4.78226%  gvim
     8	444   4.4144%   vim
     9	311   3.09207%  rm
    10	232   2.30662%  mv
    11	217   2.15749%  systemctl
    12	201   1.99841%  tar
    13	188   1.86916%  exit
    14	184   1.82939%  pacman
    15	182   1.8095%   ll
    16	178   1.76974%  sudo
    17	156   1.551%    neofetch
    18	146   1.45158%  shutdown
    19	136   1.35216%  nvm
    20	124   1.23285%  jobs
```
可以看到我使用 git 命令操作特别多，可以定义常用的 git 命令简写 alias 来优化工作流，比如 zsh 的 [git 插件](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git)；ack 使用也很多，可以使用更高效的搜索工具 [ripgrep](https://github.com/BurntSushi/ripgrep) 进行替代；cd 命令跳转目录也用得很多，可以考虑使用 [zsh-z](https://github.com/agkozak/zsh-z) 来进行目录切换。

以上都只是一个简单的示例，具体如何优化还是看个人操作习惯。

## 最后
本文是在 emacs 中编写的，其也是一个自由软件，我使用了 [Spacemacs](https://www.spacemacs.org/) 开箱即用，自己编写添加了 keyfreq layer 用于统计按键频率，以便进行优化。
本文只是一个简单的分享，或者只是自己的一个记录，我的常见配置都在 [dotfiles](https://github.com/jadegong/dotfiles)，其中也包括我的 archlinux 配置。开源的意义就在于无偿分享，共同进步，以及完全的自由！

