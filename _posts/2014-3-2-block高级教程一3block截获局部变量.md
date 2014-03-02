---
layout: post
title: 'block高级教程 ( 三、block截获局部变量 )'
category: iOS
tags: block
---


在block的内部，可以使用外部的变量。这些变量又可以分为局部变量，全局变量，静态变量等。对不同类型的变量，block对其使用及处理方式也不一样。

###一、局部变量

局部变量可以分三种情况讨论：

1 普通局部变量：block对其只读，不能修改它的值。

2 静态局部变量：block对其可读写。

3 __block类型的局部变量：block对其可读写。

为什么block对不同类型局部变量的读写特性不同？

对这个问题，我们依然可以采用clang编译获得中间代码的方式探究，详述如下：

####1 普通局部变量

重新打开文件 

vi block.m

编辑文件内容如下：

{% highlight objc %}
#import <Foundation/Foundation.h>

int main(int argc, char *argv[])
{
    int i = 2;

    void (^blk)(void) = ^{
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

这里，我导入Foundation库，并添加一个成员变量i，在block中将其打印。

保存退出

在同级目录下输入

clang -rewrite-objc -framework Foundation block.m

由于添加了 Foundation库，clang命令需要添加参数 - framework

执行结束之后会发现文件block.cpp变的巨大，别担心，这是因为引入了Foundation框架引起的，只把对我们分析有用的部分摘抄出来，如下：

{% highlight objc %}
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int i;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _i, int flags=0) : i(_i) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
     int  ; // bound by copy

 NSLog((NSString *)&__NSConstantStringImpl__var_folders_f4_6g5wrh2n5jd97jkblqdn1z0m0000gn_T_block_D8pe7B_mi_0, i);

}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

int main(int argc, char *argv[])
{
    int i = 2;

    void (*blk)(void) = (void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, i);

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

    return 0;
}
{% endhighlight %}

观察改变的地方，

1 block自身的结构体__main_block_impl_0多了一个成员变量i，并在构造函数中赋值。

2 block执行过程中__main_block_func_0，使用的i是__cself->i，即block自身的成员变量i，而不是block所在词法作用域内的局部变量i。


以上可以看出，block可以对局部变量i执行读操作。

那为什么不能修改呢？ 先让我们试图改一下看看效果。

我们再次对block.m文件进行修改

{% highlight objc %}
#import <Foundation/Foundation.h>

int main(int argc, char *argv[])
{
    int i = 2;

    void (^blk)(void) = ^{
         i = 3;
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

在block内，先把局部变量i改为3，然后打印。使用clang编译

block.m:8:11: error: variable is not assignable (missing __block type specifier)
        i = 3;
        ~ ^
1 error generated.


报错信息提示，“变量不可改，因为没有__block修饰符。”

先来看下为什么block中不能改变局部变量 i。

在上面的中间代码中。block中使用的i “i = __cself->i” ，这里的i是block结构体的一个成员变量。这个成员变量的值是从__main_block_impl_0的构造函数中得到的，由局部变量i赋值，值传递得到。

所以说，block中使用的i实际上已经不是局部变量i了，而是block中的成员变量int型的i，两个i属于不同的作用域，当然会报错。

既然值传递不行，那么可不可以地址传递呢？

在这个case中是可以的，通过指针实现对局部变量的修改。但是需要注意的是，block的大多数使用场景是作为参数回调用用，当执行回调时，如果局部变量定义时所在的作用域结束，局部变量已经被销毁，那么再去访问就会引起崩溃的。


当然，对于静态局部变量除外。因为静态局部变量是分配在静态存储区的，直到程序结束才释放内存，修改block.m

{% highlight objc %}
int main(int argc, char *argv[])
{
    static int i = 2;

    void (^blk)(void) = ^{
         i = 3;
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

编译无错误。这里我们只看block实现的结构体

{% highlight objc %}
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int *i;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int *_i, int flags=0) : i(_i) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
{% endhighlight %}

可以看到，block存储的是i的指针。通过指针操作实现对变量i的修改。

对于非静态局部变量呢？那应该怎么办呢？ 其实错误信息已经提示我们了，使用__block修饰符。

继续修改block.m

{% highlight objc %}
#import <Foundation/Foundation.h>

int main(int argc, char *argv[])
{
    __block int i = 2;

    void (^blk)(void) = ^{
         i = 3;
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

编译后的中间代码为：

{% highlight objc %}
struct __Block_byref_i_0 {
  void *__isa;
__Block_byref_i_0 *__forwarding;
 int __flags;
 int __size;
 int i;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_i_0 *i; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_i_0 *_i, int flags=0) : i(_i->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_i_0 *i = __cself->i; // bound by ref

        (i->__forwarding->i) = 3;
 NSLog((NSString *)&__NSConstantStringImpl__var_folders_f4_6g5wrh2n5jd97jkblqdn1z0m0000gn_T_block_uZk2II_mi_0, (i->__forwarding->i));
    }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

int main(int argc, char *argv[])
{
    __attribute__((__blocks__(byref))) __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 2};

    void (*blk)(void) = (void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_i_0 *)&i, 570425344);

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

    return 0;

}
{% endhighlight %}

这一下子多了好多东西：

1 __Block_byref_i_0的结构体。

2 __main_block_impl_0的成员变量i变成了__Block_byref_i_0类型。

3 修改局部变量的语句为(i->__forwarding->i) = 3;

4 两个函数__main_block_copy_0和__main_block_dispose_0

先来看第一个__Block_byref_i_0，由于它存在成员变量isa，所以它是一个对象；成员变量i实际存储值；__Block_byref_i_0 *__forwarding;__forwarding这个变量也是指向__Block_byref_i_0，有重要作用，稍后讲解。

这样实现了对局部变量的修改，但是当block回调执行时，局部变量__block i所在区域已经结束，如果i在栈上的话，也应该被销毁的？

当然不是这个样子，不然加了__block也没有效果啊。这时候起作用的是__main_block_desc_0，它的两个成员函数__main_block_copy_0和__main_block_dispose_0。

当block从栈分配到堆上时，会调用__main_block_copy_0将__block变量i从栈复制到堆上；当block释放时，会调用__main_block_dispose_0将堆上的__block变量释放。

这样，即使局部变量__block变量i所在区域已经结束，由于i已经复制到堆上，并被block持有，所以他不会被释放。

这时候解释下__forwarding得作用。__block变量i一会在栈上，一会在堆上，如果栈和堆上同时对变量进行操作怎么办呢？

__forwarding挺身而出，当一个__block变量从栈上被复制到堆上时，栈上的那个__Block_byref_i_0结构体中的__forwarding指针会指向堆上的结构。

Done

###二 全局变量，全局静态变量

再次修改block.m

{% highlight objc %}
#import <Foundation/Foundation.h>

static int i = 2;

int main(int argc, char *argv[])
{
    void (^blk)(void) = ^{
         i = 3;
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

或者

{% highlight objc %}
#import <Foundation/Foundation.h>

int i = 2;

int main(int argc, char *argv[])
{
    void (^blk)(void) = ^{
         i = 3;
        NSLog(@"i = %d", i);
    };

    blk();

    return 0;

}
{% endhighlight %}

对于全局变量和全局静态变量，clang编译后的情况是相似的，故放在一起分析。

{% highlight objc %}
static int i = 2;

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {


        i = 3;
 NSLog((NSString *)&__NSConstantStringImpl__var_folders_f4_6g5wrh2n5jd97jkblqdn1z0m0000gn_T_block_x6HM0Z_mi_0, i);
    }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};


int main(int argc, char *argv[])
{
    void (*blk)(void) = (void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA);

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

    return 0;

}
{% endhighlight %}

可见，block并没有对全局变量进行任何截获，直接操作原来的全局变量即可。
