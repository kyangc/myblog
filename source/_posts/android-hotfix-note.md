title: Android 热修复技术学习笔记
date: 2017-01-08 23:00:00
categories: 技术
tags: [Android,热修复]
description: 热补丁不是请客吃饭
---

# 引言
对于热修复技术的了解、认识从很久之前就开始了。但是因为没有业务上的驱动力，对于许多业内很火热的框架技术只是浅尝辄止的了解了一下，并没有更加深入去了解这些技术。下季度堆糖不出意外会引入热修复框架，这也为我提供了一次去了解、学习热修复相关技术的一次机会。

这篇文章可能更多的是倾向于「学习笔记」，只是起一个拾遗、拾忆的作用，不会太过深入代码，也不会针对一些具体实现纠结过多，所以有不详细、不确切、含糊不明的地方，还请各位看官（如果有的话- -）拿起手中的 Google 自己去探究吧科科。

文末附有本文所有参考借鉴过的文章的链接，感谢这些厉害的程序员在这一领域中做出的杰出贡献。

# 总览
本文不对热修复的概念做过多探讨，简单来讲，不发布新版本即完成对线上应用运行的代码进行更新的操作都能被算作热修复，所以像ReactNative/Weex这些从网页技术衍生来的混合开发技术其实也是一种热修复，只不过这些技术更加倾向于「业务更新」而不是「错误修复」，因此并不在本文的讨论范畴中。

按照主流的分类方法，Android生态内的热修复方案主要分为两个「流派」，每个流派又有不同的处理问题的思路：

* Native
	* Xpose/AOP - Dexposed
	* Native method hook - AndFix
* Java
	* Classloader - Qzone/Nuwa/RoocoFix..
	* Byte Code Injection - Robust/Instant Run-冷插拔
	* Dex replace - Tinker/Instant Run-热插拔

下面我们会对每个「流派」比较有代表性的技术及其背景进行介绍和比较，我们在选择热修复技术的时候主要针对以下几个方面进行比较。

# Native 部分
主流的Base Native 的热修复方案除了AndFix还活跃在主流的热修复框架中，其余的框架基本都处于欠维护状态。下面我们挑选两个比较有代表性的框架进行简单的介绍和比较：

## Dexposed
基于Xposed开发的AOP框架，方法级细粒度，来自手淘团队。Xposed需要Root权限，但是对于单个应用而言并不需要Root。其利用 Xposed 框架修改Android Dalvik运行时的Zygote进程，并利用Xposed Bridge hook方法并注入自己的热修复代码，以达到非侵入的runtime修改。

应用启动时会fork zygote 进程，装载各种class 和invoke各种初始化方法，xposed框架就在这个时候替换了app\_process，hook了各种入口级方法，从而实现之后的各种方法前后的拦截。Dexposed的hook并不限制于应用本身的业务代码，任何应用运行时执行的方法都可以进行hook，在绕过一些Android系统本身的Bug这种普通方式很难完成的事情上有着得天独厚的优势。

不过Dexposed框架因为无法支持ART虚拟机（Xposed不支持ART虚拟机），在ART渐渐成为Android主流虚拟机的现在显然已经不能满足需求，并且该项目目前已经停止维护了。

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-05%2016.20.31.png "Dexposed 支持的系统")

## AndFix
AndFix是来自Alibaba团队的另一个热修复框架作品，阿里百川的 Hotfix方案就是基于该方案统一工具链修改而来。该框架的原理和Xposed在大范围内对native方法进行hook不同，AndFix只对需要修复的方法进行hook。hook的原理很有意思：

开发人员对线上问题进行修改，修改完后通过工具检查新代码和问题代码之间的方法差异，并将这些差异信息写入smali文件，并在每个方法前增加注解标注，然后将所有差异信息打包生成dex文件，连同许多安全校验信息一起打包下发到问题客户端。客户端得到补丁信息之后开始在程序开始的时候载入带有修复方法的dex，然后根据dex中注解提供的参数遍历并找到原有dex中需要替换的方法，找到方法之后首先修改需要替换的方法为native方法，然后在native层对这个方法的调用进行hook，将其指向补丁包中的对应方法地址，以此完成对于方法的替换。

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-principle.png "AndFix原理")

具体的代码这里就不贴了，有兴趣的可以点击文末的链接进去仔细阅读。这个方法最大的优点是足够动态，理论上可以不用重启替换所有的方法，修改基本是即时生效的。而缺点也很明显，该方法不能动态的增减类中的字段，对部分机型不支持，修改之后的方法参数类型也有限制，而且同一个方法不能进行多次patch。

所以AndFix最适用的场景可能还是对于线上突发问题的修复，替换部分问题方法，让用户免于崩溃，至于说新增业务，资源替换，或者是较大规模的修改，可能比较力不从心。不过AndFix的思路真的很有意思，值得学习借鉴。

# Java 部分
从上面的叙述中我们不难看出，Native层面上的Hotfix其主要的思路还是寻找Java调用与Native调用的结合点，在这种情况下，新增方法、修改资源等难以与Native结合的问题点相对而言就比较无能为力了。在这种状况下，一些Java世界中的热更新框架也渐渐出现在开发者面前。

## 前置知识
在进行框架介绍之前，可能有一些前置知识需要简单的介绍一下，否则后面的内容将难以理解。

### 一个应用是怎样从代码变成手机上运行的程序的？
分开讲吧，我们先来看看一个可以安装到手机上的.apk文件是怎样构建出来的吧：

1. 使用aapt打包资源文件，生成 R.java 文件
	* 清单文件、资源文件都会被编译，生成唯一ID放入R.java
2. 处理AIDL文件，声称对应的.java文件
3. 使用Javac编译器编译所有的源代码.java文件，生成JVM使用的.class文件
	* 在这一步中如果配置有混淆，那么将使用ProGuard将.class文件中的字节码进行混淆处理。
4. 使用dx将.class文件生成Dalvik虚拟机可执行的classes.dex文件
	* 该过程可以将java字节码转换成dalvik字节码，并压缩常量池、消除冗余信息。
	* 每个dex文件最大方法数为64k，如果应用方法数超过该限制，在应用multidex的应用中，该步骤会生成多个dex
5. 使用apkbuilder把没有编译的资源、编译过得资源、.dex文件打包为一个.apk文件。
6. 签名
7. 使用zipalign进行对齐处理，提升访问速度。

OK，到这里我们就得到了一个可以运行在Android设备上的apk程序包了。众所周知，Android 系统中的应用不同于普通的Java应用运行在JVM上，Android应用程序均运行在Android系统提供的ART/Dalvik虚拟机上，我们在上一步中通过dx生成的dex文件就是dalvik虚拟机接收的字节码文件格式。

讲到这里我们稍微插入一小段知识：

> Dalvik/ART虚拟机和普通的Java虚拟机的差异在哪里？
> 
> * 核心差异：JVM架构是Stack-Based，基于栈的架构，Dalvik虚拟机的架构为Reg-Based，基于寄存器的架构。JVM之所以采用基于栈的架构，是为了更好的适应所有的底层系统，不对处理器的reg数做假设，成为一个真正的「可移植」虚拟机；Dalvik虚拟机基于寄存器的架构执行效率更高，更加适合提前优化，加上手机处理器多为多reg的ARM系统本身也更加适合这种reg-based的虚拟机。
> 
> * 由于核心架构的差异，.dex文件的字节码和.class文件的字节码是不一样的，下图可以比较清晰的说明这一点。
> 
>   ![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-102038a756f731212ebfcec193217c37_b%20-1-.jpg "Dalvik JVM 虚拟机字节码文件差异")
> 
> * JVM中通常会在同一个虚拟机中运行许多程序，而在Dalvik中，则采用了Zygote模式：在Android系统中，应用程序进程都是由Zygote进程孵化出来的，而Zygote进程是由Init进程启动的。Zygote进程在启动时会创建一个Dalvik虚拟机实例，每当它孵化一个新的应用程序进程时，都会将这个Dalvik虚拟机实例复制到新的应用程序进程里面去，从而使得每一个应用程序进程都有一个独立的Dalvik虚拟机实例。Zygote进程在启动的过程中，除了会创建一个Dalvik虚拟机实例之外，还会将Java运行时库加载到进程中来，以及注册一些Android核心类的JNI方法来前面创建的Dalvik虚拟机实例中去。注意，一个应用程序进程被Zygote进程孵化出来的时候，不仅会获得Zygote进程中的Dalvik虚拟机实例拷贝，还会与Zygote一起共享Java运行时库，这完全得益于Linux内核的进程创建机制（fork）。这种Zygote孵化机制的优点是不仅可以快速地启动一个应用程序进程，还可以节省整体的内存消耗，缺点是会影响开机速度，毕竟Zygote是在开机过程中启动的。
> 
>   ![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-dea144e77ac583b16d7a866ed8d5e891_b.jpg "Zygote模式")
> 
> * 也即是说，在Android系统中，有多少个应用在运行，那么就有多少个虚拟机正在运行，而这与JVM单虚拟机多程序的架构相去甚远，但这也是移动设备为了适应小内存、低性能所采取的非常有意义的改变。
> 
> * 除了指令集和类文件格式不同，Dalvik虚拟机与Java虚拟机共享有差不多的特性，它们都是解释执行，并且支持即时编译（JIT）、垃圾收集（GC）、Java本地方法调用（JNI）和Java远程调试协议（JDWP）等

好，现在我们已经准备好了安装包，接下来，.apk文件是如何安装在Dalvik/ART虚拟机上的？他们又是如何运行的呢？

首先我们来看安装：当我们安装应用的时候，Dalvik和ART会采取不同的方式去优化加载到本地的dex文件：

* Dalvik 环境下，安装服务PackageManagerService会通过守护进程installd调用一个工具dexopt对打包在APK里面包含有Dex字节码的classes.dex进行优化，优化得到的文件保存在/data/dalvik-cache目录中，并且以.odex为后缀名，表示这是一个优化过的Dex文件。
* ART 环境下同样安装服务PackageManagerService会通过守护进程installd调用另外一个工具dex2oat对打包在APK里面包含有Dex字节码进翻译。这个翻译器实际上就是基于LLVM架构实现的一个编译器，它的前端是一个Dex语法分析器。翻译后得到的是一个ELF格式的oat文件，这个oat文件同样是以.odex后缀结束，并且也是保存在/data/dalvik-cache目录中。
无论是Dalvik VM环境下的.odex文件，还是ART环境下的.odex文件，最终在运行程序时都需要将DEX文件载入进虚拟机，只不过DVM状态下，可能会通过Interpreter（解释器）或者JIT去把字节码转换成机器码最终执行，而在ART状态下，这些字节码会在安装时被AOT的转换成机器码存在同样以.odex为后缀的OAT文件中，使用时就不再在运行时去解释了。在程序运行时，程序依赖的系统代码会连同程序的代码一起生成一个OAT文件加载进虚拟机，所以一个OAT文件内部其实可能会含有多个DEX文件的。如果运行时有加载额外的dex文件，其同样会以该方式生成oat文件加载进ART虚拟机。
* 在AndroidN中，ART采取了一种更加「聪明」的方式去处理——混合编译，简单来讲，就是JIT、解释、AOP三种方式共存，其中的策略、优劣势、对于热修复有什么影响，请继续阅读文末给出的链接。

OK，说了这么多，似乎有点偏题，净是在说什么虚拟机啊dex啊的，这和我们讨论热修复的主题有何关系？当然有关系，让我们把目光往上挪一层，我们之前了解了在安装应用时是怎么把应用代码本地化到系统中的，DVM生成了.odex文件、ART生成了OAT文件缓存在了本地，那么，这些DEX文件又是怎么在虚拟机启动过程中被加载到JavaHeap里作为一个个Class对象供以使用的呢？这里必须要讲到Android的类加载机制了。

### Android 的类加载机制
首先我们花几分钟时间来简单的回顾一下Java世界中的ClassLoader机制：

> 双亲委托模型：源ClassLoader收到加载类或资源请求时，首先委托父ClassLoader进行加载，如果已经加载则直接返回，否则继续向上委托直到遍历到始祖类加载器。若始祖类加载器依然没有对应的类或资源，则从始祖类加载器开始，尝试从当前类加载器对应的类路径下寻找class字节码并载入，如果成功则返回class，如果失败则将加载请求委托给子加载器，一直遍历到源ClassLoader直到成功载入该class，否则抛出异常。

从前文我们知道，Android虚拟机标准和普通的JVM不一样，它们没有.class文件，而是在编译之后将所有.class文件封装成了.dex文件，在安装时又被优化成了.odex文件，那这些.odex文件的类加载又会有什么不同呢？让我们接着往下看：

在Android世界中，同样有ClassLoader类，该类为一个抽象类，其子类由以下部分组成：

* ClassLoader
	* SecureClassLoader
		* URLClassLoader - 加载 jar 文件，在Android上无法使用
	* BaseDexClassLoader
		* PathClassLoader - 在应用启动时创建，从应用目录下加载 apk 文件，只能加载已经安装的 dex或apk文件。
		* DexClassLoader - 类似于PathClassLoader，不过它能够加载来自于其他外部路径的Dex文件 —— 这也是许多热修复的基础，在不需要安装应用的情况下，完成需要的Dex的加载。
	
无论是PathClassLoader还是DexClassLoader，都只是BaseDexClassLoader的封装，具体的类加载过程都在BaseClassLoader中完成的，下面我们来看它究竟做了什么事情：

* 在外部通过`loadClass(String className)`并遍历双亲得到 class 实例

```java
public Class<?> loadClass(String className) throws ClassNotFoundException {
    return loadClass(className, false);
}

protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
    Class<?> clazz = findLoadedClass(className);
    if (clazz == null) {
        ClassNotFoundException suppressed = null;
        try {
            clazz = parent.loadClass(className, false);
        } catch (ClassNotFoundException e) {
            suppressed = e;
        }

        if (clazz == null) {
            try {
                clazz = findClass(className);
            } catch (ClassNotFoundException e) {
                e.addSuppressed(suppressed);
                throw e;
            }
        }
    }
    return clazz;
}
```

* `loadClass`方法调用了`findClass`方法，BaseDexClassLoader重载了这个方法：

```java
@Override
protected Class<?> findClass(String name) throws ClassNotFoundException {
    Class clazz = pathList.findClass(name);
    if (clazz == null) {
        throw new ClassNotFoundException(name);
    }
    return clazz;
}
```
	
* 结果还是调用了 DexPathList的findClass

```java
public Class findClass(String name) {
    for (Element element : dexElements) {
        DexFile dex = element.dexFile;
        if (dex != null) {
            Class clazz = dex.loadClassBinaryName(name, definingContext);
            if (clazz != null) {
                return clazz;
            }
        }
    }
    return null;
}
```

* DexPathList中的dexElements通过下面方法得到
	
```java
private static Element[] makePathElements(List<File> files, File optimizedDirectory,
        List<IOException> suppressedExceptions) {
    List<Element> elements = new ArrayList<>();
    // 遍历所有的包含 dex 的文件
    for (File file : files) {
        File zip = null;
        File dir = new File("");
        DexFile dex = null;
        String path = file.getPath();
        String name = file.getName();
        // 判断是不是 zip 类型
        if (path.contains(zipSeparator)) {
            String split[] = path.split(zipSeparator, 2);
            zip = new File(split[0]);
            dir = new File(split[1]);
        } else if (file.isDirectory()) {
            // 如果是文件夹,则直接添加 Element,这个一般是用来处理 native 库和资源文件
            elements.add(new Element(file, true, null, null));
        } else if (file.isFile()) {
            // 直接是 .dex 文件,而不是 zip/jar 文件(apk 归为 zip),则直接加载 dex 文件
            if (name.endsWith(DEX_SUFFIX)) {
                try {
                    dex = loadDexFile(file, optimizedDirectory);
                } catch (IOException ex) {
                    System.logE("Unable to load dex file: " + file, ex);
                }
            } else {
                // 如果是 zip/jar 文件(apk 归为 zip),则将 file 值赋给 zip 字段,再加载 dex 文件
                zip = file;
                try {
                    dex = loadDexFile(file, optimizedDirectory);
                } catch (IOException suppressed) {
                    suppressedExceptions.add(suppressed);
                }
            }
        } else {
            System.logW("ClassLoader referenced unknown path: " + file);
        }
        if ((zip != null) || (dex != null)) {
            elements.add(new Element(dir, false, zip, dex));
        }
    }
    // list 转为数组
    return elements.toArray(new Element[elements.size()]);
}
```
	
* 其中`loadDexFile()`方法最终会调用JNI方法载入dex对象，这里我们不再深入的去涉及了。
* 得到dex文件后通过调用`loadClassBinaryName`得到最终的class对象。这里在`loadClassBinaryName`方法中，其实最终调用的依然是Native的`defineClass`方法，这与JVM中`loadClass`的方法名一致，不知道是故意还是巧合。

```java
public Class loadClassBinaryName(String name, ClassLoader loader){
    return defineClass(name, loader, mCookie);
}

private native static Class defineClass(String name, ClassLoader loader, int cookie);
```

OK，到这里我们基本大概对Android虚拟机使用的字节码文件有了初步了解，对Android系统中的类加载机制也有了初步的认识，知道了Android虚拟机中的类对象究竟是怎么从.dex文件加载进虚拟机的。这也为我们接下来真正进入Java世界中热修复框架原理打下了基础。

## 基于ClassLoader的热修复原理
经过以上啰啰嗦嗦杂七杂八的前置知识铺垫之后，我们终于进入了正题，趁热打铁，我们首先来看一下最经典的基于ClassLoader的热修复方案：

如前所述，我们在载入Class的时候，会调用DexPathList对象中的findClass方法，findClass方法则会遍历dexElements数组，当发现首个dex对象的时候则直接返回。若我们在此处将修复了问题之后的dex插入到这个dexElements数组的最前方，那么不就可以直接加载到打完补丁之后修复好的类了吗？那么热修复也就完成了。

原理很简单，实现也不难，但是这里面有一个很重要的问题需要解决：``CLASS_ISPREVERIFIED``

这是个Class内部的标示，在应用安装时，系统通过dexopt或dex2oat进行dex优化时进行设置。当该标示位为真，则表示这个类直接引用到的类与该类都在同一个dex中。那么事情就变成了这个样子：假设类A直接引用了类B，类A与类B在安装之初经过校验发现在同一个Dex中，`CLASS_ISPREVERIFIED`被置为真。

此时类B出错，使用类B’代替，当类A中再次调用类B(类B’)，虚拟机因为`CLASS_ISPREVERIFIED`标示缘故对类A与类B’的Dex来源进行校验，此时因为类B’来自下发的补丁包，校验不通过，虚拟机崩溃退出。

OK，为了解决这个问题，开发人员从`CLASS_ISPREVERIFIED`的置空条件入手：若类A中引用了一个在同一Dex种的类B，同时也引用了不在同一Dex中的类C，那么包括类A类B类C都不会被打上`CLASS_ISPREVERIFIED`标签。于是开发人员只要保证在所有类的构造函数中调用一个第三方Dex提供的类X，就可以保证所有类均不会被打上`CLASS_ISPREVERIFIED`标签。

但是这个方案不是没有问题，在DVM中，因为所有类都是非preverify的状态，这导致verify与optimize操作会在加载类时触发。单次的verify+optimize耗时并不长，而且这个过程只有一次，但是当应用启动时，会一次性载入数量庞大的类，这时的性能影响就不容忽视了。

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-qzone-dalvik.png "Dalvik 类加载时优化")

而在ART中，由于ART采取了新的方式，这种处理对代码的执行效率没有太大影响，但是如果不定的类中出现修改类变量或者方法的情况，则会导致出现内存错乱的问题——因为在安装应用时，dex2oat已经将能够确定的各个地址全部写死为机器码，如果运行时补丁包的地址出现改变，原始类去调用时就会出现地址错乱。为了解决这个问题我们需要将修改了变量、方法以及接口的类的父类以及调用这个类的所有类都加入到补丁包中。这可能会带来补丁包大小的急剧增加。

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-05-qzone-art%20-2-.png "ART 环境下的变量错乱")

总的来说，ClassLoader方案好处在于开发透明，简单，这一套方案目前的应用成功率也是最高的，但在补丁包大小与性能损耗上有一定的局限性。

## Instant Run
在我们继续接下来的叙述之前，我觉得需要用一个独立的章节来叙述Google官方推出的「热更新」框架——Instant Run。

它是去年AndroidStudio2.0发布的时候Google引入的一项用于提升开发效率的新的IDE特性，这项技术能够让我们在初次构建并部署应用到手机后，后续修改能够在不经过重新安装应用即可完成部署——这不就是一种热更新吗？

虽然Instant Run由于只能运行在IDE环境下、Android系统版本要求也在5.0以上等原因无法作为一个真正的热更新框架，但是其内部的原理、思路为真正的热修复框架提供了思路：

Instant Run的热更新分为三个层次，热插拔、温插拔、冷插拔。热插拔情况下的修复，应用无须作任何操作即可更新方法的实现。温插拔则是在热插拔的基础上增加了对资源的更新，开发者只需要重启Activity即可完成更新。冷插拔则是应对更加大范围的修改，如类结构变更、方法名变更等等问题，这时开发者需要重启应用以完成更新。

Instant Run的核心设计有以下几点：

* **编译期注入字节码**

我们在通过Instant Run构建应用的时候，Instant Run通过Gradle的[Trasform api](http://tools.android.com/tech-docs/new-build-system/transform-api "Trasform api")处理了Javac生成的所有.class文件，为每个类都提供了一个字段change，该字段实例实现了IncrementalChange接口，并且在每个方法最前方插入了一段代码来判断是否需要调用一段插入的代码以作为修复之后的调用。话说起来可能比较抽象，看看图吧，原理其实很简单：

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-06-1480489905685.png "Instant Run 原理")

与此同时，在生成.class文件的时候，Instant Run同时也修改了Manifest文件，生成代码注入了一个BoostrapApplication作为原来Application类的代理，以实现修改后补丁文件的监听和ClassLoader的注入。

* **多ClassLoader机制**

在BootstrapApplication里，Instant Run利用ClassLoader的双亲代理机制，在原有的PathClassLoader之上注入了一个IncrementalClassLoader用于生成所有修改后的补丁类，同时也由于双亲委托机制的存在，IncrementalClassLoader也成为了所有类的加载器，拦截了程序中所有的类加载请求。在IncrementalClassLoader中，每个热补丁类都是由不同的ClassLoader实例创建的，这一点是整个Instant Run的核心所在，因为修改后的类实际上和修改前的类是同一个类，如果使用同一个ClassLoader是无法完成类加载的。

* **全量资源替换**

在Instant Run中，在替换Application的同时，也会对资源相关对象进行替换，将资源目录指向另一个位置。此时，如果发生了资源的修改，那么温插拔会被触发，Gradle会自动将所有资源重新打包并替换掉该资源目录下的资源，同时重启Activity完成资源更新。

* **Dex分片（Dex-Slice）**

在应用构建的时候，Instant Run会通过Gradle插件对Dex文件按照包名进行分片（也可以叫分包），最多把Dex分为10片部署到手机上。在开发者做出代码上的修改之后，Instant Run会判断修改的内容，如果改动无法通过热插拔完成，那么会对修改类所在的Dex进行全量构建并下发替换原有的Dex片，在这种状况下，实际上新的Dex是通过PathClassLoader加载进来的，因此必须通过重启应用触发类加载来完成载入。

以上就是Instant Run比较核心的一些设计，我们可以看到，从注入代理Application拦截原生ClassLoader、修改资源路径完成资源替换、注入字节码实现热修复到覆写分片Dex完成全量更新，Instant Run在针对不同的状况采取了不同的措施，逻辑清晰、步步为营，非常值得我们学习。

从Instant Run热插拔的思路出发，美团团队利用相似的原理开发出了热修复框架Rubost；从Instant Run冷启动的思路出发，微信团队则开发出了热修复框架Tinker。Rubost框架原理这里不需要过多的细讲，和热插拔类似，只是针对方法数、分包等问题进行了优化。下面我们仔细看看Tinker的思路：

## 基于Dex全量替换的热修复框架 - Tinker
Tinker的思路很简单——全量替换Dex。是不是有一种暴力美学的意味在里面？但是为了实现这个目标，却不得不放下手中的加特林，拿起绣花针把里面一个一个的坑都给踩平。

全量更新包的大小问题首先就摆在了面前——我当然可以下发一整个Dex给你替换，但是全量下发动辄十几兆的Dex文件真的大丈夫？这里Tinker采取了自研DexDiff算法，通过下发差异文件，在客户端本地合成新的Dex文件作为更新后的Dex文件。这件事听起来也很直接，但是为了最大程度的压缩差量包、最快的生成更新后的Dex文件，Tinker团队必须对Dex文件格式、Dex文件生成过程了如指掌、必须对算法的性能有最高的要求，用shwenzhang自己的话来说：

> 这不仅要求我们需要研究透Dex的格式，也要把dex2opt与dex2oat的代码全部研究透。现在回想起来，这的确是一条跪着走完的路。与研究Dalvik与Art执行一致，这是经历一次次翻看源码，一次次编Rom查看日志，一次次dump内存结构换来的结果。

Tinker做到的事情当然不止这么多，ART/Dalvik差异化执行、AndroidN混合编译的支持等等天坑都被他们填了过去，具体的技术细节这里就不分析了，这里附一张图看看Tinker都能做到些什么吧：

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-08-0.jpeg "Tinker与各种热修复框架技术对比")

# 总结
断断续续的写了两三天，终于把这篇笔记完成了……回过头去再读了一遍全文，感慨良多。就热修复技术而言，确实是一门太需要持续投入时间的技术了，技术做出来很容易，但是做得好真的太难。就用shwenzhang的一句话为本文做结吧：

> 热补丁不是请客吃饭

向那些在某些领域内深耕不辍的工程师致敬，有朝一日希望我也能成为他们那样优秀的人。

# 参考
* 综述
	* [各大热补丁方案分析和比较 - markzhai](http://blog.zhaiyifan.cn/2015/11/20/HotPatchCompare/)
	* [[Android热修复] 技术方案的选型与验证](http://www.jianshu.com/p/1683c4e6f36d "[Android热修复] 技术方案的选型与验证")
	* [Android 热修复方案对比 - itscoder](http://jaeger.itscoder.com/android/2016/08/28/android-hot-fix.html)
	* [安卓App热补丁动态修复技术介绍 - QZone](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a)
* 虚拟机
	* [Android运行时ART简要介绍和学习计划 - 老罗](http://blog.csdn.net/luoshengyang/article/details/39256813 "Android运行时ART简要介绍和学习计划")
	* [Dalvik虚拟机简要介绍和学习计划 - 老罗](http://blog.csdn.net/luoshengyang/article/details/8852432)
* ClassLoader
	* [热修复入门：Android 中的 ClassLoader](http://jaeger.itscoder.com/android/2016/08/27/android-classloader.html)
	* [从源码分析 Android dexClassLoader 加载机制原理](http://blog.csdn.net/nanzhiwen666/article/details/50515895)
	* [Android动态加载基础 ClassLoader工作机制](https://segmentfault.com/a/1190000004062880)
	* [Android类加载机制的细枝末节](http://www.jianshu.com/p/3afa47e9112e)
* Instant Run
	* [Instant Run: How Does it Work?! - Google](https://medium.com/google-developers/instant-run-how-does-it-work-294a1633367f#.luxanx6cm "Instant Run: How Does it Work?!")
	* [从Instant run谈Android替换Application和动态加载机制 - w4lle](http://w4lle.github.io/2016/05/02/%E4%BB%8EInstant%20run%E8%B0%88Android%E6%9B%BF%E6%8D%A2Application%E5%92%8C%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/ "从Instant run谈Android替换Application和动态加载机制")
	* [Android Studio的Instant Run(即时安装)原理分析和源码浅析 - coding-way](http://www.cnblogs.com/coding-way/p/5443718.html "Android Studio的Instant Run(即时安装)原理分析和源码浅析")
	* [ 从Instant-Run出发，谈谈Android上的热修复 - zjutkz](https://gold.xitu.io/entry/5731f50ef38c840067dcce48)
* Tinker
	* [微信热补丁实践演进之路 - PPT](https://github.com/WeMobileDev/article/blob/master/final-%E5%BE%AE%E4%BF%A1%E7%83%AD%E8%A1%A5%E4%B8%81%E5%AE%9E%E8%B7%B5%E6%BC%94%E8%BF%9B%E4%B9%8B%E8%B7%AF-v2016-9-24.pdf)
	* [微信Tinker的一切都在这里，包括源码](https://github.com/WeMobileDev/article/blob/master/%E5%BE%AE%E4%BF%A1Tinker%E7%9A%84%E4%B8%80%E5%88%87%E9%83%BD%E5%9C%A8%E8%BF%99%E9%87%8C%EF%BC%8C%E5%8C%85%E6%8B%AC%E6%BA%90%E7%A0%81(%E4%B8%80).md)
	* [Android N混合编译与对热补丁影响深度解析](https://github.com/WeMobileDev/article/blob/master/Android_N%E6%B7%B7%E5%90%88%E7%BC%96%E8%AF%91%E4%B8%8E%E5%AF%B9%E7%83%AD%E8%A1%A5%E4%B8%81%E5%BD%B1%E5%93%8D%E8%A7%A3%E6%9E%90.md)
	* [ART下的方法内联策略及其对Android热修复方案的影响分析](https://github.com/WeMobileDev/article/blob/master/ART%E4%B8%8B%E7%9A%84%E6%96%B9%E6%B3%95%E5%86%85%E8%81%94%E7%AD%96%E7%95%A5%E5%8F%8A%E5%85%B6%E5%AF%B9Android%E7%83%AD%E4%BF%AE%E5%A4%8D%E6%96%B9%E6%A1%88%E7%9A%84%E5%BD%B1%E5%93%8D%E5%88%86%E6%9E%90.md)
	* [Tinker DexDiff](https://www.zybuluo.com/dodola/note/554061 "Tinker DexDiff")
* Rubost
	* [Android热更新方案Robust](http://tech.meituan.com/android_robust.html "Android热更新方案Robust")
	* [Android中热修复框架Robust原理解析+并将框架代码从”闭源”变成”开源”(上篇)](http://www.wjdiankong.cn/android%E4%B8%AD%E7%83%AD%E4%BF%AE%E5%A4%8D%E6%A1%86%E6%9E%B6robust%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%E5%B9%B6%E5%B0%86%E6%A1%86%E6%9E%B6%E4%BB%A3%E7%A0%81%E4%BB%8E%E9%97%AD%E6%BA%90%E5%8F%98/)
	* [Android中热修复框架Robust原理解析+并将框架代码从”闭源”变成”开源”(下篇)](http://www.wjdiankong.cn/android%E4%B8%AD%E7%83%AD%E4%BF%AE%E5%A4%8D%E6%A1%86%E6%9E%B6robust%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%E5%B9%B6%E5%B0%86%E6%A1%86%E6%9E%B6%E4%BB%A3%E7%A0%81%E4%BB%8E%E9%97%AD%E6%BA%90%E5%8F%98-2/)
* AndFix
	* [AndFix - Github](https://github.com/alibaba/AndFix)
	* [Android热补丁之AndFix原理解析](http://w4lle.github.io/2016/03/03/Android%E7%83%AD%E8%A1%A5%E4%B8%81%E4%B9%8BAndFix%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/ "Android热补丁之AndFix原理解析")
* Freeline
	* [Freeline - Android平台上的秒级编译方案](https://yq.aliyun.com/articles/59122)