---
title: "ubuntu 启动之后 黑屏问题 --- pcie bus error"
date: 2023-04-11T10:28:30+08:00
# draft: true
---

## 问题描述

    今天遇到一个问题，我的ubuntu系统在加载了启动项之后直接黑屏，ssh也无法连接到机器上
## 解决方案
    重启机器，长按shift进入到ubuntu系统的grub选项窗口
    选择高级选项，按enter进入，
    找到recovery选项，按enter进入，
    等待恢复模式启动起来之后，以root模式进入
    首先查看系统日志，进入/var/log目录下,发现有几个日志文件特别大，有几百G，然后意识到我的磁盘没有空间了
    kern.log
    sysinfo.log
    首先把两个最大的日志文件删掉,然后重启机器
    重启过程中发现命令行一直在输出以下内容
```
    pcie bus error severity=corrected type=physical layer (receiver id)
```
    临时解决方案：
        在grub中禁用错误信息
``` bash
        sudo gedit /etc/default/grub

        编辑grub，在GRUB_CMDLINE_LINUX_DEFAULT末尾添加pci=noaer。那一行应该是这样的：
        GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=noaer"

        之后更新grub
        sudo update-grub
```
    重启之后果然没有这个问题了，但是终归时临时解决方案，禁用了错误信息，cpu还是会中断处理这些信息，造成系统资源浪费，占用过高。

    分析了一下原因应该是机箱内部灰尘过多导致总线部分地方有错
    仔细清理了了以下机箱，没想到这个问题竟然解决了