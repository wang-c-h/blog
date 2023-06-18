---
title: "python Defaultdict"
date: 2023-04-16T20:39:04+08:00
draft: true
---


### python defaultdict类  

  

今天又学到了个知识点
python的 defaultdict类
是dict的一个子类，即键值对
这个类是为了防止keyError错误
这个类为字典中的key初始化一个默认值，默认为None
使用方法：
``` python
    import collections
    dic = collections.defaultdict(0)
```
其中，参数0为自定义的默认值