---
layout: post
title: "Non-pointer isa"
categories: ios
tags: isa
---
最新一期的iOS Dev Weekly中有篇文章比较有趣，[Non-pointer isa](http://www.sealiesoftware.com/blog/archive/2013/09/24/objc_explain_Non-pointer_isa.html?utm_source=iOS+Dev+Weekly&utm_campaign=iOS_Dev_Weekly_Issue_113&utm_medium=email)。

苹果发布了iPhone 5s 和iPhone 5c，采用了64位的处理器。作者在这篇文章中指出，在采用64位处理器的iOS的系统中，Objective-C类对象的isa将不再是一个指针。

