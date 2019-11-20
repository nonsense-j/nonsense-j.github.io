layout: post
title: 一、Redis安装
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
date: 2019-08-01 16:12:02
---

# 1、前言
其实Redis安装教程网上有很多，这里记录下来主要是记录自己的实践流程。之前学习过一些Redis的知识，但是都是朦朦胧胧的，现在Redis技术越来越火。不管多小的项目都会凑一凑热闹，所以了解一下Redis还是很有必要的。所以才有了现在的开篇。
从安装开始吧。

 # 2、windows安装
软件：链接：https://pan.baidu.com/s/1JzjuFM30AAJd6jkf5pG7XQ 
提取码：oowy 

我使用的是安装版的，下载下来运行，下一步下一步就可以，注意安装路径，并且将路径加入Path 中就可以了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801144431193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

安装完成后，在服务中就可以找到Redis 服务。如果没有启动就启动，如果启动了，那么就可以直接使用Redis了。

然后在控制台输入`redis-cli`redis-cli就可以进去Redis啦，进行相关的操作。这里是没有设置密码，使用的是默认的6370端口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801144832245.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
上面输入的`keys *` 表示查询出redis 中所有的key。


# 3、Linux安装
本人装了一个Linux虚拟机，xshell6连接上去的。

## 3.1 下载解压
首先下载资源：最新的应该是4.0.9
```
wget http://download.redis.io/releases/redis-4.0.9.tar.gz

tar xzvf redis-4.0.8.tar.gz
```
## 3.2 安装

```
 cd redis-4.0.9/                           						//进入解压目录
 make                                     						 //编译
 cd src 														//进入src 目录
 make install PREFIX=/usr/local/redis							//进行安装到usr/local/下，方便部署开机启动

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801162227287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
## 3.3 部署
```
#将conf文件放大etc目录下
mkdir /usr/local/redis/etc

mv redis.conf /usr/local/redis/etc

```
进入src目录，移动 mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server到/usr/local/redis/bin/

```
mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /usr/local/redis/bin/
```

配置redis为后台启动

```
vi /usr/local/redis/etc/redis.conf //将daemonize no 改成daemonize yes
```

## 3.4 启动
启动服务端：
```
redis-server /usr/local/redis/etc/redis.conf
```
启动客户端

```
redis-cli
```

　　

# 4、设置登录密码
刚刚上面也看到了，直接输入

```
redis-cli
```
就直接进去了，这样总感觉不安全，并且我们后面肯定不是通过命令行来访问Redis的，还是需要在项目中使用才行，总会配置Redis的密码的。那怎么设置呢。
先来看看我们Redis的密码：

```
config get requirepass
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801151515183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
发现是没有密码的，现在设置一个，用了get 获取，当然用set设置啦。

```
config set requirepass 123456
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801151750393.png)
设置好之后，在想看看自己设置的密码是什么，发现没有权限，哈哈，这就证明你密码设置成功了，现在需要登录密码才能访问数据库。

```
auth 123456
config get requirepass
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801152019785.png)
但是这样设置的密码有一个问题，那就是把控制台关了，从新进入就会发现密码失效啦，这显然不是我们想要的，原来我们那样设置没有写到conf文件中，是不会重启生效的（**如果配置文件中没添加密码 那么redis重启后，密码失效**）。

所以我们需要修改redis.conf中的requirepass并重新启动。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801172840195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

杀死Redis服务端
```
#找到PID
netstat -anp|grep 6379

kill -9 PID
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801173056126.png)
重启后再进去就发现需要密码啦
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801173240679.png)

也可以这样进入

```
redis-cli -p 6379 -a 123456
```


后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)






