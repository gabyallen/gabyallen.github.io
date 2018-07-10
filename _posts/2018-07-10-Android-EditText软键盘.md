---
layout:     post                    # 使用的布局（不需要改）
title:       Android EditView软件盘的使用
    # 标题 
subtitle:      EditText是否自动弹出输入法虚拟键盘(软键盘)的问题  #副标题
date:       2018-7-10             # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - EditView
    - 虚拟键盘
    - 软键盘
---

#  Android EditText是否自动弹出输入法虚拟键盘(软键盘)的问题

##在开发过程中，我们经常会碰到Acivity中包含EditText控件时会自动弹出虚拟键盘的情况的情形，其实这是由于EditText自动获取焦点的缘故，只要让EditText失去焦点就行了，解决办法如下：

### 1.在清单文件中Manifest.xml中相应的Activity添加如下代码即可：

android:windowSoftInputMode="stateHidden"

### 2.让EditText失去焦点，用EditText的clearFocus即可：

EditText edit = (EditText)findViewById(R.id.edit);
edit.clearFocus();

### 3.强制隐藏Android输入法窗口，代码如下：

EditText edit = (EditText)findViewById(R.id.edit);
InputMethodManager imm = (InputMethodManager)getSystemService(INPUT_METHOD_SERVICE);
imm.hideSoftInputFromWindow(edit.getWindowToken(), 0);

### 4.EditText始终不弹出虚拟键盘：设置一下属性：

EditText edit = (EditText)findViewById(R.id.edit);
edit.setInputType(InputType.TYPE_NULL);

---------------------我是分割线-------------------------------

## 但有时，我们的确实在想让EditText自动获取焦点并弹出软键盘，在设置了EditText自动获得焦点后，软键盘不会弹出。

###注意：此时是由于刚跳到一个新的界面，界面未加载完全而无法弹出软键盘。此时应该适当的延迟弹出软键盘，如500毫秒（保证界面的数据加载完成，如果500毫秒仍未弹出，则延长至1000毫秒）。可以在EditText后面加上一段代码，代码如下：

		Timer timer = new Timer();
		timer.schedule(new TimerTask() {
 
			public void run() {
				InputMethodManager inputManager = (InputMethodManager) editText.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
				inputManager.showSoftInput(editText, 0);
			}
 
		}, 500);

### 可以在manifest中对应的Activity加入属性：

android:windowSoftInputMode="adjustResize"


参考资料转载：
<https://blog.csdn.net/cshichao/article/details/8536961/>

