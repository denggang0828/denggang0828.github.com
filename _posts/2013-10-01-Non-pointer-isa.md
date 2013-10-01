---
layout: post
title: "Non-pointer isa"
categories: ios
tags: isa
---
最新一期的iOS Dev Weekly中有篇文章比较有趣，[Non-pointer isa](http://www.sealiesoftware.com/blog/archive/2013/09/24/objc_explain_Non-pointer_isa.html?utm_source=iOS+Dev+Weekly&utm_campaign=iOS_Dev_Weekly_Issue_113&utm_medium=email)。

苹果发布了iPhone 5s 和iPhone 5c，采用了64位的处理器。作者在这篇文章中指出，在采用64位处理器的iOS的系统中，Objective-C类对象的isa将不再是一个指针。

比较了Xode4.6和Xcode5中关于isa的定义

Xcode 4.6
{% highlight objc %}
@interface NSObject <NSObject> {
    Class	isa;
}

typedef struct objc_class *Class;
typedef struct objc_object {
    Class isa;
} *id;
{% endhighlight %}

Xcode 5
{% highlight objc %}
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
typedef struct objc_class *Class;
{% endhighlight %}

比较发现，Xcode 5中增加了OBJC_ISA_AVAILABILITY的标示，OBJC_ISA_AVAILABILITY代表的含义如下:

{% highlight objc %}
/* OBJC_ISA_AVAILABILITY: `isa` will be deprecated or unavailable 
 * in the future */
#if !defined(OBJC_ISA_AVAILABILITY)
#   if TARGET_OS_IPHONE
#       define OBJC_ISA_AVAILABILITY  __attribute__((deprecated))
#   else
#       define OBJC_ISA_AVAILABILITY  /* still available */
#   endif
#endif
{% endhighlight %}
可以看到，苹果官方文档都将isa标为即将弃用。

作者在文中提到如下几点：

###既然isa不再是个指针，那么它现在是什么？

不管是在OS X还是在iOS系统中，都不会使用完虚拟地址空间的全部64位。所以，其中的一些位仍然是object’s class的指针；另外，Objective-C的运行时环境会使用剩余的一些位用来存储object的一些信息，比如引用计数或者其是否弱引用等。

###这么做的目的？

提高性能。充分利用这些剩余的位计算，可以大大的提高程序运行的速度，也可以减小内存的使用。在iOS7中，主要目的是优化retain/release和alloc/dealoc的性能。

###编码时需要注意什么？

不要直接使用obj->isa，否则编译器会报错。

读写操作分别使用：

[obj class] or object_getClass(obj)

object_setClass()
				  
如果覆写allocWithZone，会将object的isa模块初始化为isa指针，这样isa模块就不能存储额外的信息从而能达不到优化的目的。应该使用object_setClass()来对object初始化。

