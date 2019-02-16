title: Android IPC 学习笔记
date: 2016-05-02 10:00:00
categories: 技术
tags: [Android,IPC]

## description: 五一劳动节就看看书吧

> IPC: Inter-Process Communication

不同系统下的不同实现方式：

-   Windows: 剪贴板/管道/油槽
-   Linux: 命名管道/共享内容/信号量
-   Android: Binder/Socket

# Android中的多进程模式

使用场景：保活/推送/扩大内存空间/ContentProvider

## 如何开启？

-   在Manifest中设定四大组件的`android:process`属性
    		_ `:$remote_name`私有进程
    		_ `$package_name.$remote_name`共有进程，可以通过ShareUID的方式与其他应用共享进程
    			\* 需要两个应用拥有相同的ShareUID以及应用签名一致
-   通过JNI的方式在Native层去fork一个新的进程

## 多进程模式下的运行机制

Android对于每个进程都分配一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，通常来讲，使用多进程会造成如下几方面问题：

-   静态成员和单例模式失效——在不同的虚拟机中访问同一个类对象会产生多份副本。
-   线程同步机制完全失效
-   SharedPreference可靠性下降——SP底层是XML实现，并发读写都有可能出现问题。
-   Application会多次创建——新组建运行在新的进程中时，由于系统在创建新进程的时候会同时分配独立的虚拟机，相当于也是将应用重新启动了一次。

# IPC基础概念简介

## Serializable接口

Java提供的序列化方法，通过`ObjectInputStream`和`ObjectOutputStream`来对对象进行序列化及反序列化。

`Serializable`接口是一个空接口，只需要声明一个`serialVersionUID`即可，事实上这个字段不声明也是能够完成序列化/反序列化的。这个字段的作用在于_标识类的版本变化_，在序列化的过程中，会将这个值写入序列化的结果中，在反序列化时，会将写入值与反序列化对象的当前`serialVersionUID`值进行对比，如果相同那么能够成功反序列化，如果不一致则抛出异常。
如果该值不经手工指定，那么将按照该类的当前_成员变量_的构成计算Hash作为`serialVersionUID`值，在这种情况下，如果序列化/反序列化中间该类成员发生了改变，将导致无法完成序列化。因此为了保证序列化的成功率，最好手动指定`serialVersionUID`。

## Parcelable接口

Android系统中特有的序列化接口。需要实现的接口包括：

-   一个`CREATOR`常量作为反序列化方法，在该方法中定义通过序列化数据生成对象的构造方法。
-   一个`describeContents`内容描述符，通常情况下都为0，当且仅当当前内容含有文件描述符返回1
-   一个`writeToParcel`方法作为序列化方法

系统已经定义好了许多可以直接序列化的方法。如`Bitmap`，`Bundle`以及`Intnet`等。

> 那么，如何选择使用何种序列化方法？
>
> -   `Serializable`使用简单方便，但是需要大量的IO操作，开销较大，适合在进行数据本地化或数据序列化之后进行网络传输时使用。
> -   `Parcelable`定义较为复杂，但性能优秀，效率较高，是Android推荐的序列化方式，主要使用在内存序列化上。

## Binder

Binder既可以理解为Android中的一个实现了`IBinder`接口的类，也可以理解为一种虚拟的物理设备，他的驱动是`/dev/binder`。Binder是连接`ServiceManager`连接各个`ManagerService`之间的桥梁。最常用到的地方在Service的使用中，在`bindService`的时候会返回一个包含服务端业务调用的Binder对象，通过Binder对象来完成客户端对于服务端业务的调用。

不同进程内的Binder很简单，稍微复杂一些的Binder通常使用`AIDL`或者`Messenger`完成，而`Messenger`的底层也是由`AIDL`实现的，因此了解`AIDL`的使用方法、原理实际上对于理解Binder的上层原理是有帮助的。

![](https://ww1.sinaimg.cn/mw690/825558b1gw1f3hfkrwaavj20m60dnwey.jpg "Binder扮演的角色")

定义`AIDL`文件并不难，没有太多额外的语法需要注意，只需要按照自己的业务需求，像通常定义接口一样定义`AIDL`接口即可，注意所有自有的类都必须显式声明即可。在AS中的`AIDL`文件会被统一收拢到同一个目录下，但是这里有一个不知道是bug还是怎么回事的问题……当你已经定义好一个自定义类之后，IDE不允许再次声明同样名字的`.aidl`文件。这里没发现有特别好的解决方案，暂时只能先声明一个不同名字的文件，然后再改回去。

`AIDL`文件没什么好关心的，但是通过`AIDL`文件生成的`.class`文件则可以好好研究下：

`AIDL`生成的实际上是一个接口文件，在这个接口类中首先是定义了`AIDL`文件中规定的方法，然后生成了一个接口内部的抽象类`Stub`，这个抽象类实现了外层的接口，并继承自Binder，这个内部抽象类就是在之后使用时在`Service`内部初始化并实现接口方法的Binder对象，听过`bindService`返回给调用者，调用者使用Binder调用远端方法。

在这里需要注意的是，与日常使用的进程内利用Binder调用远端代码不同的是，这里在获取Binder对象时调用的方法是：

    public void onServiceConnected(ComponentName name, IBinder service) {
        mITaskConsumer = ITaskConsumer.Stub.asInterface(service);
        mIsRemoteBond = true;
    }

这里`asInterfase(service)`是完成整个跨进程调用的关键。我们继续看看这个方法里面做了什么：

    public static com.kyangc.ipcdemo.ITaskConsumer asInterface(android.os.IBinder obj) {
        if ((obj == null)) {
            return null;
        }
        android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if (((iin != null) && (iin instanceof com.kyangc.ipcdemo.ITaskConsumer))) {
            return ((com.kyangc.ipcdemo.ITaskConsumer) iin);
        }
        return new com.kyangc.ipcdemo.ITaskConsumer.Stub.Proxy(obj);
    }

这里`obj`是通过bindService返回的Binder对象，这时会在本地按照`DESCRIPTOR`定义的特征值去查找进程内的服务，如果查到了，那么说明这个Binder是个本地服务，那么直接将这个Binder作为处理业务的对象返回给调用者；反之如果没有查到，说明这个是一个远程的服务，这个时候会用`Proxy`对这个Binder进行包装，然后返回给用户。

下面继续看这个`Proxy`做了什么事：

首先是这个`Proxy`是继承于我们定义好的接口的：

    private static class Proxy implements com.kyangc.ipcdemo.ITaskConsumer

然后在执行接口方法时，调用的是这一段代码：

    @Override
    public void consume(com.kyangc.ipcdemo.Task task) throws android.os.RemoteException {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
            _data.writeInterfaceToken(DESCRIPTOR);
            if ((task != null)) {
                _data.writeInt(1);
                task.writeToParcel(_data, 0);
            } else {
                _data.writeInt(0);
            }
            mRemote.transact(Stub.TRANSACTION_consume, _data, _reply, 0);
            _reply.readException();
        } finally {
            _reply.recycle();
            _data.recycle();
        }
    }

这里看上去多，但实际上只是一堆模板代码。核心步骤实际上只有三步：

-   将传入参数写入`_data`
-   调用`mRemote.transact(方法序号,传入序列化数据,传出序列化数据,标志位)`
-   写入返回值，处理异常，回收序列化变量

而`transact`方法则是调用了`/dev/binder`提供的方法，将调用什么方法、使用什么参数、返回什么参数等内容传到了远程服务端。

那么远程服务端收到`/dev/binder`的提醒之后会做什么呢？我们继续看`Stub`中的一段代码：

    @Override
    public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply,
            int flags) throws android.os.RemoteException {
        switch (code) {
            case INTERFACE_TRANSACTION: {
                reply.writeString(DESCRIPTOR);
                return true;
            }
            case TRANSACTION_consume: {
                data.enforceInterface(DESCRIPTOR);
                com.kyangc.ipcdemo.Task _arg0;
                if ((0 != data.readInt())) {
                    _arg0 = com.kyangc.ipcdemo.Task.CREATOR.createFromParcel(data);
                } else {
                    _arg0 = null;
                }
                this.consume(_arg0);
                reply.writeNoException();
                return true;
            }
        }
        return super.onTransact(code, data, reply, flags);
    }

当`/dev/binder`把信息发到远端时，会调用到`Stub`中的`onTransact`方法，这个方法会接下来继续分析客户端调用了什么方法，从`_data`中取出参数，计算后将结果放到`_reply`中。注意这里有一个返回值，如果返回为`false`，那么代表该次请求失败，反之成功，这个特性可以被利用于权限验证。

这里需要注意的一点是，客户端的线程会在请求后被挂起，之后所有的操作均发生在各自Binder的线程池中，直到请求返回才停止主线程的阻塞。因此尽量不要再主线程发起远端的业务请求，尽可能在新的线程中发起业务需求通知。

下图简单描述了一下这个过程：

![](https://ww3.sinaimg.cn/mw690/825558b1gw1f3hfkvmvgrj20dg0daaax.jpg "Binder的调用过程")

# Android中其他IPC方式

## Bundle

非常常用的跨进程传输数据的方式(`Intent`)，只支持`Parcelable`数据的传输。

## 文件共享

必须手动控制并发，很容易出问题，效率较低。例如`Sharedpreference`，系统对于SP是有一定的缓存策略的，在多进程模式下会变得非常不可靠。

## Messenger

一种基于`AIDL`的轻量级IPC手段。底层由`Handler`+`AIDL`实现，服务端存在一个固定宽度为1的消息队列，因此没有并发的问题。通过指定`Message`中的`replyTo`字段来完成服务端的消息转发。

## ContentProvider

底层实现依然依赖于Binder。实现自定义的ContentProvider只需要实现对应的CRUD方法以及一个生命周期方法和一个返回MIME类型的getType方法即可。ContentProvider的储存方式是任意的，可以使用数据库、文件甚至是内存中的一个对象。

这里需要注意的是，ContentProvider的OnCreate方法是调用在UI线程中的，因此需要注意不要放入太多耗时操作。而其他CRUD方法则是运行在Binder线程池中，因此是存在多线程并发访问的可能性的，ContentProvider本身没有对这些方法做处理，需要用户自己管理并发。

## Socket

Socket运行在TCP/UDP层上，本身支持传输任意字节流，因此也可以被用于IPC。
