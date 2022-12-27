---
title: SSD 文件系统修复
date: 2022-12-23
categories:
tags:
---

# SSD 文件系统修复

## 背景
Ubuntu 20版本以后在笔记本上似乎有bug，会有偶发性的显示卡顿的问题，具体现象为屏幕卡住不动，鼠标键盘没有响应（实际上kernel是有处理鼠标和键盘输入的），等卡顿结束后（一般是10s左右），会看到之前的键盘鼠标的输入信息，原因不明。

也许是部分因为这个原因，加上ssd在做大文件夹拷贝时（百G级别）发热巨大导致ssd读写降速甚至无法进行读写（可以看到在拷贝某个文件时卡住不动）。这种时候ssd也无法弹出，但是ssd应该也无法响应了，只能强行断电重启系统。反复一两次之后ssd就进不去系统了，只能开机显示initramfs。 这个时候ssd可能也无法正常通过内置的`fsck`命令恢复，因为ssd硬件上是有一份FTL表来管理内部存储的，这个也可能因为异常断电而导致该硬件表没有正常完成处理。

## 解决方法
不要使用ssd启动系统，准备一个usb系统盘，例如装机时的ubuntu系统盘。进入系统后，如果发现系统尝试挂载ssd（可能因为ssd文件系统损坏而卡住），则关闭自动挂载功能重新进一次系统。这样确保系统不会抢占ssd不放，导致对其恢复无法进行。

```shell
gsettings set org.gnome.desktop.media-handling automount false # 关闭自动挂载

gsettings set org.gnome.desktop.media-handling automount-open false # 关闭自动挂载并打开

gsettings set org.gnome.desktop.media-handling automount true # 开启自动挂载

gsettings set org.gnome.desktop.media-handling automount-open true # 开启自动挂载并打开
```

这时使用`sudo fdisk -l`查看一下系统是否能正常识别ssd。如果没有识别，就停止操作，把ssd一直保持通电状态半个小时后再看。因为硬件FTL表是有备份的，ssd通电后会自动检查并尝试恢复。如果长时间发现ssd还是不能被识别，那ssd可能硬件级别损坏无法恢复了。如果能被识别，那么就还能继续尝试操作。在`fdisk`识别ssd之后，应该可以看到里面的分区。分区也是在磁盘里有备份信息的，这也是能够尝试恢复的原因。

例如，fdisk显示磁盘有`/dev/sda1`, `/dev/sda2`, `/dev/sda3`三个分区，则使用fsck命令逐个尝试修复各个分区：

```shell
# -v 表明verbose， -C 表示显示处理进度
sudo fsck -v -C /dev/sda1
sudo fsck -v -C /dev/sda2
sudo fsck -v -C /dev/sda3
```

该命令会显示可能修复的方法，按照提示输入yes/no即可自动恢复分区。

## 总结
上述方法仅针对磁盘分区能被识别的情况下可能起作用，具体还需要根据实际的故障现象查阅尽可能多的资料分析。
