---
layout:     post                    # 使用的布局（不需要改）
title:      Android中框架MVC、MVP、MVVM区别    # 标题 
subtitle:     MVC、MVP、MVVM联系  #副标题
date:       2018-5-29              # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
---

# android中使用的框架模式MVC、MVP、MVVM区别

##算来学习Android开发已有2年多的历史了，在这2年多的学习当中，基本掌握了Android的基础知识。越到后面的学习越感觉困难，一来是自认为android没啥可学的了（自认为的，其实还有很多知识科学），二来网络上的很多框架已经帮我们做了太多的事情了，我们只需要画画UI就可以了，感觉Android开发没有太多的技术含金量。最近闲来无事，开始总结之前学过的知识点，想着是否应该学点其他的东西呢？总不能局限于Android基础知识吧。慢慢的探索发现在大的项目工程中，一个好的框架，好的设计模式，能减少很大的工作量。因此接下来两篇博客来学习一下Android中常用的两种框架设计模式 MVC和MVP。

##Mvc概念
M指的是model,V指的是View，C指的是controller,是一种常见的设计模型；用一种业务逻辑，数据逻辑，页面，然后把他们集合在一起。其中M层处理业务逻辑，数据，V处理界面显示结果；C作为桥梁，指得是activity。

###for mvc android 
M层：可以做一些业务逻辑，eg.数据库操作，耗时处理；网络造作；复杂的算法等
V层：可以画一些界面，就是XML中写界面
C层：是桥梁，指的是Activity,在Android中，Activity处理用户交互问题，因此可以认为Activity是控制器，Activity读取V视图层的数据（eg.读取当前EditText控件的数据），控制用户输入（eg.EditText控件数据的输入），并向Model发送数据请求（eg.发起网络请求等）。

## MVC使用总结
利用MVC设计模式，使得这个天气预报小项目有了很好的可扩展和维护性，当需要改变UI显示的时候，无需修改Contronller（控制器）Activity的代码和Model（模型）WeatherModel模型中的业务逻辑代码，很好的将业务逻辑和界面显示分离。

在Android项目中，业务逻辑，数据处理等担任了Model（模型）角色，XML界面显示等担任了View（视图）角色，Activity担任了Contronller（控制器）角色。contronller（控制器）是一个中间桥梁的作用，通过接口通信来协同 View（视图）和Model（模型）工作，起到了两者之间的通信作用。

什么时候适合使用MVC设计模式？当然一个小的项目且无需频繁修改需求就不用MVC框架来设计了，那样反而觉得代码过度设计，代码臃肿。一般在大的项目中，且业务逻辑处理复杂，页面显示比较多，需要模块化设计的项目使用MVC就有足够的优势了。

4.在MVC模式中我们发现，其实控制器Activity主要是起到解耦作用，将View视图和Model模型分离，虽然Activity起到交互作用，但是找Activity中有很多关于视图UI的显示代码，因此View视图和Activity控制器并不是完全分离的，也就是说一部分View视图和Contronller控制器Activity是绑定在一个类中的。

MVC的优点：

(1)耦合性低。所谓耦合性就是模块代码之间的关联程度。利用MVC框架使得View（视图）层和Model（模型）层可以很好的分离，这样就达到了解耦的目的，所以耦合性低，减少模块代码之间的相互影响。

(2)可扩展性好。由于耦合性低，添加需求，扩展代码就可以减少修改之前的代码，降低bug的出现率。

(3)模块职责划分明确。主要划分层M,V,C三个模块，利于代码的维护。
转载
<https://blog.csdn.net/feiduclear_up/article/details/46363207/>
源码下载：<https://github.com/gabyallen/Mydialog/>

##MVP概念
其实Mvp与mvc区别就是c与p的区别，因为他们联系
Presenter 替换掉了Controller，不仅仅处理逻辑部分。而且还控制着View的刷新，监听Model层的数据变化。这样隔离掉View和Model的关系后使得View层变的非常的薄，没有任何的逻辑部分又不用主动监听数据，被称之为“被动视图”。

即View层触发操作通知到M层业务层完成逻辑处理，业务层完成业务逻辑之后通知Model层更新数据，数据更新完之后通知View层展现。在实际运用中人们发现View和Model之间的依赖还是太强，希望他们可以绝对独立的存在，慢慢的就演化出了MVP。

我个人认为如果绑定适配器后就是使用Mvc模式，使用容器来加载数据就是MVP模式；我觉得这就是他们之间的交互差别。

##MVVM概念
至于MVVM基本上和MVP一模一样，感觉只是名字替换了一下。他的关键技术就是今天的主题(Data Binding)。View的变化可以自动的反应在ViewModel，ViewModel的数据变化也会自动反应到View上。这样开发者就不用处理接收事件和View更新的工作，框架已经帮你做好了。

##Data Binding Library
今年的Google IO 大会上，Android 团队发布了一个数据绑定框架（Data Binding Library）。以后可以直接在 layout 布局 xml 文件中绑定数据了，无需再 findViewById 然后手工设置数据了。其语法和使用方式和 JSP 中的 EL 表达式非常类似。 下面就来介绍怎么使用Data Binding Library。

目前，最新版的Android Studio已经内置了该框架的支持，配置起来也很简单，只需要编辑app目录下的build.gradle文件，添加下面的内容就好了

android {
    ....
    dataBinding {
        enabled = true
    }
}


