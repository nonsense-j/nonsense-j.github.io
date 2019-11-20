layout: post
title: 内网穿透ngrok_ngrokcc_cpolar
author: QuellanAn
categories: 
  - 建站之路
tags:
  - 内网穿透
  - ngrok
  - ngrokcc
  - cpolar
date: 2019-09-05 18:46:02
---

# 前言
先来说说问题吧，我们的项目在测试环境上搭建好了，也就是在内网上可以正常运行，但是呢，局方的人想要看一下效果先，那问题就来了，不在同一个局域网，他们访问不了我们的内网啊，现在又想看。这咋整。所以就有了这篇文章。
这三个软件都差不多，都有一个免费的，我自己都试了一下，window和Linux的都可以，做演示的话问题不大。刚刚好满足要求。
# 官网
ngrok:[https://ngrok.com/](https://ngrok.com/)
ngrokc:[http://www.ngrok.cc/](http://www.ngrok.cc/)
cpolar:[https://www.cpolar.com/](https://www.cpolar.com/)

大家对着官网教程来就可以了，无非都是注册会员，领取那个免费的authtoken.然后下载客户端，设置端口，启动项目，用域名进行访问。

# 说明
1、ngrok 和 cpolar 基本用法都是一致的，都是生成一个authtoken ,然后设置一个ip和端口，会随机的生成一个http的域名和一个https 的域名随机访问。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190905185742950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
2、ngrokcc 是国内的一个网站，没有authtoken ,但是有隧道，需要你在控制台建好隧道，然后在客户端连接隧道id 就好了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190905190106504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

```
./sunny clientid 隧道id
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190905190543757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 总结
使用起来还是比较简单，能满足我们将项目搭建在本地，用公网访问演示。

# 番外
这几个是三个软件都是利用Nginx反向代理实现的。但是免费的只有一个隧道或者一个端口，这个时候我们可以在自己本地再搭建一个Nginx做虚拟主机，这样就可以自由飞翔了吧哈哈

# 再番外
因为看了一下官方文档，发现不仅可以代理一个端口网站，还能代理tcp协议。

我来举一个例子。我有两台电脑，一台装的是win ,一台装的是Ubuntu，一般我都是把Ubuntu当做服务器用。两台电脑在同一个局域网直接用xhell 的ssh 连接起来当然很方便啦。但是我有时候需要把win 带到其他地方，那做服务器的那台电脑就不能访问啦，
所以我参考一下官网的，可以在用外网访问这台服务器啦。
操作也很简单，我三个也都测试了一下，用的是cpolar 的，感觉比ngrok 要稳定些。
1、在我们Ubuntu服务器上安装好cpolar客户端，然后认证tocken 这些和之前是一样哒。
2、

```
cpolar tcp 22
```
就这么简单。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190906141633105.png)

然后在主机和端口号填上对应的就好了，就是这么简单。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190906141742822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

这样之后，就算两台电脑不在同一个局域网，也可以直接访问了，很实用。



后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)





