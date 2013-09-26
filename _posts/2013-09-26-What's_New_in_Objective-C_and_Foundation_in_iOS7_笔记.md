---
layout: post
title: "What’s New in Objective-C and Foundation in iOS 7 笔记"
categories: ios
tags: ios7
---

今天读了matt的 [What’s New in Objective-C and Foundation in iOS 7](http://www.raywenderlich.com/49850/whats-new-in-objective-c-and-foundation-in-ios-7), 整理如下:

----
###一、Modules

modules的概念是在2012年LLVM开发者大会上首次提出的。通过优化引入framework的方式达到提高编译速度的目的。module将整个framework作为一个独立的block，以和PCH文件相似的方式引入头文件进行预编译。并且可以帮你链接相应的framework。

使用modules:

1 旧工程。Build Settings -> Apple LLVM 5.0 –Language-Modules -> Enable Modules (C and Objective-C) 将其值设置为Yes。

2 新工程。使用Xcode5新创建的工程是默认使用modules的。

使用新语法@import，代替#import。但非强制，系统会默认做这一转换。

Note:只有系统的framework可以使用modules这一特性，自己编写的或者三方的framework暂时还不能应用modules。


----
###二、New return type – instancetype

从编译器的角度加强了纠错检查机制，大大减小程序崩溃的几率。instancetype只能作为Objc方法的返回值类型，它的作用是告诉编译器这个方法返回值的类型必须为这个方法所在类的一个实例。


iOS7之前：
+ (id)arrayWithObjects:(id)firstObj, ...;

iOS7之后：
+ (instancetype)arrayWithObjects:(id)firstObj, ...;


----
###三、New in NSArray

----
###四、New in NSData

----
###五、New in NSTimer

----
###六、New in NSProgress
