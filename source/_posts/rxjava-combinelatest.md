title: 使用RxJava.combineLatest时犯的错误总结
date: 2017-01-14 23:00:00
categories: 技术
tags: [Android,RxJava]
description: 好好读文档啊少年
---

最近在做页面开发的时候遇到这么个问题：我在一个页面同时请求了3条API，通过CombineLatest进行组装，返回一个合成之后的结果用于UI展示。但是在实际使用过程中，我们发现，API正常的返回了数据，但是List的数据绑定发生了两次。经过排查，发现问题出来对于CombineLatest操作符的理解错误上：我错误的认为「CombineLatest会将所有传入的Observable通过FunN组合后返回一个结果」。实际是怎样的呢？我们来看看官方文档中的描述：

> Combines a list of source Observables by emitting an item that aggregates the latest values of each of the source Observables each time an item is received from any of the source Observables, where this aggregation is defined by a specified function.

以及官方文档里面的流图：

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-14-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-15%2000.15.53.png "CombineLatest 操作符")

所以不管是流图还是文档描述中，都强调了「每次接收都会根据前次source合并出一个结果」，所以其本质上是「把一系列的输入先后合并然后输出合并之后的结果」，而不是我最早理解的「将所有输入源的发射内容一起合并之后返回」。如果要实现我所需求的效果——将多个观测源的输出结果合并之后输出——还是应该老老实实用zip操作符。

OK，其实如果我在用之前有认真读过这些说明和流图之后应该就不会犯这个错误了，但是这个操作符接受多个Source的重载方法确实很具有迷惑性。

所以今后在使用一些新的方法、尝试一些新的内容的时候，第一点需要认真的读一下这些方法的文档，大多数成熟稳定技术的文档还是很健全的。第二点是，在使用过程中需要好好地检查输出结果是否符合预期——实际上在两个输入源的状况下，combineLatest的行为是符合我们预期的，但是在多个输入源的状况下，问题就凸显出来了。这个时候就需要我们能够准确的发现问题，然后解决这些问题。