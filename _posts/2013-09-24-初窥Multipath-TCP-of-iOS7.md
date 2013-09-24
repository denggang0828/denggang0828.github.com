---
layout: post
title: "初窥Multipath TCP of iOS7"
categories: ios
tags: Multipath_TCP
---

###一、什么是 Multipath TCP

更新iOS7后，很多小伙伴们都反映流量耗费的特别快。Olivier Bonaventure的这篇文章中，[Apple seems to also believe in Multipath TCP](http://perso.uclouvain.be/olivier.bonaventure/blog/html/2013/09/18/mptcp.html) 提到iOS7用到了Multipath TCP这一技术。他通过截取网络数据包发现使用Siri的过程中,系统建立Multipath TCP的连接。

[揭开 iOS 7 之 Multipath TCP 的面纱](http://imtx.me/archives/1852.html)国内的这篇文章中对Multipath TCP进行了一个大概的解释。

iOS 7是首个正式采用 Multipath TCP技术的商用操作系统。简而言之，这种技术就是用户在建立连接的时候是通过多点建立的，系统会根据各个点连接的优劣情况动态的去选择，只有所有点都连接不通的情况下才会断开网络链接。对用户而言，优点是可以有更好的网络体验；缺点是流量的耗费啊（由于多点连接都需要备份，所有消耗流量较大）。

----
###二、可能带来的改变

1 视频类的应用的改进。如果在3G的网络环境下播放视频，苹果官方文档严格要求码率不能超过64kbps，为的是避免耗费用户的大量数据流量。所以在开发这类应用时，会时刻监测网络环境的变化从而限制视频的播放与否。显然，这种以牺牲用户体验来换取流量节省的理念和Multipath TCP的设计初衷是相违背的，所以，也许苹果官方会逐渐放宽64kbps这一限制。当然，这需要国内广大运营商能提供非常优惠的流量套餐包支持的。


2 语音类应用的爆发。苹果是首先在Siri上应用这一技术的。因为Siri的使用包括语音包传输、语音识别、语义识别及实时TTS等过程，极大的依赖于网络环境。提高传输速度可以带来更好的用户体验，从而培育整个语音的市场，对于国内的讯飞、云之声等厂家来说，是个很好的机会。


虽然说Multipath TCP这一技术已经研究了5年，但是这是首次应用在商用系统下，和现有市场环境肯定需要一个磨合的阶段，但是这种趋势是移动互联网发展的核心方向，希望它能给国内的各语音助手类产品起一个推动的作用，引发语音助手类产品的爆发。特别是我参与的第二个iOS平台的产品---搜狗语音助手。O(∩_∩)O哈哈~




