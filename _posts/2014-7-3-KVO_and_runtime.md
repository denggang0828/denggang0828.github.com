---
layout: post
title: 'KVO and Runtime'
category: iOS
tags: KVO
---

####一、 Runtime

Objective-C的一个亮点在于它的动态性，它的实现依赖于强大的runtime运行时环境。

以C++为例，当想执行一个方法时，会通过调用函数的方式实现，编译器需要检查函数的名称，返回值及参数的个数和类型等，如果对象不存在这个方法，可能会发生崩溃等现象。但是在Objective-C中，执行方法是通过发送消息的方式实现的，消息发送过程是不关心对象能不能够响应这个方法，如果不能响应，会对消息进行相应的转发，即使转发到最后没有一个对象可以处理这个消息，应用程序也不会出现崩溃的现象。这个强大的转发功能就是通过runtime实现的。当然runtime的功能不仅仅如此，借助它我们可以更灵活的实现我们的代码，可以实现method swizzling，动态创建类和对象等。

####二、KVO

KVO的实现就是依赖于强大的runtime，它利用的是runtime可以动态创建类的特性。

使用时，把所关心的属性通过addObserver:forKeyPath:options:context:
进行注册，当这个属性发生变化时，我们就会收到相应的通知，在observeValueForKeyPath:ofObject:change:context:中进行处理即可，如果不需要继续观察，则执行removeObserver:forKeyPath:
移除观察者的操作。其实这就是一种典型的observer设计模式。使用起来就是这么简单，我们不需要为此编写大量的代码，只需要注册一下即可，因为Cocoa的底层框架已经对此做了支持。

先简叙下原理：

当执行addObserver:forKeyPath:options:context:添加观察者时，Cocoa会通过runtime为这个属性对象所属的类创建一个子类，然后将这个属性对象的isa指向新创建的子类上。这个过程就是通过runtime实现的。
    
那为什么要新创建一个子类呢？当然是和KVO有关。对于这个属性对象所属的类，默认情况下是不具备KVO功能的。在子类中重新实现对象属性的setter方法，实现发送通知的功能。这样，表面上看，是属性对象所在的类实现的KVO，但实际上是通过runtime新生成的子类完成的这一功能。

为了使这一过程更加清晰，参照https://mikeash.com/pyblog/friday-qa-2009-01-23.html中得实例，解释一下这个过程

{% highlight objc %}

#import <Foundation/Foundation.h>
#import <objc/runtime.h>

@interface TestClass : NSObject
{
    int x;
    int y;
    int z;
}

@property int x;
@property int y;
@property int z;

@end

@implementation TestClass

@synthesize x, y, z;

@end

static NSArray *ClassMethodNames(Class c)
{
    NSMutableArray *array = [NSMutableArray array];
    unsigned int methodCount = 0;
    Method *methodList = class_copyMethodList(c, &methodCount);
    unsigned int i;
    for(i = 0; i < methodCount; i++)
        [array addObject: NSStringFromSelector(method_getName(methodList[i]))];
    free(methodList);
    
    return array;
}

static void PrintDescription(NSString *name, id obj)
{
    NSString* string = [NSString stringWithFormat:@"%@:%@\n\tclass %@\n\tobjclass %@\n\timplementmethod %@\n",
                        name,
                        obj,
                        [obj class],
                        object_getClass(obj),
                        [ClassMethodNames(object_getClass(obj)) componentsJoinedByString:@" , "]];
    printf("%s", [string UTF8String]);
}


int main(int argc, char * argv[])
{
    @autoreleasepool {
        
        TestClass* textClass = [[TestClass alloc] init];
        
        PrintDescription(@"origin", textClass);
        
        [textClass addObserver:textClass forKeyPath:@"x" options:NSKeyValueObservingOptionNew context:nil];
        PrintDescription(@"add x", textClass);
        
        [textClass addObserver:textClass forKeyPath:@"y" options:NSKeyValueObservingOptionNew context:nil];
        PrintDescription(@"add y", textClass);
        
        [textClass addObserver:textClass forKeyPath:@"z" options:NSKeyValueObservingOptionNew context:nil];
        PrintDescription(@"add z", textClass);
        
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

{% endhighlight %}


输出为：

{% highlight objc %}

origin:<TestClass: 0x8d7c250>
	class TestClass
	objclass TestClass
	implementmethod z , x , setX: , y , setY: , setZ:
add x:<TestClass: 0x8d7c250>
	class TestClass
	objclass NSKVONotifying_TestClass
	implementmethod setX: , class , dealloc , _isKVOA
add y:<TestClass: 0x8d7c250>
	class TestClass
	objclass NSKVONotifying_TestClass
	implementmethod setY: , setX: , class , dealloc , _isKVOA
add z:<TestClass: 0x8d7c250>
	class TestClass
	objclass NSKVONotifying_TestClass
	implementmethod setZ: , setY: , setX: , class , dealloc , _isKVOA

{% endhighlight %}


可以看到，当不添加addObserver:时，属性对象所属的类和运行时指向的类都是TestClass；当添加addObserver时，运行时所指的类就变成子类NSKVONotifying\_TestClass，通过子类实现发送通知的功能。子类NSKVONotifying_TestClass中通过重写setter方法实现，当然，只重写添加的属性对象所对应的setter方法。


至于重写的过程，涉及到kvc的知识，之后会单独讲解。
