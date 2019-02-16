title: 几行伪代码解决大多数 View 触摸事件传递的问题
date: 2017-01-13 10:00:00
categories: 技术
tags: [Android,View]
description: 先抓住主干，再研究细节
---
看书的时候总结了一下View触摸事件的传递逻辑。这里用伪代码写写看。

Activity / Window / ViewGroup 处理触摸事件的逻辑：

```java
if(!dispatchTouchEvent(e)){
	onTouchEvent();
}
```

View 中 dispatchTouchEvent 的逻辑：

```java
public boolean dispatchTouchEvent(MotionEvent e){
	return onTouch(e) || onTouchEvent(e);
}
```

ViewGroup 中 dispatchTouchEvent 的逻辑

```java
public boolean dispatchTouchEvent(MotionEvent e){
	if(onInterceptTouchEvent(e)){
		return onTouch(e) || onTouchEvent(e);
	} else {
		return child.dispatchTouchEvent(e);
	}
}
```

当然里面还有各种问题，但是把握好主干总是会让我们在学习这部分知识的时候更加有针对性。