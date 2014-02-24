---
layout: post
title: 'viewDidUnload弃用的真相'
category: iOS
tags: viewDidUnload
---

今天讨论一个古老的话题，iOS6弃用了viewDidUnload，当收到内存警告时，不会回调这个方法，而是进入到didReceiveMemoryWarning。所以我的措施是把之前在viewDidUnload里执行的release语句全部copy到didReceiveMemoryWarning里，然后置为nil。确保在接受到内存警告的信号后可以释放不在屏幕上的view的内存，但是当看到这篇文章[View Controller Lifecycle in iOS6](http://stablekernel.com/blog/view-controller-lifecycle-in-ios6/)及[再见，viewDidUnload方法](http://blog.devtang.com/blog/2013/05/18/goodbye-viewdidunload/)，才发现自己完全理解错了，下面将一步一步的描述我的最新理解。

####一、为什么不用viewDidUnload了？

咱们来设想一种场景，一个注册信息的界面，可能大概有5、6个输入框，用户费劲力气填完这些输入框的信息，突然发现有个注册信息帮助界面的链接，用户好奇的点了过去，进入到这个帮助界面，可是这时不幸的事情发生了，系统发出内存警告的信号了，这时我们的app把当前不在屏幕上的view的内存release，当用户点击返回按钮想返回到注册信息界面时，由于这时候的注册界面已经不是刚刚的注册界面了，所以用户会发现刚才努力填写的信息都没了，这种用户体验很糟糕吧。
   
当然啦，我们可以在release这些view的时候，在VC中先把这些信息存一份，等到重新构建这个界面的时候再把这个值赋上去，但是这个还得增加部分代码控制这个逻辑，无形中增大了工作量。

所以这个问题需要优化，在内存释放和用户体验之间找一个平衡点。要达到这个目的，先看一下UIView各部分所占用的内存情况。

####二、UIView的内存
   
UIView能响应事件，是因为它为UIResponder的子类。
UIView能绘制在屏幕上，是因为它有一个CALayer的成员变量。

CALayer是个容器，盛的是bitmap图像。当view执行drawRect方法时，layer才会创建bitmap图像。

再看各部分所占用的内存：
	
UIView: 96 bytes

CALayer: 48 bytes

layer's bitmap: 由layer的大小决定，一个retain iPad的全屏UIView的bitmap会占到12M

所以相对于bitmap，view和layer所占用的内存可以忽略了。基于这个，iOS6中，当收到内存警告时，系统只是把bitmap的内存释放掉，保留了CALayer和UIView的内存。当需要重新显示bitmap时，可以直接在drawRect中绘制出来。这样，我们既释放了绝大部分内存，又没有丢失view的信息，更关键的是这些都是系统帮我们做的，省时省力省心，一举多得。  
   
####三、进一步的优化
drawRect是耗CPU的（参考[正确理解drawRect](http://denggang0828.github.io/正确理解drawRect/)），bitmap的内存释放后如果还需要显示，需要调用drawRect重新绘制屏幕，苹果对这一块也进行的优化，可以减小drawRect的调用。   

如果一块内存被分配，则会被标为in-use，当被释放时，会标为not in use。然而，对于bitmap所占用的内存，是被优化过的，标为volatile，这不是真正意义上的释放，如果这块内存没有被再次占用的话，当这块内存的bitmap需要显示时，可以直接拿过来使用，这样就避免再次执行drawRect。

综上所述，苹果抛弃viewDidUnload还是有很大意义的，当时太想当然就没去思考为什么苹果这么做，每一个改变都有其前因后果，引以为戒，下次注意了。
