title: Android启动模式归纳 
date: 2015-03-17 20:00:00
categories: 技术
tags: [Android] 
description: 启动模式是个坑，谁跳进去谁知道
---

>启动模式对于Android开发是很重要的，各种启动模式之间的区别必须要掌握，这里简单的归纳一下。

<!--more-->

*	### 启动模式
	*	**Standard**
	
		标准模式，不会新建Task，会重复建立Activity。
		
		`A B C D  -New B->  A B C D B'`
		
	*	**SingleTop**
	
		单Task，单栈顶元素模式。
		
		当新建Activity的时候，如果该Activity在栈顶，那么将不会新建activity，直接复用原有的activity。
		
		`A B C D  -New D->  A B C D`
		
		如果不在栈顶，那么会重新建立这个Activity。
		
		`A B C D  -New B->  A B C D B`
		
	*	**SingleTask**
	
		在Task中只会有一个Activity的实例。
		
		*	如果Task中没有该Activity的实例，那么在栈顶加入新建的Activity
		
		`A B C D  -New E->  A B C D E`
		
		*	如果Task中有该Activity的实例，并且在栈顶，那么和SingleTop一样的策略
		
		`A B C D  -New D->  A B C D`
		
		*	如果Task中有该Activity的实例，但是不在栈顶，那么这个时候新建该Activity，会将已有的该Activity实例之前的Activity全部出栈。
		
		`A B C D  -New B->  A B`
		
	*	**SingleInstance**
	
		在该模式下，不论从哪个Task加载Activity，只会创建一个Activity实例，并且会使用一个全新的Task栈来装载。
		
		*	要创建的Activity实例不存在，这个时候会创建一个新的Task栈，并将Activity实例化后放入该栈。
		
			`A B C D  -New E->  A B C D (New Task) E`
			
		*	要创建的Activity实例存在，这时候系统会将该Activity所在的Task转到前台。
		
			注意此种模式下加载的Activity总是处于栈顶的，使用该模式创建的Task只包含该Activity
		
	
*	### Intent Flag
	*	Brought to front
		
		如果该Activity的实例已经存在了但是不在栈顶，那么将会把这个Activity的实例转移到栈顶，如果没有的话当然就是在该Task的栈顶加入该Activity的实例了：
		
		**注意，这里是A通过该Flag启动B，然后D通过任意方式启动B，会实现这种效果。**
		
		`A-Flag->B C D  -New B->  A C D B`
		
	*	Clear top
	
		同SingleTask的启动模式。				
	
	*	New task
	
		同Standard启动模式。
	
	*	No animation
	
		启动时不使用过渡动画。
		
	*	No history
	
		从该Activity的实例跳转之后不会保留该实例在Task栈中：
		
		**注意，这里是C通过该Flag启动D，D以任意方式启动E得到的结果。**
		
		`A B-Flag->C D  -New E->  A B C E`
	
	*	Reorder to front
	
		如果Task栈中已有Activity实例，那么通过该Flag启动的话，会直接将其转入栈顶。
		
		注意这里与Brought to front的区别，前一个Flag是在第一次启动那个任务的时候设置的，这个Flag不需要第一次启动的时候设置，在下一次用得到的时候再设置，将已有的Activity转到前台。
		
		`A B C D  -Flag New B->  A C D B`
		
	*	Single top	
	
		相当于SingleTop模式。