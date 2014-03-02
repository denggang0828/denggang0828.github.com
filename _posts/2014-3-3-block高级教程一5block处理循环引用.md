---
layout: post
title: 'block高级教程 ( 五、block如何处理循环引用 )'
category: iOS
tags: block
---

使用block时比较容易引起循环引用，因为block会自动持有它引用的对象，对象也会持有block，从而引起内存泄露

1 ARC中用__weak

2 __block。 坏处，必须执行一次block，否则还是会循环引用。

其中__forwarding在初始化的时候是指向自身的，当NSConcreteStackBlock类型的Block通过copy变为NSConcreteMallocBlock时，栈空间的Block的__forwarding指针会改变指向堆空间的Block。

####1 使用__weak
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

这里 MyObject类强引用blk_， blk_又对MyObject的对象强引用，形成循环引用。


改正：

使用__weak

{% highlight objc %}
- (id)init
{
     self = [super init];

     id __weak tmp = self;
     blk_ = ^{NSLog(@"self = %@", tmp); };
     return blk;
} 
{% endhighlight %}

ios4之前使用__unsafe_unretained

####2 使用__block

在ARC和非ARC情况下都可以使用。

ARC情况下最好用__weak

非ARC:
在不用__block的说明符的情况下，当Block发生copy的时候会retain Block内部用到的对象实例。在用了__block说明符的情况下，打包成的__Block_byref_i_0只是记录的用到的对象的指针，在copy的时候并没有retain结构体内部的对象实例。

ARC：需要执行block打破引用。

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
     return blk;
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
     NSLog(@"%@", o);
     return 0;
}
{% endhighlight %}
必须执行block

####3 对比：

__block优点：
 可控制对象的持有期间
在不能使用__weak修饰符的环境中不使用__unsafe_unretained即可（不必担心悬垂指针）
在执行block时可动态决定是否将nil或其他对象赋值给__block变量中

__block缺点：
为避免循环引用必须执行block


根据block的具体用途选择合适的方法。








:wq



