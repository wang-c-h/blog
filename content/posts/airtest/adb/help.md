---
title: "Help"
date: 2023-04-19T15:24:21+08:00
draft: True
---

## windows环境下通过 adb 安装apk
进入到platform-tools目录下，即adb.exe同级目录,打开终端或者powershell  
``` shell
创建临时目录
    .\adb.exe shell mkdir /data/local/tmp/apk
将本地安装包 push 到设备上
    .\adb.exe push apk_path /data/local/tmp/apk/base.apk
安装apk
    .\adb.exe shell pm install install_options /data/local/tmp/apk/base.apk
```
