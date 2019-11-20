layout: post
title: 七、Redis持久化的两种方式RDB和AOF理解
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
date: 2019-08-22 19:53:09
---

# 前言
前面将了redis的主从复制以及怎么搭建，还介绍了哨兵模式以及哨兵模式的搭建。虽然操作跟上了，但是还是补一下redis的持久化。redis之所以这么流行，很大一部分原因便是持久化，断电重启数据不消失，使得redis在数据库领域中站稳了脚。前文将的主从复制其实就是依赖持久化的，如果没有持久化，这些数据都不会从主服务器备份到从服务器。下文我们就讲讲redis的持久化。

说起redis持久化，大家或多或少都知道一些，简单点一句话也能概括。redis通过RDB和AOF方式将数据存入磁盘，实现持久化。RDB是定期生成快照存入磁盘中，AOF是将写操作存入磁盘中。二者各有优劣，RDB 是存放数据库中数据，适合做数据备份，但是数据可能不全，最近几分钟的数据可能没有。AOF是每秒中执行一次，如果有写操作的命令就存储起来，最多只会丢失1秒钟的数据，适合做数据恢复。但是这个就不适合做数据备份了，并且由于每秒都会执行多多少少会抢占redis的内存，会影响性能。但是在实际应用中是二者是配合使用的。

下面就来具体的讲讲RDB和AOF吧
# RDB
RDB 的全称是 redis database. 顾名思义，RDB就是将redis数据库，用来存储数据的，所以通过RDB方式持久化就是将存在redis内存中的数据写入到RDB文件中保存到磁盘上从而实现持久化的。
RDB文件是一个压缩的二进制文件，通过这个文件可以还原redis数据库的数据，从而达到数据恢复的目的，借用一下《redis设计与实现》讲的这张图，途中数据库状态可以理解为redis内存中存储的数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822110411147.png)
既然RDB持久化的方式是生成RDB文件，那么这个RDB文件是怎么生成的呢？
RDB文件生成的方式有两种，一种是通过命令手动生成快照，还有一个是通过配置自动生成快照。下面我们来分别看看。

## 通过save和bgsava命令生成RDB文件
这两个命令都是可以直接运行的。

save命令，会阻塞服务器进程，只有当RDB文件生成成功才会接着响应服务端的其他命令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822111713522.png)
而bgsave ，既然有这个命令，肯定是和save有所不同的，bgsave 不会阻塞服务器进程，会创建一个子进程来创建RDB文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822112439225.png)
但是注意的是使用bgsave命令的时候，虽然是通过子进程生成RDB文件，不会阻塞服务进程，其他的命令可以执行，但是有几个命令是不能执行的。

```
save 
bgsave
bgrewriteaof
```
在bgsave 期间 服务器拒绝这三个命令，主要是方式线程间竞争产生问题。

## 通过配置文件自动生成RDB
初了手动执行这两个命令外，还可以在配置文件中配合参数，达到条件的时候就会自动的生成RDB
打开我们的配置文件redis.conf,找到如下图，这个是默认的配置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822131647656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

```
save 900 1 
表示在900秒内，如果发生了一次写操作，就触发bgsave命令生成RDB

同理 
save 300 10 在300秒内，发生了10次写操作，就触发bgsave

save 60 10000  在60秒内发生了10000次写操作，就触发bgsave 
```
上面的这些可以进行配置，可以看到默认的设置，如果短时间内发生大量的写操作就会自动的触发bgsave ,生成RDB文件， 防止数据丢失。

好了，上面虽然说达到这三个条件中的一个，redis就会自动的生成RDB文件，那系统是怎样控制，又是怎样识别是否满足条件呢？
原来啊，服务器维持了一个dirty 计数器，以及一个lastsave属性。
dirty 计数器记录着从上次save/bgave 到现在发生了多少次写操作，没进行一次写操作，计数器就加1
比如

```
set a  123
计数器dirty 加1 

set a 123 b 234 c 456
计数器dirty 加3
```
而lastsave 是unix时间戳，记录上次save或bgsave的时间。
有了这两个属性，就可以判断什么时候执行啦，redis服务器会周期性的执行serverCron函数,默认的话是每100毫秒执行一次。
这个serverCron 函数先通过当前时间减去lastsave 获取时间间隔。
如果dirty 大于 saveparm.chranges
并且时间间隔大于saveparm.seconds
那么就会触发bgsave 生成 RDB文件

其中saveparm.seconds 和saveparm.chranges  分别对应的是配置文件中设置的save 900 1等。

既然生成了RDB文件，我们只知道RDB是一个压缩的二进制文件，那RDB文件到底结构是什么样的呢？
我看了一下《redis 设计与实现》没有怎么看明白哈哈，感兴趣的可以去看看。

RDB方式就讲到这里了，记住RDB方式，是定时的执行bgsave 命令生成RDB文件保存在磁盘上实现持久化的。适合数据备份，用于数据恢复可能会丢失最近几分钟的数据。

# AOF 
全称是append only file.  AOF 持久化的方式是通过redis服务器记录保存下所有的写命令到AOF文件存放在磁盘上，实现持久化的，看下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822174320529.png)
怎样采用AOF的方式持久化呢？
打开我们的配置文件，在配置文件中找到appendonly 改成yes 就可以采用AOF的方式备份了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822185305746.png)
我们启动redis服务，为了测试方便，我们新启的一个redis服务，数据库中没有任何key
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822185520265.png)
我们看看appendonly.aof 也是没有任何东西的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822185617470.png)
现在我们存入一个key
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822185725947.png)
然后我们来看下aof文件的内容：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822185741824.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
可以看到，初了一些$3等一些特殊符号外我们可以看到我们执行的命令。

```
select 0
set a bbb
```
但是我们一些读操作的就不会记录。由此可见，AOF 持久化就是将所有的写操作存入AOF文件中，当数据恢复的时候，执行AOF文件中的命令就可以获取数据了。

我们接着来看那，我向数据库中先加一个key b ,然后删除key a .
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822190344226.png)
好了，现在我们再来看看aof 文件中是什么情况。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082219043083.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
可以看到命令有：

```
select 0
set a bbb
set b ddd
del a
```
所以aof 文件中包含了这四条命令，到这大家有没有发现一个问题，如果我重复的对某一个key值进行操作，那么aof文件中就会记录所有的操作命令，但是实际上只有最后一次操作才是有效的，那这个aof文件中是不是就有很多冗余的数据呢？

实际上是这样的，那怎么解决这个问题呢？这里就要提到一个命令啦

```
BGREWRITEAOF
```
和RDB中的save 以及bgsave 是类似的。不过bgrewriteaof 命令的作用是重写aof文件，为什么要重写呢，就是为了解决aof文件中冗余的问题。
我们先来手动执行一下这个命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822191203171.png)
然后看看aof 文件中的内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822191234835.png)
可以看到命令变成了

```
select 0
set b ddd
```
重写之后，aof的文件里的命令就是有效的啦，但是我们总不能自己手动执行bgrewriteaof 命令吧，那我们在哪里配置呢?
在redis.conf 配置文件中有这两个参数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822193916987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
 auto-aof-rewrite-percentage 100
 当Aof log增长超过指定比例时，重写log file， 设置为0表示不自动重写Aof 日志，重写是为了使aof体积保持最小，而确保保存最完整的数据。

auto-aof-rewrite-min-size 64mb

触发aof rewrite的最小文件尺寸

也就是说在实际应用中，如果开启了aof 备份，可以设置这两个参数来重写AOF文件。


好了上面说了那么多，那redis服务怎么通过aof文件来恢复数据的呢？
其实很简单，就是将aof 文件中的命令一条一条的读取出来执行。看下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822194539918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

最后再说一点嘿嘿。
我们在配置文件中同时启用了RDB 和AOF ,那么服务启动的时候，会在用那个文件来回复数据呢？
看下面这张图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822194804742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
可以看到如果启动aof ,就会采用aof 文件来回复数据，这是为什么呢，因为AOF 文件更新的频率更高，模式一秒中一次，所以用AOF 恢复的数据更加准确。

好了，只有这么多了哈哈，推荐大家看看《redis 设计与实现》也不建议大家从头往后看，调感兴趣的看吧，毕竟里面还是有很多原理对我们而言也不是一定非得弄懂的，大概了解一下就行，我比较懒哈哈。
