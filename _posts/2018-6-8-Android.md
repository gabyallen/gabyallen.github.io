---
layout:     post                    # 使用的布局（不需要改）
title:       Android——getPreferences()与getSharedPreferences()区别
    # 标题 
subtitle:    getPreferences()与getSharedPreferences()区别  #副标题
date:       2018-6-8              # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
---

# Android——getPreferences()与getSharedPreferences()区别

##SharedPreferences文件存储共享变量的文件路径位于“/data/data/应用程序包/shared_prefs/”目录下 
###首先从调用层次上来分： 
1.getPreferences是Activity调来的
2.getSharedPreferences是Content调用的

###从文件上来区分：
1.getPreferences(int mode)不需要传递文件名，默认使用不带包名的类名作为文件名（即Activity java文件名，不带后缀）。 
2.getSharedPreferences(String name,int mode)需要提供文件名，当然是以提供的name作为文件名 

##注意：使用getPreferences只能在当前的activity中使用，如果想要在不同activity种使用则需要使用getSharedPreferences就可以了。


