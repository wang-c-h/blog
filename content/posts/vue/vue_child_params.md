---
title: "vue 子组件传参问题"
date: 2023-04-11T10:28:49+08:00
# draft: true
---

#### 父子组件传参
    情况描述：
        父组件下有一个dialog的子组件，需要监听子组件中的关闭操作
    解决方法：
        在父组件中调用子组件时，使用*v-on*监听一个参数，在js函数中调用方法关闭dialog。具体用法：
``` vue
    v-on:paramsVisible='listenToEditeChild'
```
``` javaScript
    listenToEditeChild() {
        this.dialogvisible = false;
        this.updateData();
    },
```
    子组件中触发事件：
``` javaScript
    this.$emit('paramsVisible')
```
