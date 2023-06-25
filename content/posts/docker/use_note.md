---
# 标题
title: "docker command use note"
# 时间
date: 2023-06-25T10:50:00+08:00
# 描述
description: "docker 命令使用笔记"
# 标签
tags: [Docker]
# 背景图
# featured_image: "/images/notebook.jpg"
# 类别
categories: Docker
# 评论功能
comment : false
# 是否部署
draft: false 
---

docker 命令示例
``` bash
docker run -itd -p 10831:8031 --shm-size 50g -v /home/wch/data:/data --gpus all --name wch_yolo_test2 ultralytics/ultralytics:latest /bin/bash
```
## 参数解释

### 交互方式运行
```bash
-itd
```


### 端口映射
``` bash
docker run -itd -p host_port:docker_port docker_image_name /bin/bash
```
映射ssh 链接时,需要将docker的22端口映射出去  

### gpu 映射
``` bash
docker run -it --gpus all
```

### docker 名字 
``` bash
--name your_docker_name
```

### 映射目录
```bash
-v host_dir:docker_dir
```
此种方式会直接覆盖掉docker中原有目录中的数据,请谨慎选择目录

### 共享内存
```bash
--shm-size 50g
```