---
title: 老破博客复活记
date: 2019-02-25 00:02:38
tags: [折腾,hexo,七牛,备案]
description: 阿日的博客活又过来惹！
categories: 折腾
---

# 背景

荒废好久的博客，前段时间 Lancy 居然翻出来其中一篇文章的引用，震惊之余略感尴尬，这地方确实……emm，一是年久失修，二是确实有不少挺羞耻的东西在上面。思来想去下决心要好好整治一下这里，当你——我未知的、一年就那么几个的读者——能看到这篇 blog 的时候，应该是已经搞定了大部分东西，下面简单记录一下这个 blog 是怎么复活过来的吧，过程其实还真挺有趣的，2333

# 升级 Hexo

原来的博客貌似还是基于 Hexo 很早期的版本开发的，虽然我的确说不出来旧版本和新版本 Hexo 有啥区别……但是干他妈的，升就对了！莽到最高版本！RUA！

# 升级 Next

[Next](https://theme-next.org/)实乃我国程序员人手一个的主题，不知不觉已经升级到了 7.x 版本，啥也别说了，升就对了！莽到最高版本！RUA！
额……然后发现配置文件都变了……变得还真鸡儿多……我只能一个个挨着把原来的配置文件再搬迁了一遍………
搬迁的过程中，发现新版 Next 对于很多插件采取了分仓库维护的方案，很多功能要启用的话，需要依赖源码，需要将这些依赖以源码仓库作为 next project 的 submodule 的方式依赖进来，很多依赖其实都没咋更新的，基本可以认为是终极稳定版本，新版 Next 搞这个优化说实话真是闲的蛋疼……
不过，折腾了半天之后，终于在本地跑起来 blog 惹！RUA！

# 然后遇到问题了……

emmm，啥问题呢，就是我发现我原来 [qiniu](qiniu.com/) 的图床挂了……
而且，如果仅仅只是挂了都还好，关键是 qiniu 取消了我原来的临时域名的访问权限，如果不额外绑定一个域名的话，原来仓库里面的图片你连下载下来都做不到……
黑人问号？这什么鸡巴鬼逻辑，流氓啊？家里排行老二啊？你咋这么屌呢？
……吐槽归吐槽，我想了下，这个事儿不难啊，我再绑定一个域名就完了呗？
对，的确可以再绑定一个域名，但是吧，这个域名必须是经过备案的域名……
域名我有啊，备案？啥是备案？
……艹！备案！

# 漫漫备案路

……妈妈，咱能回家不玩儿这个了吗，我这就 git revert，:)
咳咳，算了，知难而退不是我的作风，怎么着也得现在强国的边缘疯狂试探一番再走啊！
于是研究了一下备案是个啥玩意……
这一研究就是一天啊，我愣是没他妈看懂这鸡巴玩意有啥用……
可能就是我说了啥不利于国家的话，能够立刻定位到你本人然后请你喝星巴克吧……
嗯，对我没啥用，对国家有用就好（握拳
因为我的域名在阿里云，刚好就跟着阿里云的备案走了一遍整个备案流程……不得不说，是真的麻烦：

- 首先你得有一台阿里云的服务器，有效期三个月以上
- 然后你的域名得先经过实名认证
- 然后你开始填一大堆奇奇怪怪的个人信息，提交阿里云审核，如果有问题会有专人打电话给你让你改
- 然后等审核通过了，你需要打印一份文件签字画押三份寄给阿里云在贵州的某个办公室
- 然后下载一个上海管局的 app 在里面上传你的各种个人信息，等审核
- 然后等 app 审核通过了，纸版材料也受理并审核通过了，阿里云会帮你提交工信部继续审核
- 然后继续等吧，快的话过个几天应该就能审核通过了，慢的话……就等着罢！

好吧，我走完整个流程拿到 ICP 备案号，大概花了前后一个多星期，还算挺快的（……吧

# HTTPS 和 Netlify
拿到备案号的我仰天长啸啊哈哈哈哈这下狗日的牛老二拿我没辙了吧哈哈哈哈！
麻利的给我图床绑定了一个新的域名 `img.kyangc.com`，我终于可以看到我亲爱的图片了……mua~
正在我徜徉在我的博客中不能自拔的时候，Chrome 左上角那把小锁不断地刺激着我的眼球————都9102年了，您的博客还没全站 https 啊？
……被自己嘲讽了，呵，男人……
emmm，原来我的博客托管在 github.io 上，当然是没有证书的，想要给我的博客上一把绿色的小锁该咋整呢？
搜了一下，发现了一个神奇的网站…… Netlify.com
简单地说，这个神奇的网站，可以自动化的监听你 Github 上博客源文件的变更，然后自动运行博客部署程序，自动部署你的网站，并且你可以自定义域名，它还可以通过 Let’s Encrypt 自动给你的域名签发证书……
而且免费……
而且还带 CDN 部署……
而且部署速度贼快……
卧槽，这谁做的网站，家里有矿啊？给你比五十八个小心心啊！简直是个人静态博客维护神器啊……
想想以前麻烦得要死的博客发布流程……再看看现在……
时代变了啊！
改革开放好啊！
说干就干，Netlify 用起来还是很顺手的，功能不复杂，也没太多配置项，跟着引导几步操作就搞定了……
然后，发现个问题……
果然没那么顺利啊……
Netlify 对于递归的 submodule 支持很差……
啥意思呢，就是说，我 blog 源码里面依赖了 next 主题的 submodule，next 主题里面依赖了很多 lib 的 submodule，但是貌似 netlify 在部署的时候没有去拉取这些嵌套的 submodule……
这可就蛋疼了呀……
在尝试了改部署命令等等方案之后，没辙，只能自己手动添加这些插件的源码直接添加到 next 源码中，不再通过 submodule 去维护，这才总算把网站跑起来……

# 七牛我又回来啦！

网站部署的时候，Netlify 不断的提示我说，网站里有很多 http 图片链接，要我替换。
我仔细一看，哟，这不是我刚换过的牛二哥的图床链接吗，得，这就给我牛二哥带个套，啊呸，加个证书。
SSL证书签发服务七牛有，阿里云也有，本来我想着七牛自己给自己签发个证书应该挺快，结果……等了两天都没签下来……
于是我去了阿里云，2 分钟搞定……
2分钟……
2……
牛二哥你还是好好的入土为安吧。
同样是免费的证书，为啥差距就这么大？？
给我的 imgs.kyangc.com 域名绑定好证书之后，终于，我的博客实现了全站 Https 化！
至此，我的老博客……终于又活过来辣！

# 后记
年纪大了，有时候越来越会觉得好记性不如烂笔头，有空有精力就多记录点东西吧，越发觉得记忆力终归是不靠谱的……
所以这里以后应该会加快更新的频率，毕竟有了 Netlify 之后更新起来也不麻烦，很方便了呢！
应该今后这里不仅是技术相关，应该也会写一写其他的一些读书笔记之类的东西，甚至旅游游记也可以放放，啥都可以，嗯。

**今后，请多多指教！**