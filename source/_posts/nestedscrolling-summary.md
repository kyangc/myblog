title: Android Nested Scrolling 学习笔记
date: 2016-05-09 22:00:00
categories: 技术
tags: [Android] 
description: 源码之前没有秘密
---

我们知道，在Android中，触摸事件的处理流程是通过`dispatchTouchEvent`分发触摸事件，通过自下而上的拦截事件(`onInterceptTouchEvent`)以及自上而下的处理事件(`onTouchEvent`/`onTouch`)完成事件的分发，如果想要完成子控件与父控件同时对某一事件的处理的话，需要自己额外实现较多代码，非常繁琐而且容易出错，而`NestedScroll`接口的出现正是为了应对这种情况。

该接口已经在Android官方控件中广泛的得到了使用，包括在*Android Design*包中引入的`CoordinateLayout`、`FabIcon`以及之前的`RecyclerView`、`SwipereFreshLayout`等等常用控件现在均已实现了该接口，都可以作为嵌套滑动的子容器或父容器使用。然而令人非常遗憾的是，`ViewPager`作为一个使用频次非常高、使用场景中非常容易出现嵌套滑动场景的控件，官方并没有提供任何相关的支持，甚至在`ViewPager`的源码文档开头依然宣称该控件依然停留在一个非常早期的开发阶段（This class is currently under early design and development），令人唏嘘不已。 

`NestedScroll`系列接口包括以下四个部分：

	NestedScrollingChild//实现嵌套滑动的子控件需要实现的接口
	NestedScrollingParent//实现嵌套滑动的父容器需要实现的接口
	NestedScrollingChildHelper//在需要嵌套滑动的子控件中一个用于辅助进行嵌套滑动的Helper
	NestedScrollingParentHelper//在需要嵌套滑动的父容器中一个用于辅助进行嵌套滑动的Helper
	
为了实现父容器和子控件之间的嵌套滑动，首先我们需要选定父子容器，明确是哪个父容器需要配合子容器进行嵌套滑动，这里需要注意的是，嵌套滑动只能发生在父容器与子控件之间，并且滑动的发起者是子控件，父容器是被动的接受子控件发出的嵌套滑动请求并作出响应的一方。

当选定好我们需要进行嵌套滑动的控件双方时，我们首先需要在子控件中实现`NestedScrollingChild`接口，然后在子控件的构造函数中生成一个`NestedScrollingChildHelper`对象，并将接口中所有方法委托给helper对象。同样的，我们类似的处理父容器——实现接口、委托方法，然后准备工作就完成了。

可以看到，我们仅仅是把接口代理给了一个helper，仅仅依靠这个是怎么完成嵌套滑动呢？这些方法又应该在哪里调用呢？又有什么意义呢？我们继续往下看。

我们首先分析下主要的嵌套滑动发起者`NestedScrollingChild`中的方法，由于所有方法均被代理给了helper类，因此我们直接看helper中的对应方法就好了：

1. `setNestedScrollingEnabled`，该方法开启/关闭嵌套滑动
		
		public void setNestedScrollingEnabled(boolean enabled) {
		    if (mIsNestedScrollingEnabled) {
				//如果正在嵌套滑动，那么通知父容器停止嵌套滑动
		        ViewCompat.stopNestedScroll(mView);
		    }
		    mIsNestedScrollingEnabled = enabled;
		}
	
2. `isNestedScrollingEnabled`，该方法用于检查是不是能够嵌套滑动

		public boolean isNestedScrollingEnabled() {
		    return mIsNestedScrollingEnabled;
		}
	

3. `hasNestedScrollingParent`，该方法用于检查是否关联有父容器用来接收嵌套滑动过程事件

		public boolean hasNestedScrollingParent() {
		    return mNestedScrollingParent != null;
		}
	
4. `startNestedScroll`，该方法用于开启一个嵌套滑动的过程，返回true表示可以开启嵌套滑动，返回false表示无法开启嵌套滑动。

		public boolean startNestedScroll(int axes) {
		    if (hasNestedScrollingParent()) {
		        //如果已经有了一个嵌套滑动的父容器对象，那么直接返回true
		        return true;
		    }
			//首先检查嵌套滑动是不是开启了
		    if (isNestedScrollingEnabled()) {
		        ViewParent p = mView.getParent();
		        View child = mView;
				//遍历从该子控件开始往上的所有父容器，尝试去调用这些父容器的onStartNestedScroll方法，
				//如果成功，那么将该父容器作为嵌套滑动的对象，并调用父容器的onNestedScrollAccepted方法。
		        while (p != null) {
		            if (ViewParentCompat.onStartNestedScroll(p, child, mView, axes)) {
		                mNestedScrollingParent = p;
		                ViewParentCompat.onNestedScrollAccepted(p, child, mView, axes);
		                return true;
		            }
					//如果父容器无法开始嵌套滑动，那么将子控件和父容器都向上遍历一级，继续这个过程，直到父容器为空。
		            if (p instanceof View) {
		                child = (View) p;
		            }
		            p = p.getParent();
		        }
		    }
		    return false;
		}
		
我们可以看到，在这个过程中，helper类会不断的从子控件开始向上遍历可以作为嵌套滑动对象的父容器，直到找到第一个可以作为嵌套滑动对象的父容器返回。这个机制也保证了我们不用强制子控件的直接父容器必须实现该接口，只要保证目标父容器实现了这个接口即可。

5. `dispatchNestedPreScroll`，该方法用于在子控件消费滑动事件之前去向父容器分发滑动事件，并允许父容器预先消费一部分滑动距离，该被消费的距离会通过数组`int[] consumed`来传递，并且如果子控件的绝对位置在父容器消耗距离的过程中发生了改变，那么这个改变的值会通过数组`int[] offsetInWindow`回传过来:

		public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow) {
			//只有在可以嵌套滑动的时候才分发事件
		    if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
				//只要滑动的距离在xy轴上任意不为0，那么开始分发事件
		        if (dx != 0 || dy != 0) {
					//初始化起始位置
		            int startX = 0;
		            int startY = 0;
					//初始化子控件在屏幕上的相对位置
		            if (offsetInWindow != null) {
		                mView.getLocationInWindow(offsetInWindow);
		                startX = offsetInWindow[0];
		                startY = offsetInWindow[1];
		            }
					//初始化consumed数组
		            if (consumed == null) {
		                if (mTempNestedScrollConsumed == null) {
		                    mTempNestedScrollConsumed = new int[2];
		                }
		                consumed = mTempNestedScrollConsumed;
		            }
		            consumed[0] = 0;
		            consumed[1] = 0;
		            
					//调用父容器中的onNestedPreScroll，让父容器预先滑动            
					ViewParentCompat.onNestedPreScroll(mNestedScrollingParent, mView, dx, dy, consumed);
					
					//重新计算子控件在屏幕上的位置，通过offsetInWindow返回给子控件
		            if (offsetInWindow != null) {
		                mView.getLocationInWindow(offsetInWindow);
		                offsetInWindow[0] -= startX;
		                offsetInWindow[1] -= startY;
		            }
					//如果在x轴或y轴上父容器发生了消耗，那么这个方法的返回值为true，否则为false，方便子控件跳过处理
		            return consumed[0] != 0 || consumed[1] != 0;
		        } else if (offsetInWindow != null) {
		            offsetInWindow[0] = 0;
		            offsetInWindow[1] = 0;
		        }
		    }
		    return false;
		}
6. `dispatchNestedScroll`，该方法在子控件消耗滑动距离之后调用，通知父容器将剩下的滑动距离消耗掉。

		public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
		        int dxUnconsumed, int dyUnconsumed, int[] offsetInWindow) {
			//只有在可以处理的时候才处理
		    if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
				//只有在发生了距离消耗或还有距离没有消耗完时才进行下一步
		        if (dxConsumed != 0 || dyConsumed != 0 || dxUnconsumed != 0 || dyUnconsumed != 0) {
		            int startX = 0;
		            int startY = 0;
		            if (offsetInWindow != null) {
		                mView.getLocationInWindow(offsetInWindow);
		                startX = offsetInWindow[0];
		                startY = offsetInWindow[1];
		            }
		
					  //注意这里的方法没有consumed数组返回了，只会更新offsetInWindow数组            
					  ViewParentCompat.onNestedScroll(mNestedScrollingParent, mView, 
					            dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed);
		
		            if (offsetInWindow != null) {
		                mView.getLocationInWindow(offsetInWindow);
		                offsetInWindow[0] -= startX;
		                offsetInWindow[1] -= startY;
		            }
		            return true;
		        } else if (offsetInWindow != null) {
		            // No motion, no dispatch. Keep offsetInWindow up to date.
		            offsetInWindow[0] = 0;
		            offsetInWindow[1] = 0;
		        }
		    }
		    return false;
		}

该接口中还有一些fling相关的方法，都是大同小异就不再仔细看了。事实上以上五个方法基本就完成了和嵌套滑动相关的在子控件上的全部操作，我们可以看到，helper因为是通过该view注入进构造方法新建出来的，所以helper中是持有view的引用的，因此可以遍历该view的视图树，并以此找到目标的嵌套滑动的父容器对象建立联系。同时，该helper也帮助使用者分发滑动时间、返回滑动消耗结果，帮用户省去了很多模板代码。

至于parent，由于嵌套滑动的parent通常是作为「回调」的接收方，其接口多为用于完成实际的控件滑动，所以基本都需要用户自己手动去完成实现，需要代理给parent helper的方法很少，这里就略过不讲了。

至此我们不难得出NestedScroll系列接口的正确用法：

在子控件开始滑动时通过`startNestedScroll`通知父容器配合，然后在子容器捕获到手势滑动时调用`dispatchNestedPreScroll`让父容器有机会在子容器消耗掉滑动距离之前先滑动一段距离，然后子容器完成自己的滑动，最后通过`dispatchNestedScroll`通知父容器自己完成了滑动，让父容器有机会再根据子容器的滑动做出一些滑动处理，所有的子容器方法，都会经过helper类转接，通过遍历视图树找到最近的一个嵌套滑动父容器，并尝试调用相应的`on**`回调方法完成父容器的响应。

可以看到整个传递触摸事件的思路是与原有的触摸事件分发机制是完全不一样的，这也为我们今后自己做相关的开发提供了思路和启发。
