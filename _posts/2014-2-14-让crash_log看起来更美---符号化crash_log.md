---
layout: post
title: '让crash log看起来更美---符号化crash log'
category: iOS
tags: Symbolication
---

分析崩溃日志是每个iOS开发者必备的技能，从崩溃日志中获取有用信息能帮助我们完善代码，让程序更稳健。对于如何分析crash log，网上已经有很多相关教程，推荐大家先读一下这篇文章[iOS应用崩溃日志揭秘](http://www.raywenderlich.com/zh-hans/30818/ios应用崩溃日志揭秘)，它对整个分析过程进行了详细描述。我的这篇文章是基于它，把其中符号化的过程单独拿出来讲一下，并做了一些小扩展和补充，希望能有助于大家的理解。

----

###一、获取crash log

####方法：

1 设备与iTunes同步，在电脑的对应路径上获取crash log。

2 通过XCode获得。

3 应用上架之后，从iTunes Connect上可以下载得到。

这三种获取方式都在[iOS应用崩溃日志揭秘](http://www.raywenderlich.com/zh-hans/30818/ios应用崩溃日志揭秘)一文中有详细介绍，不再详述。

4 另外，这里再介绍一种常见的方式。有些应用在崩溃后再启动的时候，会弹个对话框，说系统检测到上次使用时异常退出了，要不要把异常信息反馈。这种采集crash log的方式是通过对函数NSSetUncaughtExceptionHandler的扩展实现的。

NSSetUncaughtExceptionHandler可以用来进行异常处理，但功能非常有限，无法处理“内存访问错误”，“重复释放”等问题，因为这种错误抛出的是Signal，所以必须要专门做Signal处理。

使用时可以参考这里开源的代码：
[UncaughtExceptionHandler]( https://github.com/denggang0828/UncaughtExceptionHandler)

它的原理很简单，基本流程如下：

1 用NSSetUncaughtExceptionHandler()注册自定义异常处理Handler，并注册signal处理机制

在函数void InstallUncaughtExceptionHandler(void)中

2 程序崩溃

3 程序下次启动时如果发现异常文件，则进行自定义处理

使用更简单：

在didFinishLaunchingWithOptions函数中加入 InstallUncaughtExceptionHandler() 即可。

----

###二、获取dsym和app文件

如果产生crash log的App是在你自己的机器上编译的，那么直接把log拖到XCode->Organizer->Device Logs即可自动完成符号化的过程。否则，你需要获得对应的dsym和app文件才能完成这一过程。

为什么要获得dsym和app文件呢？

先看一下崩溃日志一条信息的格式：

8 Rage 0x000d4046  0xd2000+8262

每一列分别为帧编号，二进制库的名称，调用方法的地址，
最后分为两个子列，一个基本地址和一个偏移量。此处是0×83000 + 8740, 第一个数字指向文件，第二个数字指向文件中的代码行。

只看这些内存地址是没有意义的，我们需要将这些地址符号化Symbolication，转换成方法名和行数，才有助于我们定位错误。Symbolication的过程就需要获取dsym和app文件。

对于dsym文件，一句话描述就是：是个编译过程中的中转文件，内部为一个映射表，映射的内容为“内存地址”和“方法名”。

在这提一下快速正确获得dsym和app的方法
打开XCode，Window->Organizer，选择Archives，定位到当时提交的那一列，右键选择Show in finder，选择对应的xcarchive文件，右键显示包内容，看以看到两个文件夹，分别存放dsym文件和app文件。

----

###三、符号化过程

此时，已经拿到了crash log(log.crash)，dsym(projectName.app.dSYM)文件和app文件，首先要把他们放到一个目录下。执行如下命令：
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash log.crash projectName.app.dSYM > crashlog.crash

如果有如下报错：

Error: "DEVELOPER_DIR" is not defined at /usr/local/bin/symbolicatecrash line xx.

要先输入：

export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer

这样，正常情况下crashlog.crash就是已经符号化过的log了，可以根据它方便的定位导致程序崩溃的原因。

那异常情况呢？

首先要排查的是crash log，dsym文件和app文件是否是一个版本的

####检查方法：

1 crash log中有一项Incident Identifier。

2 检查app的版本号

dwarfdump --uuid YourApp.app/YourApp

3 检查dSYM的版本号

dwarfdump --uuid YourApp.app.dSYM

确保三者得到的udid是一致，才可能正确的进行符号化的过程。此处注意要区分armv7和armv7s分别对应的地址。

----

###四、使用atos

另外，ChenLong提供了一种方法，使用atos符号化指定的内存地址：

####步骤为：

1 otool -arch armv7s -l YourApp | grep -B 3 -A 8 -m 2 "__TEXT"

记下输出信息中的vmaddr的值：address1。

2 记下log信息中待符号化那一行第三列的地址值：address2。
记下log信息中待符号化那一行第四列加号前的地址值：address3。

3 计算address = address1 + address2 – address3。

4 atos -arch armv7s -o EssayZone address 输出即为符号化的信息。

####注意：

1 YourApp的获得方式：右键打开包YourApp.app.dSYM，一直到根目录可获得。

2 armv7s或者armv7的选取要根据crash log中得信息决定



关于这块的讨论可以参考:

[Matching up offsets in iOS crash dump to disassembled binary](http://stackoverflow.com/questions/10578155/matching-up-offsets-in-ios-crash-dump-to-disassembled-binary)

[Atos cannot get symbols from dSYM of archived application](http://stackoverflow.com/questions/7675863/atos-cannot-get-symbols-from-dsym-of-archived-application?lq=1)
