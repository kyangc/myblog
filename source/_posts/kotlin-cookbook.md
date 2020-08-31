---
title: Kotlin Cookbook 读书笔记
date: 2020-08-31 14:22:50
tags: [Kotlin]
description: 记录一下
categories: 技术
---

## 第二章 Kotlin 基础

- as? 可以尝试类型转换，若失败则返回空值
- @JvmOverloads 注解可以生成 java 侧的带默认参数的方法
- Int.toString(radix: Int) 可以将数字转换为2到36之间任意进制的字符串
- shl/shr/ushr/ushl 是 kotlin 的位运算
	- 可以通过无符号位移 ushr 来达成对于溢出的大数进行除2的用处

## 第三章 Kotlin中的面向对象编程

- data 类提供 toString, hashcode, equals, copy 函数, 解构函数等等，其中 copy 函数是浅拷贝。
- 幕后属性技术：by lazy 委托，可以比较优雅的实现实例非空属性的懒加载
- lateinit 于 var 配合使用，用于延迟初始化部分可变非空参数
- Nothing 类是个没有实例的类，一般用于完全抛出异常的函数，或者用于表征一个被声明为 null 且没有显式类型声明的变量
	- Nothing是所有类型的子类
	- Any 是所有类型的父类

## 第四章 函数式编程

- fold 和 reduce 的区别在于，fold 进行过程中的初始值可以设定，而 reduce 进行过程中的初始值就是集合的第一个数
- 可以通过 tailrec 关键字来进行尾递归优化

## 第五章 集合

- 可以通过 associate 函数来快速的由 list 生成 map
- 可以使用 ifEmpty 在集合为空的时候返回一个给定的集合
- number.coerceIn() 可以返回一个满足区间条件的数
- 可以通过 sortedWith 和 compareby 组合出一个多重的排序规则：比较A -> 比较B -> 比较C

## 第六章 序列

- 没有序列操作的末端操作（first toList 等），序列操作不会执行

## 第七章 作用域函数

- 使用 also 来在不打断工作流的情况下执行一段代码，返回原本的对象，通过 传入参数 it 引用到原有的对象
- apply 也是返回原本的对象，与 also 不同的点在于，它没有传入参数，而是将对象作为上下文传给 lambda
- let 返回 block 的返回值，可以认为是一种 map 行为
- run 可以单独使用也可以作为扩展函数，返回lambda的返回值
- with 同 apply，不过不是扩展函数，在 block 里以 context 使用上下文
- 总结一下：
	- 可以指定 lambda 中的 context ：apply，with
	- 通过 lambda 影响返回值：run，let
	- lambda 既影响不到返回值又无法指定 context：also

## 第八章 委托

- 可以通过 class C(a:A = A(), b:B=B()): A by a, B by b 这种简单的方式来将 AB 类组合并把各个方法委托给各自的实例
- 可以使用 Delegates.vetoable 代理对赋值行为进行监督
- crossinline 用于标识一个 inline 函数的 lambda 参数被作为一个参数传入另一个函数时，期望不允许其局部返回时使用

## 第十一章 其他

- 可以通过对 invoke 操作符重载，来将一个只包含单一方法的类，定义为一个可执行类
- 可以通过 measureTimeMillis/measureNanoTime 来便捷的计算执行时长
- 可以在方法名中加入反引号或者下划线，但是前提是这只允许发生在测试代码中
- 通过 @Throws(ExceptionClass::class) 注解标注一个可能抛出异常的方法
