---
layout: post
title:  "如何使用vscode在WSL写R代码"
date:   2020-04-20
categories: [memo]
---

- [如何使用vscode在WSL写R代码](#如何使用vscode在wsl写r代码)
  - [为何要使用WSL](#为何要使用wsl)
  - [VSCode使用R](#vscode使用r)
  - [如何在WSL使用vscode中的R?](#如何在wsl使用vscode中的r)
  - [如何在WSL环境中使用vscode](#如何在wsl环境中使用vscode)
    - [绘图问题.](#绘图问题)

# 如何使用vscode在WSL写R代码

## 为何要使用WSL

我想很多需要接触服务器的人,往往并不会直接在服务器上写代码. 服务器是生产环境,写代码的时候不能在上面随意的测试.
但是用过unix-like系统的各种shell之后, Windows上的命令行让人觉得实在蹩脚.

但是Windows系统的office和其他的软件往往又无法替代.所以很多的人选择了osx. osx实际上就是苹果原先的桌面软件结合上unix-like的命令行, 立刻被追捧成生产力利器.

所以windows系统如果能够结合unix-like的命令行,那么也是功效加倍.
而之前的windows上想要一个unix-like的命令行可没那么容易, 比如cygwin migw这些软件虽然可以模拟一个unix的命令行环境,但是用起来门槛较高, 很麻烦. 

而在win10上,Windows提供了一个相对无痛的方案:使用WSL.

## VSCode使用R

在接触WSL之前, 我们首先看一下如何让vscode变成R编程的顺手工具.

R的原生界面功能十分有限, 可以使用Rstudio来提升使用体验,但是Rstudio的代码编辑功能比较薄弱.

VSCode是微软出品的一个免费开源的编辑器, 近来相当火爆,而它拥有的庞大插件生态让它从一个编辑器变成一个多面手. 这方面能煮咖啡的emacs 走的更远,被人称为"伪装成编辑器的操作系统".

所以为了能在vscode中愉快的使用R,需要如下的两个插件:

1. R ikuyadeu.r

2. R LSP Client 


安装好之后需要设置相关的参数. 


参数设置好之后,你需要


## 如何在WSL使用vscode中的R?

## 如何在WSL环境中使用vscode

首先vscode需要安装并且在安装的时候,和注册表相关的东西都要勾选.
安装插件 Remote-WSL, 之后 在WSL命令行中
输入 

> code.

此时如果你的WSL中安装了code的话,它会提醒你删除. 如果你没有安装code, 那么它会自动安装code server, 然后打开vscode, 但请注意这个时候编辑器的左下角已经发生了变化.

同时你的插件也可能要安装wsl版本. 


### 绘图问题.


需要python3 的 radian才好用.