---
layout: post
title: 'block高级教程 ( 二、block结构分析 )'
category: iOS
tags: block
---


新建一个文件，vi block.m
 
编辑文件内容如下：

{% highlight objc %}
int main(int argc, char *argv[])
{
    void (^blk)(void) = ^{
        //do nothing
    };
    blk();
    return 0;
}
{% endhighlight %}

保存退出，在同级目录下输入 clang -rewrite-objc block.m，执行结束之后会发现多了一个文件block.cpp，这个文件中包含了clang编译的中端代码信息，把其中对我们分析有用的部分摘抄出来，如下：

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
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

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

创建block：
   
可以看到，创建block的语句：

void (^blk)(void) = ^{    //do nothing   };

转换成：

void (*blk)(void) = (void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA);

简单点看，
    
blk = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);



__main_block_impl_0是根据block所在的函数名称（main）和出现的位置（第0个）来命名的。

 
可以看到，block自身是一个结构体，对应此处，为结构体__main_block_impl_0

__main_block_impl_0有两个成员变量__block_impl结构体和__main_block_desc_0结构体，构造函数完成对成员变量的初始化。


__block_impl是block的实现，由于存在成员变量isa，可以判断block也是一个对象。

它的另外一个比较重要的成员变量为FuncPtr，是一个函数指针。


__main_block_desc_0，是block的一些描述信息。


调用block是通过__main_block_func_0，参数为__main_block_impl_0的指针，即为block自身，这是一个以block自身作为参数的函数执行过程。


这是一个最简单的block，没有输入参数，没有返回值，但是我们可以清晰的看到block的结构，可以得到结论：

1 block是个对象

2 block可以看成是一种函数

3 block的实现是个结构体


在教程一中说到，block可以截获变量，那么截获的变量是怎么处理的呢？ 下面一节将讨论这个问题














