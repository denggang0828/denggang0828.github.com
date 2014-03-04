---
layout: post
title: 'block高级教程 ( 五、block如何处理循环引用 )'
category: iOS
tags: block
---

使用block时较常见的一个问题是容易引起循环引用。因为block会截获局部变量从而持有它引用的对象，而对象也会持有block，这种互相持有的情况下就会造成内存无法释放从而引起内存泄露。

当然，我们也有多种方案避免block造成的循环引用。

####1 ARC中推荐使用\_\_weak。

使用\_\_weak修饰block所引用的对象，这样block弱引用外部对象，打破了外部对象和block之间的循环引用。

####2 MMR中，与ARC中的\_\_weak对应，我们可以使用\_\_unsafe\_\_unretained修饰符。

使用它的风险是容易造成空指针错误，(与\_\_weak的区别为：如果\_\_weak变量不指向任何对象内存区域，则会自动置为nil；但是\_\_unsafe\_\_unretained不会，会指向一块未知的内存。)如果使用不当，容易引起程序崩溃。

####3 ARC和MMR中都可以使用 \_\_block修饰符。

但是在ARC和MMR中，\_\_block的用途有很大的不同，如下：

MMR中，block对\_\_block修饰的变量的处理，实际上是将其转换成\_\_Block_byref_i_0结构体。通过之前的介绍，我们知道\_\_Block_byref_i_0结构体有个指向自身的\_\_forwarding指针，当\_\_block从栈复制到堆上时，\_\_forwarding转而指向堆上的\_\_Block_byref_i，并没有使引用计数加1。所以，Block没有对\_\_block变量进行retain操作，从而避免循环引用。


ARC中，没有retainCount的概念，代替的是强引用和弱引用。只有\_\_weak和\_\_unsafe_unretained声明的变量不会被强引用，block对\_\_block变量的持有是强引用。所以，block变量持有\_\_block变量，\_\_block变量持有引用对象，对象持有block变量。看起来这还是一个环，没能解决循环引用问题。所以对于这种方案，还需要执行一下block，在block的执行体中使用完\_\_block变量后将其置为nil，打破block变量对\_\_block变量的持有从而解决循环引用。
 坏处，必须执行一次block，否则还是会循环引用。

----

下面以几段示例代码描述以上解决方案：（这里只讨论ARC有效的情况）

----
###一、问题代码

在ARC中有如下代码：

{% highlight objc %}

typedef void (^blk_t)(void);

@interface MyObject: NSObject
{
   blk_t blk_;
}
@end


@implementation MyObject

- (id)init
{
     self = [super init];

     blk_ = ^{NSLog(@"self = %@", self); };

     return blk;
}

- (void)dealloc
{
     NSLog(@"dealloc");
}
@end


int main()
{
     id o = [[MyObject alloc] init];

     NSLog(@"%@", o);

     return 0;
}

{% endhighlight %}

这是一段典型的由于block引起循环引用问题的代码。 MyObject类的对象强引用blk\_变量， blk\_又对MyObject的对象强引用，形成循环引用。

----
###二、使用__weak方案

{% highlight objc %}

- (id)init
{
     self = [super init];

     id __weak tmp = self;

     blk_ = ^{NSLog(@"self = %@", tmp); };

     return blk;
} 

{% endhighlight %}

将MyObject变量赋值给一个\_\_weak变量，blk\_使用这个\_\_weak变量。这样，blk\_变量对MyObject类对象的引用变为弱引用，从而打破了循环引用。

iOS4之前使用__unsafe_unretained与之类似，不详述。

----
####三、使用__block方案

{% highlight objc %}

typedef void (^blk_t)(void);

@interface MyObject: NSObject
{
   blk_t blk_;
}
@end


@implementation MyObject

- (id)init
{
     self = [super init];

    __block id tmp = self;

     blk_ = ^{

        NSLog(@"self = %@", tmp);
    
        tmp = nil;    
     };

     return self;
}

- (void)execBlock
{
     blk_();
}

- (void)dealloc
{
     NSLog(@"dealloc");
}
@end


int main()
{
     id o = [[MyObject alloc] init];

     [o execBlock];

     return 0;
}

{% endhighlight %}

在ARC中，必须执行block，打破block变量对__block的引用

----
###四、结论

对于block引起的循环引用：

ARC中，推荐使用\_\_weak修饰符来解决；

MMR中，推荐使用\_\_block修饰符来解决；不推荐使用\_\_unsafe_unretained。
