---
layout:     post
title:    windows python3 or python?
subtitle:   从一个神奇现象说起
date:       2021-05-09
author:     cliu
header-img:  img/post-bg-dualboot
catalog: true
tags:
    - 
---

# 神奇现象

一直以来,对于 python 和 python3,我都是这样记忆的:
linux 上 pip3 和 python3, windows 上 pip 和 python

但在做作业的时候, 忘了自己在 windows 上, 于是 `python3 ./load_data.py` 就这么顺手打出来了, 可是什么也没有发生. 经检查不是代码的问题,毕竟 `print()` 都没执行, 最后发现应该用 `python` 来执行命令. 那么问题来了:

+ 为什么 `python3` 执行时没有报错: no such command
+ 在 powershell 中输入 `python3` 后跳到了microsoft store? 但我显然不会通过此途径下载 python
+ 在 powershell 中输入 where python3 后,得到的结果是 `C:\Users\******\AppData\Local\Microsoft\WindowsApps\python3.exe`, 而输入 where python 后, 得到的结果是 `D:\ProgramData\Anaconda3\python.exe` 和 `C:\Users\******\AppData\Local\Microsoft\WindowsApps\python.exe`. 说明python3.exe 是存在的, 只不过安装的方式比较特别


于是终极问题是: 为什么用这个 python3.exe 执行代码, 不报错但也什么都没有发生? 直观感觉这个 python3.exe 的运行环境肯定是存在问题的(我最初以为是后台下载的时候网络还是什么东西出问题了导致环境不全). 或者说, 它是什么?


# 解释
本篇参考如下博客: 迷惑行为：Win10 中的 Python: https://shuhari.dev/blog/2019/11/win10-store-python

大概是因为, 微软试图让 windows 自带 python 环境(这个看上去很容易办到, 但是为啥非要以这个方式实现呢?疑惑) 它的作用在于当调用 python/python3 命令时, 自动跳转到 microsoft store 下载. 可是如下的结果确实说明这个方法不一定对每位用户都是一个好的选择:

![image.png](https://i.loli.net/2021/05/09/BR6PYEc9xl1tISF.png)

实话说用 microsoft store 下载东西好像体验一直不是很好.....

不过上述功能可以通过"应用程序别名"关闭掉:

![image.png](https://i.loli.net/2021/05/09/dHNE3uv9LmD4WIq.png)

# python3 or python
最后的问题: why `python3` on OSX and linux, but `python` on windows?

so上是这样说的:
> OSX and Linux have python executable installed by default as a rule and it refers to Python 2 version in most cases at the moment that is why you need a separate python3 name there. There is no Python on Windows by default. And therefore any version that you've installed is just python (I guess). The recommended way to manage multiple python versions is to use the Python launcher.

看来等 windows 也有自带的 python 的时候,也得用 python3 跑程序了.