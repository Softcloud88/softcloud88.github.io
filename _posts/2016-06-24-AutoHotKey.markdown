---
layout: post
title:  "AutoHotKey简明教程"
date:   2016-06-23 23:35:54
categories: tools
---

### 简单功能介绍

其实写这篇文章的原因主要是由于换了家公司，开始用windows系统，而windows平台没有诸如karabiner一类简单粗暴的改建工具，而我又习惯于用RAlt+EDSF进行方向键操作所以简单看了下AutoHotKey的官方教程，在此做以记录。

AutoHotKey有两种功能，一种叫做"hotkey"，另一种叫做"hotstring"。

hotkey字面意思为热键，你可以用它来定义快捷键，比如你想定义ctrl+i直接输出"我是天才"，又或者像我一样有需求想用alt+edsf来操作方向。

而hotstring可以用来快捷输入，比如你的偶像叫做"科比·布莱恩特"，那么你又是一个经常把偶像挂着指尖的人，就可以定义只要"kb"就直接输出"科比·布莱恩特"。当然，如果你是拳皇爱好者，那就用各种快捷键作弊吧，只要简单几行代码，简单优雅高逼格。有没有心动?! Let's check it out!

先送上官方教程的[传送门](https://autohotkey.com/docs/Tutorial.htm)，想全方位了解的同学请自行参考。

---
### 下载及安装

点击[这里](https://autohotkey.com)进入官网，然后进行傻瓜式下载安装。稍提醒一下，安装时候会让选择UNICODE or ANSI，选择UNICODE会支持非英文的字符（比如中文..）。

---

### 小试牛刀

安装好了之后，随便找个地方右键 -> 新建 -> AutoHotKey Script 随便起个名字，你会得到一个后缀名为.ahk的文件。打开它，你可以把里面的东西都先删了，然后输入一下内容：

	^i::
		Send, 我是天才
	Return

保存关闭，然后双击它，这货就开始运行了，你可以在右下角(或者右下角的三角点开)看见它的图标，我是在mac上写这篇文章了，就不截图了。然后随便找个能打字的地方(新建可txt即可)，然后在打字状态下按"ctrl + i"试一试，你会发现就这么简单，你就定义了一个打印"我是天才"的快捷键。

回来再来试一试hotstring，在原来的文件里另起一行开始输入：

	::kb::科比·布莱恩特
	
然后保存，重新启动一下脚本(把之前提到的右下角那玩意右键关了，再双击你的文件打开)，随便找个能打字的地方，输入"kb"，是不是没啥神奇的事情发生？再输入个"space"或"enter"看看，有木有，你的偶像出现在你的眼前。

怎么样，是不是很简单？

---

### 一些符号对应

---

|符号|描述|
|:-:|:-:|
|#|Win(Windows logo key)|
|!|Alt|
|^|Control|
|+|Shift|
|&|组合件的链接符号|

另外，左Ctrl是LCtrl，右Alt是RAlt，↑是Up，←是Left。这里有些需要注意的东西，举例如下：

	send, Multiple enter lines have enter been sent. ; WRONG 中间的enter不会被当做回车
	send, Multiple{enter}lines have{enter}been sent. ; CORRECT 经过{}转意，enter会被当做回车
	
若是定义快捷键时：
	
	; When making a hotkey... 
	; WRONG 这么用是错的，在这里不需要转意
	{LCtrl}::
		send, "我点击了左Ctrl"
	Return

	; CORRECT 此为正确形式
	LCtrl::
		send, "我点击了做Ctrl"
	Return


### 简单的语法说明：

经过上面几个例子，相信你已经了解了大致的语法，下面列举几个例子供你参考帮助你实现你想要的功能：

#### 1.组合键：
	
Alt + e 相当于按下↑：

	!e::
		Send, {Up}
	Return
	
Ctrl + Shift + Alt + e 相当于按下Ctrl + Shift + ← （用于向左选中一个单词）：

	^!+e::
		Send, ^+{Left}
	Return
	
左Alt + c 相当于Ctrl + c （Mac的复制是这个键位，觉得较为人性化）：

	LAlt & c::
		Send, ^c
	Return
	
#### 2.按键的按下与抬起：

	; This is how you hold 1 key down and press another key (or keys).
	; If 1 method doesn't work in your program, please try the other.
	send, ^s                     ; Both of these send CTRL+s
	send, {ctrl down}s{ctrl up}  ; Both of these send CTRL+s
	Send, {ctrl down}c{ctrl up}
	Send, {b down}{b up}
	Send, {TAB down}{TAB up}
	Send, {Up down}  ; Press down the up-arrow key.
	Sleep, 1000      ; Keep it down for one second.
	Send, {Up up}    ; Release the up-arrow key.
	
这里直接引用官方的例子，很容易理解，{按键 down}代表你的按键被按下，{按键 up}代表你的按键抬起，想要按一会儿？最后三行表示按住"↑"键持续一秒。

#### 3.关于hotstring的一点说明：

	::kb::科比·布莱恩特		；你输入kb后要输入一个EndChar（回车或者空格）才会将kb转换成你偶像
	:*:kb::科比·布莱恩特		; 你输入kb之后，直接会出现你偶像

#### 4.掉起一些事件：

	Numpad0 & Numpad1::
		MsgBox "我好帅".
	Return

	Numpad0 & Numpad2::
		Run Notepad
	Return

写入这几行，运行后点击"0 + 1"会弹出对话框并显示"我好帅"，点击"0 + 2"会掉打开一个记事本。

另外可以打开网页和应用程序，如：

	; 打开应用程序，除了一些系统应用，大部分都要输入全路径
	Run, %A_ProgramFiles%\Some_Program\Program.exe

	; 打开autohotkey的官网
	Run, https://autohotkey.com
	
想深入了解的话，点击[这里](https://autohotkey.com/docs/commands/Run.htm)查看更多相关信息。

#### 5.如果你针对游戏设置开机键发现不行，可以尝试一种或几种方法：

a.用[SendPlay](https://autohotkey.com/docs/commands/Send.htm#SendPlayDetail)，第一步，SendPlay替换Send;(参见[这里](https://autohotkey.com/docs/commands/Send.htm)),第二步，SendMode Play(参见[这里](https://autohotkey.com/docs/commands/SendMode.htm));第三步(非必须)，设置hotstring option SP(参见[这里](https://autohotkey.com/docs/Hotstrings.htm#SendMode)).

b.增加SetKeyDelay，如："SetKeyDelay, 0, 50"或者 "SetKeyDelay, 150, 150, Play"

c.如果以上都没能解决你的问题，研读一下[这里](https://autohotkey.com/docs/commands/ControlSend.htm)。

如果以上方案都没能解决你的问题，那么可以宣布你作弊未遂了...

### 结语

相信对于一些简单的需求，你简单过一眼例子，随手就可以搞的定的。如果你是输入狂人，那autohotkey也提供了更多的功能，[官方教程](https://autohotkey.com/docs/Tutorial.htm)有更多的详细介绍，包括如何写函数，创建对象等，有兴趣的童鞋，可以继续学习。