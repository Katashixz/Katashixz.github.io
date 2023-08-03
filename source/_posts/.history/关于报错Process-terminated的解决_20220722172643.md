---
title: 关于报错Process terminated的解决
date: 2021-08-17 15:39:47
tags:
---
Spring boot学习打包时遇到的问题以及对应的解决方案
![1](https://img-blog.csdnimg.cn/20210309183327911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg5NTI1NA==,size_16,color_FFFFFF,t_70)
![2](https://img-blog.csdnimg.cn/20210309185627253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg5NTI1NA==,size_16,color_FFFFFF,t_70)

原帖地址来自CSDN https://blog.csdn.net/weixin_43895254/article/details/114594201

一番操作过后发现仍然报错，似乎不是版本号的问题
是在对settings.xml添加镜像时未在指定区域添加，以及添加过后未对应上下代码
例如<profiles>对应 </profiles>结尾，但是复制进来的代码中也有一个 </profiles>提前对应掉了
 
 
