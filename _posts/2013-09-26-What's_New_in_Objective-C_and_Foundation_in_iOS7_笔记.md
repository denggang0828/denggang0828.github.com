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

从编译器的角度加强了纠错检查机制，大大减小程序崩溃的几率。instancetype只能用于Objc中方法的返回值类型，它的作用是告诉编译器这个方法返回值的类型必须为调用这个方法的类的类型。

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

iOS7之前：编译器不会有任何报错。因为arrayWithObjects返回值的类型为id，它是一个可以指向所有Objective-C类的指针，所以赋值给NSDictionary* 类的对象没有一点错误。这也是id的一个重要用途。

那是不是说如果把arrayWithObjects的返回值改为NSArray*的话是不是就可以了呢？当然不是，如下示例代码2所示，
示例代码2
{% highlight objc %}
@interface MyArray : NSArray
@end
MyArray *array = [MyArray arrayWithObjects:@(1), @(2), nil];
{% endhighlight %}
MyArray作为NSArray的子类，如果使用arrayWithObjects返回NSArray*的话，还需要进行类型转换才能赋给array。

针对这个问题，instancetype横空出世。

iOS7之后：arrayWithObjects返回值为instancetype。表示哪个类使用该方法，返回值的类型就为该类的指针。所以在示例代码1中，arrayWithObjects返回值的类型应为（NSArray*）,在Xcode5中就会提示编译错误。

当然，在示例代码2中，arrayWithObjects返回值的类型应该为（MyArray*）了。


----
###三、New in NSArray

开放了
{% highlight objc %}
- (id)firstObject NS_AVAILABLE(10_6, 4_0);
{% endhighlight %}
这一接口。

可以发现，从iOS4.0之后就可以支持firstObject，但苹果一直没公开，直到iOS7。
[array firstObject] 与 array[0]相比的话，还是推荐使用前者。因为如果array为空的话，前者返回的是nil，后者程序直接崩溃。后者使用的话还需要加一些检查的逻辑。

----
###四、New in NSData

增加了base64编码的相关接口，可以再也不用去找三方库了。
{% highlight objc %}
@interface NSData (NSDataBase64Encoding)
/* Create an NSData from a Base-64 encoded NSString using the given options. By default, returns nil when the input is not recognized as valid Base-64.
*/
- (id)initWithBase64EncodedString:(NSString *)base64String options:(NSDataBase64DecodingOptions)options NS_AVAILABLE(10_9, 7_0);
/* Create a Base-64 encoded NSString from the receiver's contents using the given options.
*/
- (NSString *)base64EncodedStringWithOptions:(NSDataBase64EncodingOptions)options NS_AVAILABLE(10_9, 7_0);
/* Create an NSData from a Base-64, UTF-8 encoded NSData. By default, returns nil when the input is not recognized as valid Base-64.
*/
- (id)initWithBase64EncodedData:(NSData *)base64Data options:(NSDataBase64DecodingOptions)options NS_AVAILABLE(10_9, 7_0);
/* Create a Base-64, UTF-8 encoded NSData from the receiver's contents using the given options.
*/
- (NSData *)base64EncodedDataWithOptions:(NSDataBase64EncodingOptions)options NS_AVAILABLE(10_9, 7_0);
@end
{% endhighlight %}

----
###五、New in NSTimer

NSTimer是作为资源添加到runloop中的，如果timer过多的话，CPU就要不断的去响应这些事件，CPU消耗可能会过大。所以引入了tolerance这一属性，对所有的timer进行归并，在可允许的范围内尽量的集中处理。
{% highlight objc %}
- (NSTimeInterval)tolerance;
- (void)setTolerance:(NSTimeInterval)tolerance;
{% endhighlight %}

----
###六、NSProgress

这是iOS7新添加的类。用于记录一个任务集中每个任务完成的状态。使用KVO可以监听到这些状态供我们使用。

NSProgress内部结构类似于一个树，存在父任务与子任务的概念，加上完善的通知机制，可以处理复杂关系的多任务。


