---
title: 'git reset,checkout,revert使用区别'
date: 2017-01-25 08:55:52
categories: tech
tags: [git,shell]
---
## git常用命令reset, checkout, revert
在平时使用git版本管理中，很多时候会涉及到版本回退，查看旧版本，放弃某一个版本，以及放弃本地修改等的情况，这需要使用git的三个命令：reset, checkout, revert.
## commit层面
``git reset``和``git checkout``的参数决定了它们的作用域。如果没有包含文件路径，便会在提交层面生效。
### reset
在提交层面，``git reset``将一个分支的末端指向另一个提交。意思就是将该分支回滚，而且不保留回滚目标之后的提交，可以用来移除当前分支的一些提交，例如舍弃branchName分支的最后两个提交：<!-- more -->
```shell
git checkout branchName
git reset HEAD~2
```
通过参数来修改分支提交，缓存区或工作目录：
- --soft : 缓存区和工作目录都不会改变，只改变分支提交
- --mixed : __默认选项__。缓存区和分支提交改变，工作目录不变
- --hard : 三个空间都会被更改
![](http://7xnyb9.com1.z0.glb.clouddn.com/git-reset.svg)
由于``git reset``会舍弃部分分支提交，所以__不适合在主分支上进行操作__
### checkout
``git checkout``传入分支名时，会切换当前工作目录至指定的分支，不会修改分支。例如切换工作目录至两个版本前：
```shell
git checkout HEAD~2
```
### revert
revert的作用也是将分支回滚到之前的指定版本，与reset不同的是，revert会新建一个修改至指定版本，然后提交，意思就是不会舍弃之前的修改，通过添加一个修改提交，来进行版本回滚。
![](http://7xnyb9.com1.z0.glb.clouddn.com/git-revert.svg)
例如将版本回滚至两个版本之前，并保留所有修改：
```shell
git checkout branchName
git revert HEAD~2
```
## File层面
``git reset``和``git checkout``命令如果检测到文件路径参数，行为会作用于指定的文件。
### reset
当参数为文件路径时，``git reset``将缓存区同步到你指定的那个版本的提交（不会影响工作目录）。例如将倒数第二个提交的demo.json加入到缓存区：
```shell
git reset HEAD~2 demo.json
```
``git reset``的参数--soft, --mixed, --hard在文件层面__没有作用__
### checkout
``git checkout``对于文件层面，只会修改工作目录的文件至指定版本，分支不会切换。
## 总结
参考表格：
<table>
    <tr><th>命令</th><th>作用域</th><th>常用场景</th></tr>
    <tr><td>git reset</td><td>提交层面</td><td>在私有分支上舍弃一些没有提交的更改</td></tr>
    <tr><td>git reset</td><td>文件层面</td><td>将文件从缓存区中移除</td></tr>
    <tr><td>git checkout</td><td>提交层面</td><td>切换分支或查看旧版本</td></tr>
    <tr><td>git checkout</td><td>文件层面</td><td>舍弃工作目录的修改</td></tr>
    <tr><td>git revert</td><td>提交层面</td><td>在公共分支上回滚更改</td></tr>
    <tr><td>git revert</td><td>文件层面</td><td>(然而并没有)</td></tr>
</table>
