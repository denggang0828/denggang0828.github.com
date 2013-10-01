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

