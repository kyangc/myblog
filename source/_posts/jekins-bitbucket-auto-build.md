title: Jekins+Bitbucket+Fir.im 自动构建部署最新测试包
date: 2017-01-04 10:00:00
categories: 技术
tags: [Android,技术] 
description: 对低效说不要不要啦
---

在日常开发中，我们的流程通常是：

> 开发-\>自测-\>提交测试-\>测试反馈-\>开发-\>……

很多有着成熟流程的公司在其中很多步骤上都能做到充分的解耦和自动化，开发无需关心应用的构建，只需要向特定分支提交符合标准的代码即可，测试也不需要关心开发是否开发完成、构建是否成功等等问题，只需要关注推送通知得知「有一个最新的测试包就绪，它相对于前次版本更新了xxxx」，并且能够很便捷的获取到应用，进行测试之后也能够把出现的问题对应到对应的版本反馈给开发。

堆糖的CI一直一来使用的都Jekins，前代开发已经在上面建立了众多的Project以用于构建各种版本的堆糖应用，但是可能是因为理解的偏差，只是把Jekins当做了一个「重复性任务执行脚本列表」来使用，只是让Jekins代替执行自己手写的Shell脚本，既没有利用到Jekins对于源代码的管理功能，也没有很好的利用Jekins提供的丰富的自动化插件，开发需要不定期告诉测试我们往xx分支提交了有xx feature的代码，测试则需要不定期去内部CI网站上配置构建工程，并等待构建结果生成——这样的状况无论是对于开发还是测试都是极大的生产力浪费，很多事情其实都是不需要这么做的。

最近业务压力稍小，就花了一下午时间开发并调试了一下Jekins，利用Bitbucket的WebHook功能，当某条分支上推送了最新代码时，自动触发Jekins的构建，构建完毕自动上传到公司内部的Fir.im，并通过BearyChat推送一条通知，告知测试最新的测试包已经构建完毕，以及本次构件包含的内容。整个流程无需人为干预，无论是测试还是开发都不需要关心CI的细节，只需要关心自己职责范围内的工作即可。
下面记录下都做了些什么事吧：

* 升级Jekins（内部服务器上的Jekins版本低的令人发指），安装BitBucket插件

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-install-bitbucket-plugin.png "安装 Jekins Bitbucket 插件")

* 在Bitbucket中配置WebHook，填入Jekins实例的地址

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-bitbucket-hook.png "Bitbucket 中配置")

* 填入需要触发构建的分支名，这里我们随便填个 Dev看看

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-bitbucket-config.png "Bitbucket 配置触发条件")

* Bitbucket上的配置已经完成啦，这里我们回到jekins上，我们新创建一个Project好了，在源码管理部分填入项目地址、构建分支、访问凭证等等东西

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-jks-git.png "Jekins 配置源代码管理")

* 在触发器上钩选 `Build when a change pushed to Bitbucket`

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-jks-trigger.png "Bitbucket 触发 Jekins 构建")

* 新增Execute shell 部分，在里面写好构建应用的脚本代码。这里算是整个部分最核心的地方了，清理构建环境、升级SDK、更新Build号等等事情，不过这里就不附图了，代码比较敏感。

* 配置Fir.im的上传插件

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-jks-firim.png "Jekins 自动将构建好的安装包上传到 Fir.im 上")

* 配置BearyChat机器人，通知事件。这里只通知构建成功，失败了的话就让他去吧……

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-jks-bc.png "BearyChat 通知构建成功")

* 构建完毕，收到通知，测试现在可以看到这次构建发生了什么变化，以及能够直接访问公司固定的Fir.im链接下载最新的安装包了。

![](http://pn0uv0rw1.bkt.clouddn.com/2017-01-09-bearychat-noti.png "BearyChat 接收通知")

至此基本所有的流程都走通了，整个过程虽然看上去简单，但是其中总会在某些不知名的角落卡住……比如Bitbucket和Jekins无法互相访问，这个只能联系公司运维解决；比如Jekins的工作环境总是会出一些各种各样的问题……这里也不细讲了，案例都比较个例。

可能这个案例本身并没有特别厉害的技术含量在里面，但这种对于任何低效保持「不妥协」的态度却是值得记录的。程序员要成长要提高，执着于「业务」开发是永远没办法达到很高的高度的，**只有保持对于「低效」「不适」的不满并且不断努力去解决这些痛点**——可能是流程上的繁琐，可能是框架代码的不合理，可能是开发效率的底下，可能是不愿意写很多模板代码——发现他们，解决他们，才能真正的让程序员不断的进步下去。
