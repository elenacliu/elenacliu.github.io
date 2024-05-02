---
title: 记录 Apple 屏幕镜像功能的 bug
date: 2024-05-02
math: true
image:
  placement: 2
---

# Background
近日需要用到苹果自带的 AirPlay 功能，实现 Macbook Air 朝 iPad 投屏。但实际使用时出现找不到设备的现象。

如果查询[用户手册](https://support.apple.com/zh-cn/guide/mac-help/mchlf3c6f7ae/mac)，会发现它的步骤并不包含可能出现问题的原因。

# DAY 1
环境1: 两台设备已经使用同一个 Apple ID 进行登录，并且在同一个局域网（使用的是家里的路由器，此处称为 Wi-Fi A）下，AirPlay/AirDrop/Bluetooth 开关均已打开。iPad 系统是当前最新的 iPadOS 17.4.1，Macbook Air 没有升级到最新小版本。

现象1: 发现 Macbook Air 的屏幕镜像无法找到 iPad，但是 iPad 可以找到 Macbook Air。只能实现单向投屏。

# DAY 2
环境2: 此时怀疑是 Macbook Air 的系统不是最新导致的，于是升级了它的系统。其余条件不变，但是由于是第二天，路由器有每天重启的习惯，所以网络条件不能保证和前一天完全一致。

现象2: 两台设备均无法找到对方。


环境3: 将两台设备分别重启若干次，并且断连和重连了若干次 Wi-Fi A

现象3: 两台设备均无法找到对方。

环境4: 将两台设备重新连接到个人热点 B。其中 iPad 没有连接过这个热点，但 Macbook Air 连接过。

现象4: 两台设备可以找到对方，问题解决。此时基本可以确定是网络问题。

环境5: 重新连接 Wi-Fi A。

现象5: 因为两台设备已经找到过对方了，所以这时候切换 Wi-Fi A 也没有任何问题。这么看应该也不是完全是路由器的设置问题，否则找到了设备也无法投屏，但这种表现还是非常奇怪。

环境6: 切换 Macbook Air 为个人热点 B，iPad 依旧是 Wi-Fi A。

现象6: 看起来不在同一个局域网下还是能够实现屏幕镜像。

其实我怀疑环境4这里可以尝试将 forget Wi-Fi A，然后重连，也许效果是一样的。但是因为我不知道怎么让设备忘记这个 history，所以没法再尝试了。如果有人碰巧看到了这篇博客，可以自己试试。