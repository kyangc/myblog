title: 你真的会使用 Scheduler.io() 吗？
date: 2017-01-16 10:00:00
categories: 技术
tags: [Android,RxJava]
description: 老司机还是翻车了
---

最近堆糖6.7.0版本正在准备上线，在堆糖最近的灰度版本中我们观测到了许多不正常的OOM——来自于各种方面的OOM都有，非常奇怪。有很多代码都是没有改动过的，但是这次灰度版本中却发现因为OOM的关系FC了。不过由于在6.6.0到6.7.0两个版本中间被我改动了近3W行代码，所以顿时浑身冷汗，担心是不是在某个未知的角落的改动造成了问题。

光靠想不起作用，打开Fabric，老老实实一条条崩溃记录的检查吧。第一遍看下来，没什么头绪，发现的唯一的特征是多数OOM发生在RxJava的调用过程中——但是依然有少量OOM和RxJava毫无关联。回想了一下，这个版本似乎并没有升级过RxJava的依赖版本，问题应该不是出在RxJava本身。

继续追崩溃日志，有一个异常点突然引起了我的注意——在这些所有的崩溃记录中，线程数都异常的高：

![](https://imgs.kyangc.com/2017-01-16-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-16%2021.26.57.png "超高的线程数")

Android/Java虚拟机的线程资源是有限的，在这么高的线程数之下，这个程序基本也活不了太久了……[这篇文章](http://jzhihui.iteye.com/blog/1271122 "这篇文章")大致讲述了Java虚拟机的线程资源与堆栈大小之间的关系，有兴趣的可以看一下。

OK，OOM的根源基本定位到了——超高的线程分配数是罪魁祸首——那么这些超高的线程数是怎么来的呢？我们继续研究报错堆栈。很快，在报错栈的线程列表中，我们发现了大量名为「RxIoScheduler-xx」的线程：

![](https://imgs.kyangc.com/2017-01-16-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-16%2021.41.14.png "超多的 RxIoScheduler-xx 线程")

看到这里，熟悉RxJava Scheduler的使用的同学一定联想到了线程调度符Schedulers.io()，在处理异步的IO动作时，我们正是通过这个将工作调度到IO线程中，在RxJava中的具体实现则是通过一个类似CachedThreadPoolExecutor的线程池来承载业务、分配线程，这个线程池的线程数会随需求的增减动态改变。

到这里不难看出，疯狂增长的线程数肯定与这个IO调度有关，但是为什么会出现这种状况？IO操作符的使用难道哪里出了问题？

StackOverflow一下，果然，有个哥们和我一样遇到了同样的问题，而他的解决方式则是：**在完成异步操作之后，显式的调用`subscriber.onComplete()`来终结这次Subscription**。经过实践，确实通过在所有耗时操作结束之后调用onComplete方法，能够有效地释放线程资源，线程数也恢复了正常。

知道怎么做是不够的，为了搞清楚这究竟是怎么一回事，我们接下来从源码部分简单的看看Scheduler究竟做了些什么：

首先我们从入口方法 subscribeOn(Scheduler scheduler) 开始：

```java
public final Observable<T> subscribeOn(Scheduler scheduler) {
    if (this instanceof ScalarSynchronousObservable) {
        return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
    }
    return create(new OperatorSubscribeOn<T>(this, scheduler));
}
```

scalarScheduleOn 部分的我们忽略，这个方法实际上是利用Scheduler对象以及原本的Observable对象，重新生成了一个Observable对象，下面看看在构造OperatorSubscribeOn的时候做了些什么事：

```java
public final class OperatorSubscribeOn<T> implements OnSubscribe<T>
```

OperatorSubscribeOn 实现了 OnSubscribe 接口，实际上就是另一层最初的OnSubscribe的封装，我们主要看看对应的call(Subscriber subscriber)方法中做了些什么：

```java
public void call(final Subscriber<? super T> subscriber) {
	//创建一个worker对象
    final Scheduler.Worker inner = scheduler.createWorker();
	//注册这个worker对象
    subscriber.add(inner);
	//worker对象将本操作符之前的所有操作打包，放入其所在的线程池(io)中等待执行
    inner.schedule(new Action0() {
        @Override
        public void call() {
            final Thread t = Thread.currentThread();
			//将原本的subscriber重新组装 - 在这次封装中包含了对于worker对象的取消订阅操作
            Subscriber<T> s = new Subscriber<T>(subscriber) {
                @Override
                public void onNext(T t) {
					//注意！这里只调用了onNext，并没有取消对于源的订阅！这也是为什么只调用onNext不调用onComplete或onError不会取消订阅者对于发送者的订阅的原因
                    subscriber.onNext(t);
                }
	
                @Override
                public void onError(Throwable e) {
                    try {
                        subscriber.onError(e);
                    } finally {
						//取消了订阅
                        inner.unsubscribe();
                    }
                }
	
                @Override
                public void onCompleted() {
                    try {
                        subscriber.onCompleted();
                    } finally {
						//取消了订阅
                        inner.unsubscribe();
                    }
                }
	
                @Override
                public void setProducer(final Producer p) {
                    subscriber.setProducer(new Producer() {
                        @Override
                        public void request(final long n) {
                            if (t == Thread.currentThread()) {
                                p.request(n);
                            } else {
                                inner.schedule(new Action0() {
                                    @Override
                                    public void call() {
                                        p.request(n);
                                    }
                                });
                            }
                        }
                    });
                }
            };
	
			//将重新组装之后的subscriber重新用源observable订阅起来
            source.unsafeSubscribe(s);
        }
    });
}
```

在上面的代码我们看到，在新生成的OnSubscribe对象中，当call方法被调用时，Scheduler对象会生成一个worker对象，作用是将该操作符之前的所有动作一起打包放到该worker所在的线程池中执行任务，并且worker对象也实现了subscription接口，可以用于取消本次任务订阅。

可以看到，在向worker所在的线程池发出任务的时候，实际上是重新封装了一个Subscriber，并让该Subscriber重新订阅发射源，在onNext方法中并没有将该worker对象取消订阅，只在onComplete方法和onError方法中调用了worker对象的取消订阅相关的代码——这也是为什么在使用该操作符时如果不手动处理订阅或显式调用onComplete就无法完成自动取消订阅的原因。

其实在worker对象的生成、io线程的底层CachedThreadPool实现以及worker对象的取消订阅这些方法也有很多内容，不过不属于本篇内容，这里就不做过多叙述，有兴趣的读者（我知道这文章没啥读者- -）可以自己读读源码。讲道理，RxJava的源码算是我读过的代码里面相当恶心且绕且难懂的代码了……要读下去真的需要一些耐心……

言归正传，最终我们得出了这么一个结论：

> 如果不是直接使用类似于`just`、`from`、`zip`等等已经封装好的操作符，而是直接新建`onSubscribe`对象，自己处理`subscriber`的`onNext`、`onError`等操作的话，最好是能做到在正确返回数据时调用`onNext`，在错误时调用`onError`，并且保证在所有动作处理结束之后能够调用`onComplete`动作结尾。

