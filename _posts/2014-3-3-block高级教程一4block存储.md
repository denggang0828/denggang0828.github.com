---
layout: post
title: 'block高级教程 ( 四、block的存储 )'
category: iOS
tags: block
---


对于一个block的内存，它可以分配在堆、栈、数据区上。当创建一个block时，如果是在全局区域中，则block是分配在数据区的，否则分配在栈上。

结合之前几节clang编译之后的中间代码，注意到在block的结构体__main_block_impl_0中的构造函数，有这么一句

{% highlight objc %}
impl.isa = &_NSConcreteStackBlock;
{% endhighlight %}

这里的_NSConcreteStackBlock表示的是block的内存是分配在栈上的。

除了NSConcreteStackBlock，block还有NSConcreteMallocBlock和NSConcreteGlobalBlock 

表示分配在堆上和数据区。


####分配在栈上

我们知道block也是一个对象。在Cocoa中，大多数情况下新生成的对象，是在堆上开辟一块内存区域。但block是个例外，生成一个block时，他的内存默认是在栈上的。所以前面几节给出的例子，impl.isa = &_NSConcreteStackBlock 都是这个标示。

所以，更严格的来说，block是一个仿对象。因为它的isa是指向NSConcreteStackBlock而不是objc_class。

####分配在堆上

由于创建block默认是分配在栈上的，在它作为返回值等情况时，为了能够正常使用，需要对它进行一次copy操作，copy操作的目的是把block从栈上复制到堆上。

也就说是，如果不执行copy操作，则在栈展开时block就已经被释放，再去执行就会崩溃，如果想长期使用，请复制。

所以，将block分配到堆上是解决这一问题的方法。

大多数情况下，编译器会恰当的做出判断，帮助我们将block从栈复制到堆上，如

1、Cocoa框架方法且方法名中含有usingBlock时。

2、GCD中的API。

其他情况下，就得需要我们自己对block进行copy操作，如：

例子如下：

{% highlight objc %}
- （id）getBlockArray
{
     int val = 10;
     
     return [[NSArray alloc] initWithObjects:
          ^{NSLog(@"blk0:%d",val);},
          ^{NSLog(@"blk0:%d",val);},nil
     ];
}

id obj = getBlockArray();
typedef void (^blk_t)(void);
blk_t blk = (blk_t)[obj objectAtIndex:0];
blk();
{% endhighlight %}

程序会crash，因为getBlockArray函数执行结束时，栈上的Block被废弃。这时候又去调用调用[obj objectAtIndex:0]，造成程序错误。

因为在这种情形下，编译器判断是不需要复制block到堆上。需要自己手动进行复制。

当然，我们也可以强行设置编译器，使它在所有情况下都复制block到堆上。但是，这一过程是很废CPU的，如果block在栈上也可以使用，这部分的工作就浪费了。

要让上述代码可正常运行，更改如下：

{% highlight objc %}
- （id）getBlockArray
{
     int val = 10;
     
     return [[NSArray alloc] initWithObjects:
          [^{NSLog(@"blk0:%d",val);} copy],
          [^{NSLog(@"blk0:%d",val);}copy],nil
     ];
}

id obj = getBlockArray();
typedef void (^blk_t)(void);
blk_t blk = (blk_t)[obj objectAtIndex:0];
blk();
{% endhighlight %}

所以说，如果要使用block作为返回值：确保它在返回时被复制到堆上，从而能继续使用它；确保它是一个autorelease类型，避免内存泄露。

####分配在数据区

最后，看一下分配在数据区的情况。如果block是在全局区域创建的，那么它分配在NSConcreteGlobalBlock上。

如下：

{% highlight objc %}
void (^blk)(void) = ^{

};

int main(int argc, char *argv[])
{
    blk();
    return 0;
}
{% endhighlight %}


clang编译的中间代码为：

{% highlight objc %}
struct __blk_block_impl_0 {
  struct __block_impl impl;
  struct __blk_block_desc_0* Desc;
  __blk_block_impl_0(void *fp, struct __blk_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteGlobalBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __blk_block_func_0(struct __blk_block_impl_0 *__cself) {


}

static struct __blk_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __blk_block_desc_0_DATA = { 0, sizeof(struct __blk_block_impl_0)};
static __blk_block_impl_0 __global_blk_block_impl_0((void *)__blk_block_func_0, &__blk_block_desc_0_DATA);
void (*blk)(void) = (void (*)())&__global_blk_block_impl_0;

static int i = 3;

int main(int argc, char *argv[])
{
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
{% endhighlight %}

可以看到确实为_NSConcreteGlobalBlock。因为在全局区域只能使用全局变量，所以不存在对局部变量的截获。整个程序生命周期中存在一个实例即可，因为将它放到全局区域是合理的。

另外，有一个特殊情况需要注意，如果一个block没有使用任何局部变量，则 它也是分配在NSConcreteGlobalBlock上，虽然中间代码会给出 NSConcreteGlobalBlock。
