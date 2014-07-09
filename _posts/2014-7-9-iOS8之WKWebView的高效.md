---
layout: post
title: 'iOS8之WKWebView的高效'
category: iOS
tags: WKWebView
---

----

iOS8中引入了全新的WebKit框架，提供的WKWebView，自称可以达到Sarari的性能，远远优于之前的UIWebView。这篇文章会在解答三个疑问的过程中，让你清晰的了解这一改进。

###一、WKWebView比UIWebView性能优化了多少？

[Why iOS 8′s WKWebView is a Big Deal for Hybrid Development
](http://developer.telerik.com/featured/why-ios-8s-wkwebview-is-a-big-deal-for-hybrid-development/) 这篇文章中对Chrome for iOS和Safari的性能做了一个对比。Chrome for iOS采用的是UIWebView的方式载入网页，代表的是UIWebView的性能；由于WKWebView可以达到Sarari的性能，所以使用Sarari来进行等价比较。

可以清晰的看到比较结果：

JS性能提升了368.42%
整体性能提升了20%

所以，Sarifi的性能是远远由于UIWebView的，WKWebView的性能优势凸显出来，在iOS8中，应当使用WKWebView取代UIWebView。


###二、为什么UIWebView低效？

Sarifi 之所以高效，是因为它使用了强大的Nitro JavaScript engine，极大的提升了浏览网页的速度。但是，对于三方的App，苹果对其使用JavaScript engine做了限制，导致的结果就是无论三方App如何优化UIWebView，都达不到Sarifi的性能优势。

为什么iOS对三方App使用Nitro JavaScript engine做了限制呢？

有种比较搞笑的说法：这是苹果的战略，避免Web App取代Native App从而影响整个App Store的生态系统。

但是如果不用iOS的UIWebView，自己实现一套Nitro JavaScript engine可不可以呢？当然可以！那为什么不实现呢？这肯定是有更深层次的原因。

苹果的整个生态系统很封闭，原因是最大程度上保证系统的安全性。这一设计也不例外。

Nitro JavaScript engine之所以高效，是因为使用了JIT ( “Just-In-Time”)编译，而JIT需要获得在内存页中标志的权利。iOS处于安全性的考虑禁掉对Nitro JavaScript engine的使用（尽管很多OS允许使用,e.g. Max OS X, Windows, Android），因为获得内存页读写执行的权限意味着程序可以执行未签名的代码，这是存在安全风险的。


###三、WKWebView做了哪些改进？

iOS8中，WkWebView的高效是因为解除了对Nitro JavaScript engine的限制吗?

不是的。

iOS8中，进程间通信的API得到了极大的改进-XPC。带来的收益有：接入三方键盘，共享存储空间，滤镜，及高效的Webkit

iOS8之前，这些以插件形式运行在应用程序内部的功能都是不安全，可能是程序crash或存在安全隐患。随着XPC的增强，可以把这些功能放在一个独立的沙盒中，从而保证系统的高效和安全。



参考资料：

1 [http://nshipster.cn/ios8/](http://nshipster.cn/ios8/)

2 [iOS 8 WebKit changes finally allow all apps to have the same performance as Safari
](http://9to5mac.com/2014/06/03/ios-8-webkit-changes-finally-allow-all-apps-to-have-the-same-performance-as-safari/)

3 [Why iOS 8′s WKWebView is a Big Deal for Hybrid Development
](http://developer.telerik.com/featured/why-ios-8s-wkwebview-is-a-big-deal-for-hybrid-development/)

4 [Why the Nitro JavaScript Engine Isn’t Available to Apps Outside Mobile Safari in iOS 4.3](http://daringfireball.net/2011/03/nitro_ios_43)
4[iOS8,WebKit Performance,and XPC](http://daringfireball.net/linked/2014/06/09/ios-8-webkit)
