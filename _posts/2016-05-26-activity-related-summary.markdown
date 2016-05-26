---
layout: post
title:  "Activity相关整理"
date:   2015-05-26 20:31:54
categories: components
---

## Activity的生命周期

### 正常情况生命周期：

![Activity normal life circle](http://s16.sinaimg.cn/mw690/71e00b88gda37b3c6040f&690)

当A启动B时相关的方法的调用顺序：
A的onPause->B的onCreate、onStart、onResume->A的onStop。

onPause不宜做耗时操作，可能会影响到下个Activity到前台，不过对于一些数据的保存工作建议放到onPause里，以确保被调用；同时，例如注销BrodcastReceiver之类放到onStop里就会更好些。

---

### 异常生命周期：

系统的Activity异常终止和重建时候才会调用onSaveInstanceState和onRestoreInstanceState来储存和回复数据，正常情况下不会被触发。onSaveInstanceState调用发生在onStop之前，和onPause之间没有确定顺序，onRestoreInstanceState在onCreate之后。

可以选择在onCreate和onRestoreInstanceState进行相应的重建，区别在于onCreate的Bundle savedInstaceState在正常启动时候是null，所以要判空；而onRestoreInstanceState只要被调用，其Bundle savedInstanceState就是非空的，不用额外判断。官方推荐在onRestoreInstanceState方法中来回复。

对于更改系统配置而重建Activity，可以通过在manifest中对Activty设置conficChanges属性而避免相应的配置修改引起的重建。常用的三个配置:

locale:设备的本地位置发生变化，一般指切换了系统语言。
orientation:屏幕方向发生了改变，比如旋转了手机屏幕。
keyboardHidden:键盘的可访问性发生了电话，比如用户调出了键盘。

例如：

    <activity   
        android:name="com.softcloud.ExampleActivity"
        android:configChanges="orientation|locale"/>

经过上面配置，当旋转屏幕或是调整系统语言时，这个Activity就不会被重新创建了。

---
