---
layout: post
title: 'block高级教程(一、block概述)'
category: iOS
tags: block
---

在iOS的开发过程中，我们会大量用到block。block可以作为参数，方便的实现回调；GCD使用block实现多线程等等。然而，官方文档对block的介绍却不是很详细，只是告诉了怎么去用，没说明block实现的原理。对于"^"这种符号，相信很多开发者还是很感兴趣的，为此，我查阅了一些资料，并作了汇总、提炼，写下这个教程，希望能有助于大家对block的理解。

这个教程的结构如下：


***************************

一、block概述

二、block的结构分析

三、block截获局部变量

四、block的存储

五、block如何处理循环引用


***************************

一些推荐的参考资料：

1 iOS官方文档：[https://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1 ](https://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1 )


2 Objective-C 高级编程：[http://book.douban.com/subject/24720270/](http://book.douban.com/subject/24720270/)


----
####一、概述

#####1、block的应用场景

1）block作为代码片段的封装，作为一个工作单元。

如：

{% highlight objc %}
dispatch_async(dispatch_get_global_queue(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)), ^{
       
        NSLog(@"I am a block");

    });
{% endhighlight %}

这段代码通过GCD开辟了一个子线程，线程执行的任务是通过block来实现的，从而支持多任务的处理。


2）block作为参数可以方便的实现回调。在调用函数的时候即可完成后续的处理，不需要跑到另一个地方写回调函数了，使程序逻辑更紧凑。

如：

{% highlight objc %}
[UIView animateWithDuration:1.0 animations:^{
    NSLog(@"animate ing ...");
} completion:^(BOOL finished) {
    NSLog(@"animate finish ...");

}];
{% endhighlight %}

动画执行的动作和执行完毕之后的动作都写到了一起，使程序的逻辑更加清晰。

#####2、block的定义

block第一眼看过去的特征是有一个 "^" 号，然后是 "{ }" 内的一块代码段。

block是基于C语言的扩展。是一个闭包。

Objective-C 高级编程一文中对它的定义为：带有自动成员变量的匿名函数。

对于它的功能，官方文档上的描述：

1、可以像普通函数一样带有参数；

2、可以像普通函数一样有返回值；

3、可以在block定义的词法作用域内捕获变量的值；

4、可以在block定义的词法作用域内更改捕获的变量的值；

5、在同一词法作用域内，可以和其他block共享修改的变量值；

6、在词法作用域中定义的变量，超过词法作用域后，block仍然可以使用并修改这些变量。

注：词法作用域的定义不做详述，你可以把它理解成中括号“ { } ”包围的区域。

#####3、分析block的方法

对block理解的难点在于block对变量的处理。

官方文档中描述：

在 block 里面使用变量遵循以下规则:

全局变量可访问,包括在相同作用域范围内的静态变量。

传递给block的参数可访问(和函数的参数一样)。

程序里面属于同一作用域范围的堆(非静态的)变量作为const变量(即只读)。 它们的值在程序里面的 block 表达式内使用。在嵌套 block 里面,该值在最近的封闭范围内被捕获。

属于同一作用域范围内并被__block存储修饰符标识的变量作为引用传递因此是 可变的。

属于同一作用域范围内block的变量,就和函数的局部变量操作一样。 每次调用 block 都提供了变量的一个拷贝。这些变量可以作为 const 来使用,或在 block 封闭范围内作为引用变量。 
 

为什么block能截获变量？

为什么加上__block能够修改局部变量？

block变量及__block变量的存储区域在哪？

这些问题官方文档都没给出详细的分析。为了能够弄清楚其中的原理，需要借助clang的rewrite-objc命令，获取编译的中间代码，分析block的结构以及编译器的处理过程，来帮我们更好的理解block。 

rewrite-objc的使用是很简单的，格式为：

clang -rewrite-objc xxxx

此处的xxxx为待分析的源码文件

关于这些疑问，我们会在接下来的章节中详细讨论。
