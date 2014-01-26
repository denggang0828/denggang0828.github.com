---
layout: post
title: 'CoreData与sqlite的一些比较'
category: iOS
tags: CoreData
---

----
最近一段时间的开发接触到了CoreData，使用它作为本地数据持久化的方案，而在此之前曾经也使用过Sqlite作为本地数据持久化的方案。对于这两种方案的差异，很多新手在刚接触时可能会有些迷惑，所以总结一下自己这段时间对它们的理解，比较一下两种方案的异同。

----
####一、	概念上的不同

sql负责从磁盘的数据库文件中读取/写入数据，至于程序对读取出来的数据做了哪些操作，它不会去关心这些事情，它只关心数据能否正确的存取。

相反，CoreData把90%的精力放在对数据的处理上，将数据模型封装成对象处理。另外10%的工作是才是对数据的存取上。

进一步讲，sql的主要功能是对本地数据进行持久化，而这只是CoreData一小部分可选的功能。在使用CoreData的过程中，我们完全可以不使用它的持久化功能，值得我们关注的是它对数据模型的处理功能上。
[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html#//apple_ref/doc/uid/TP30001200-SW1)中对它的描述为：

“
The Core Data framework provides generalized and automated solutions to common tasks associated with object life-cycle and object graph management, including persistence
”

可以看到，CoreData的重点在于数据对象的图形化管理，更能体现面向对象的编程思想，而sql体现的是面向过程的编程思想。

----
####二、	效率上的比较

sql直接操作数据库，所以读写效率很高，而且对数据的增删改查可以直接在数据库中生效。对于coredata，对数据的操作是在内存中进行的，需要先将数据从数据库中读取到内存才可以操作，不能直接操作数据库。

1、写入性能。两者是没有区别的，因为使用CoreData的存储功能时可以选择采用sqlite格式，真正存储时也是在执行sql语句。使用CoreData存储数据时要注意的一个问题是，批量写入数据的时候，要避免每写入一条数据就执行一次save函数，应当把所有需要写入的数据存储到内存后，执行一次save即可。

2、查询性能。 sqlite有丰富的sql语句，查询数据时可以灵活的使用所以效率较高。CoreData由于抽象的层次比较高相比会弱一些，在fetch数据时，可用的predicate也没有sql语句丰富，但是，可以把fetch的数据读入内存中做一些filter处理，借助内存的高效计算，使效率在一定程度上得到改善。

----
####三、一些tips

1 CoreData不是数据库，没有主键，需要自己根据业务需求添加。

2 在生成大量数据时，注意，是“生成大量数据”而不是“写入大量数据”，CoreData要高效的多。因为coredata是在内存中生成数据的。

3 使用CoreData读取数据时，应当尽量使用Batch，会极大提高读取速度。

4 对于一些app，由于版本升级可能会对数据库的表结构做一些改变，如果不加处理的话，程序会产生崩溃。对于这部分的处理，CoreData会自动完成，还是很方便的。

5 当然，对于可移植性来说sql会好一些，毕竟core data只是cocoa的一个框架。 

----
####四、对多线程的处理

core data是线程不安全的。如果想在多线程中访问Core Data的话，一个方法是每个线程创建一个NSManagedObjectContext，每个NSManagedObjectContext的对象可以使用同一个NSPersistentStoreCoordinator实例。这个实例可以安全的顺序访问持久化存储文件。其他类似加锁的方式也是适用的。同样，sqlite也是线程不安全，但是一些对它的封装或其他数据库是可以支持多线程。

对个人来说，自从接触CoreData之后，就喜欢上这个框架了。把数据真正的作为对象处理，充分体现了OO的思想。而且在数据存储时，直接对对象处理即可，避免了再写 对象-》sql字段 的转换，而且也避免了写大量的sql语句，极大的提高了开发效率。


一些参考资料：

1 [Core Data over SQLite Performance Tests ](http://i.ndigo.com.br/2010/07/core-data-over-sqlite-performance-tests-part-1)

2 [Use CoreData or SQLite on iPhone?](http://stackoverflow.com/questions/1318467/use-coredata-or-sqlite-on-iphone)

3 [differences between coredata and a database](http://www.cocoawithlove.com/2010/02/differences-between-core-data-and.html)

