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

以NSArray的类方法arrayWithObjects的声明为例，
iOS7之前为：
{% highlight objc %}
+ (id)arrayWithObjects:(id)firstObj, ...;
{% endhighlight %}
iOS7之后为：
{% highlight objc %}
+ (instancetype)arrayWithObjects:(id)firstObj, ...;
{% endhighlight %}

示例代码1:
{% highlight objc %}
NSDictionary *d = [NSArray arrayWithObjects:@(1), @(2), nil];
NSLog(@"%i", d.count);
{% endhighlight %}
下面开始分析示例代码1，

iOS7之前：编译器不会有任何报错。因为arrayWithObjects返回值的类型为id，它是一个可以指向所有Objective-C类的指针，所以赋值给NSDictionary类的对象没有一点错误。这也是id的一个重要用途。

那是不是说如果把arrayWithObjects的返回值改为NSArray*的话是不是就可以了呢？当然不是，如下示例代码2所示，
示例代码2
{% highlight objc %}
@interface MyArray : NSArray
@end
MyArray *array = [MyArray arrayWithObjects:@(1), @(2), nil];
{% endhighlight %}
MyArray作为NSArray的子类，如果使用arrayWithObjects返回NSArray*的话，还需要进行类型转换才能赋给array。

针对这个问题，instancetype横空出世。

iOS7之后：arrayWithObjects返回值为instancetype。表示哪个类使用该方法，返回值的类型就为该类。所以在示例代码1中，arrayWithObjects返回值的类型应为NSArray*,在Xcode5中就会提示编译错误。当然，在示例代码2中，arrayWithObjects返回值的类型应该为MyArray了。


----
###三、New in NSArray

----
###四、New in NSData

----
###五、New in NSTimer

----
###六、New in NSProgress
