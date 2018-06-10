---
layout:     post                    # 使用的布局（不需要改）
title:       Android return用法
    # 标题 
subtitle:    return用法  #副标题
date:       2018-6-3              # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
---

# Android return用法
##1.返回方法的数值（这个数值一般固定）
##2.结束方法执行（直接写个return）
如：写个简单for循环
for(int i=0;i<6;i++){

if(i==5){
  return;
}

System.out.println(i);

}
当i=5时，就会跳出方法体；


