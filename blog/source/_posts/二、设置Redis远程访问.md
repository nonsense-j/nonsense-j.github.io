layout: post
title: 二、设置Redis远程访问
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
date: 2019-08-02 09:59:05
---

# 前言
昨天在Linux服务器上安装了Redis，那我怎么在本地直接连接redis呢，不需要先登录服务器，然后再连接redis的那种。
其实最初的问题是我想在项目中连接redis，进行使用，发现总是报连接不上，但是我通过xshell6连到虚拟机，然后连redis是没有问题的。所以才想应该是有什么配置需要修改才行。才有了下面的记录。

# 修改配置
进入redis.conf目录

```
vim  /usr/local/redis/etc/redis.conf
```
1、 将bind 127.0.0.1 注释掉
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190802095042617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
2、将protected-mode yes  改成  protected-mode no
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190802095147669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

3、重启redis服务
昨天用到的
```
查PID
netstat -anp|grep 6379

kill -9 PID

redis-server /usr/local/redis/etc/redis.conf
```

# 测试
在本地运行cmd

```
redis-cli -h ip -p port - a password
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190802095721858.png)


---

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)






