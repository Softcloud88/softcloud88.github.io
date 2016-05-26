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

###Activity的启动模式：

#### LaunchMode:
standard:在启动该Activity的Activity的栈中启动，非Activity的Context（如ApplicationContext）没有任务栈就会报错，相应的解决方法是设定"FLAG\_ACTIVITY\_NEW_TASK"标志位创建新的任务栈，实际上是用了singleTask模式。

singleTop:在栈顶时不会被重新创建，onNewIntent方法会被调用，可以利用该方法取出当前请求的信息。

singleTask: 若存在实例，则其上面的Activity会出栈，同时onNewIntent方法会被调用。该模式下，系统会先寻找是否存在其想要的任务栈，若没有会创建任务栈。

singleInstance:singleTask的加强版，该模式下的Activity单独位于一个任务栈。

可以通过设置TaskAffinity属性来制定栈名，默认的栈名是应用的包名。该属性主要和singleTask启动模式或者allowTaskReparenting属性配对使用。

设置启动模式的方式有两种，分别为在manifest里设置和启动时设置标志位。后者的优先级大于前者，同时，两种方式的限定范围也稍有区别。

    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    
* FLAG\_ACTIVITY\_NEW\_TASK: 同singleTask;
* FLAG\_ACTIVITY\_SINGLE\_TOP: 同singleTop;
* FLAG\_ACTIVITY\_CLEAR\_TOP: 若同singleTask一起则与singleTask无区别，若与standard一起，则它及它上的都出栈，系统重新创建实例放入栈顶。
* FLAG\_ACTIVITY\_EXCLUDE\_FROM\_RECENTS: 同android:excludeFromRecets="true"，不会出现在历史Activity列表中。

---

### IntentFilter匹配：

#### 匹配规则：

    Intent intent = new Intent("com.softcloud.blog");
    intent.addCategory("com.softcloud.myCategory");
    intent.setDataAndType(Uri.parse("file://abc"), "text/plain");
    startActivity(intent);

* action: 必须有一个，且必须与IntentFilter中的某个相同；
* category: 可以没有也可以有多个，没有时候默认为android.intent.category.DEFAULT，多个的时候必须每个都与IntentFilter中的某个相同才能匹配；
* data: 若IntentFilter指定了data，那么Intent重必须也要定义可匹配的data才可匹配。data包含mimeType和URI两部分。要注意的是，setData(Uri data)会把mimeType设为null，同理setType会把URI设为null，可调用setDataAndType同时设置两项。在IntentFilter中data的类型：

    <data android:scheme="string"
        android:host="string"
        android:port="stirng"
        android:path="string"
        android:pathPattern="string"
        android:pathPrefix="string"
        adnroid:mimeType="string" />

#### 匹配检验：

隐式启动Activity时候可以做一下判断，看是否有可匹配的Activity，若不判断有可能会出错。判断方法有两种：

PackageManager的resolveActivity方法:

    public abstract List<ResovleInfo> queryIntentActivities(Intent intent, int flags);

Intent的resolveActivity方法：

    public abstract ResolveInfo resolveActivity(Intent intent, int flags);
    
其中第一个参数无需解释，第二个要使用MATCH\_DEFAULT\_ONLY，原因在于category不含有DEFAULT的Activity无法接受隐式的Intent，所以要把无DEFAULT的过滤掉避免启动失败。

>#### 本文参考资料：《Android开发艺术探索》 任玉刚
