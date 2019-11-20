layout: post
title: 八、redis集群
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
  - 集群
date: 2019-09-03 20:21:09
---

# 前言
前面写完了 Redis 的主从复制、哨兵模式、Redis 持久化方式。这篇文章开始写 Redis 集群啦。

我们项目中使用 Redis 一般都不是使用单台 Redis 提供服务，除非是很小的项目，不过很小的项目也没有必要使用Redis了。所以一般使用 Redis 都会配上 Redis主从备（就是前面将的主从复制），配上哨兵模式实现故障转移。更大的项目就搭建一个 Redis 集群。

但是呢，我们大多数在项目中即使使用了 Redis ，都是直接使用的，框架什么的都已经被前面的人搭建好了。所以我们在使用的时候其实就是配置的时候有些不同，代码中使用起来和单机的 Redis 是没有什么区别的。所以并不能说我们对 Redis 集群有多了解。

所以接下来，就让我们来了解一下 Redis 集群吧，手把手搭建一个 Redis 集群 ，让我们对 Redis 集群有更多的了解 。

# 什么是 Redis 集群 
进入正文啦，既然学习 Redis 集群，那什么是 Redis 集群呢？

**Redis 集群是 Redis 提供的分布式数据库方案，集群通过分片( sharding )来实现数据共享，并提供复制和故障转移。**

可以说上面这句话是对 redis 集群的高度概括了，redis 集群提供分布式数据库方案，前面我们将的主从复制和哨兵模式可以知道，只会有一个主服务器( master )。主从复制，只会有一个 master ，可以有多个 slave。而哨兵模式是在主从复制的基础上，发现 master 挂掉，会自动的将其中一个 salve 升级成 master 。但是最终结果还是只有一个 master。所以如果系统很大，对Redis 写操作的压力就会很大，所以才出现的集群的模式。集群模式可以有多个 master 。来看下面集群模式的图(下面的图不是最终版，便于理解的。后文有最终版的)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190828163053884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
手动画的可能有点丑，但是大概的意思就是这样啦。单个节点就是我们之前了解的主从复制的一主多从(图中是一主两从)，加上哨兵模式，来监听节点的 master 是否正常。 那集群就是多个节点组成的，多个节点的 master 数据共享，横向分担单个节点 master 的压力。那从上图可以看出 哨兵模式 其实是 集群模式的一个子集，集群模式是哨兵模式的一个拓展。

# Redis 集群有什么好处，用在哪些场景
上面讲了什么是 Reids 集群，那为什么要使用 Redis 集群呢，不直接使用哨兵模式呢？使用 Redis 集群有什么好处呢？

要回答上面这个问题，其实在上一节已经差不多介绍了，集群模式是哨兵模式的一种拓展，既然是拓展，当时是因为哨兵模式不能满足需求才会产生的。
 
在没有Redis 集群的时候，人们使用哨兵模式，所有的数据都存在 master 上面，master 的压力越来越大，垂直扩容再多的 salve 已经不能分担 master 的压力的，因为所有的写操作集中都集中在 master 上。所以人们就想到了水平扩容，就是搭建多个 master 节点。客户端进行分片，手动的控制访问其中某个节点。但是这几个节点之间的数据是不共享的。 并且如果增加一个节点，需要手动的将数据进行迁移，维护起来很麻烦。所以才产生了 Redis 集群。

所以 Redis 集群有什么好处，就是进一步提升 Redis 性能，分布式部署实现高可用性，更加的稳定。当然还包含主从复制的数据热备份以及哨兵模式的故障转移等有点啦。

那 Redis 集群用在哪些场景呢？

其实我感觉一般较大的项目使用了 redis 的话，都会使用 redis 集群。毕竟在部署的时候先做好充分的拓展准备，比到时候项目出现瓶颈再去拓展成本就要小太多了。并且 Redis 是轻量级的，采用 redis 集群，也许在项目初期根本就用不上多个节点，单个节点就够用，多节点造成浪费。但是其实我们启动多个节点没有用到的话，节点所占用的内存和CPU 是非常小的。所以建议一般项目使用 Redis的话，尽量使用 Redis 集群吧。


# 集群的主从复制和故障转移
Redis 集群的主从复制，其实和单机的主从复制是一样的。前面 Redis 集群结构图可以看到。单个节点中有一个 master 和多个 slave 。这些 slave 会自动的同步 master中的数据。注意的是，这些 salve 只会同步 所属的 master 中的数据，集群中其他的 master 数据是不会同步的。

同样的 ，当个节点中可以配置多个哨兵，来监控这个节点中的master 是否下线了，如果下线了就会将这个节点的slave 选择一个升级成 master 并继承之前 master 的分片，继续工作。

但是其实啊，在集群模式中，并没有配置哨兵，我们也能实现故障的自动转移。其实真正的集群的图是这样的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190828175641528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
如图可以看到并没有为每个节点配置 sentinel 。那怎么实现对 master 的监听，实现故障的自动转移呢？

我们在讲哨兵模式的时候说过，其实哨兵也是一种特殊的 redis 服务对吧。我们master 是通过 ` redis-server` 启动的。我们哨兵是通过 ` redis-sentinel`启动的。然后哨兵的作用就是定期的给 master 发送 `ping `检测 master 是否下线，然后通过选举的方式选择 slave 升级成 master 
那放在集群中可以发现，哨兵的这些工作，完全可以交给master 来做。之前单个节点，master 做不了才交给 sentinel 的。现在有多个 master ，当然就可以用 master 来代替salve 的工作啦
在集群中，每个节点的master 定期的向其他节点 master 发送 `ping `命令，如果没有收到`pong` 响应，则会认为主观下线，并将这个消息发送给其他的 master。其他的 master 在接收到这个消息后就保存起来。当某个节点的 master 收到 半数以上的消息认为这个节点主观下线后，就会判定这个节点客观下线。并将这个节点客观下线的消息通知给其他的master。
这个客观节点下线后，其他的  master 节点 就会选举 下线的master中的 slave 一个变成 新的master 继续工作。从而实现故障自动转移。这个选举过程和哨兵模式中是一样的，只不过是 master 代替了 sentinel 的工作。
# 搭建一个 Redis 集群的实例
好接下来让我们一起来搭建一个集群模式吧，因为我只有一台服务器，所以我集群就搭建在一台服务器上，在实际项目中肯定是多台服务器搭建集群的。但是搭建的方式都是一样的。这里我将两种集群的搭建方式，第一种是手动的搭建集群，手动分片，这种方便我们对集群有更多的了解，第二种的话借助 Redis 自带的辅助工具来搭建集群，方便快捷。
## 方式一
好了，让我们开始吧，在开始之前，我们不用觉得搭建集群很麻烦，其实一样的是修改redis.conf配置文件中的内容。只需要将配置文件中集群模式打开就可以了。如下：
```
cluster-enabled yes
```
是不是很简单，还是不说废话的，搞起来。
### 准备工作
先创建一个 cluster 目录，然后在 cluster 目录下创建6个文件夹。因为的演示中我的cluster 目录已经占用了，我就再创建是cluster2 目录。

```
cd  /usr/local/redis/etc
mkdir cluster2
cd  cluster2
mkdir 8000
mkdir 8001
mkdir 8002
mkdir 8003
mkdir 8004
mkdir 8005
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829142130721.png)
然后将redis.conf文件copy到这六个文件中。

```
cp ~/workSpace/redis-4.0.9/redis.conf ./8000/redis.conf
cp ~/workSpace/redis-4.0.9/redis.conf ./8001/redis.conf
cp ~/workSpace/redis-4.0.9/redis.conf ./8002/redis.conf
cp ~/workSpace/redis-4.0.9/redis.conf ./8003/redis.conf
cp ~/workSpace/redis-4.0.9/redis.conf ./8004/redis.conf
cp ~/workSpace/redis-4.0.9/redis.conf ./8005/redis.conf
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829142604241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
### 修改配置文件
准备工作做好了，现在我们来修改这六个目录下的配置文件吧。都将开启集群模式。

```
vim ./8000/redis.conf
```
修改配置文件如下，我们为了方便就先不进行太复杂的配置，我们就配置搭建集群最小力度修改。

```
port 8000
daemonize yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829143706226.png)
上面修改的是8000这个目录下的redis.conf ，接下来我们按部就班的把其他几个文件中的配置也进行修改。和上面的基本一样。这里就不一一写出来了。贴一个8002的修改，大家就一目了然了。

```
port 8002
daemonize yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

### 启动
修改好这几个配置文件后，然后就启动这些服务

```
cd /usr/local/redis/etc/cluster2/8000/
redis-server ./redis.conf 

cd /usr/local/redis/etc/cluster2/8001/
redis-server ./redis.conf 

cd /usr/local/redis/etc/cluster2/8002/
redis-server ./redis.conf 

cd /usr/local/redis/etc/cluster2/8003/
redis-server ./redis.conf 

cd /usr/local/redis/etc/cluster2/8004/
redis-server ./redis.conf 

cd /usr/local/redis/etc/cluster2/8005/
redis-server ./redis.conf 
```

如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829202739916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
上图可以看到，会先进入到对应的文件夹，然后启动redis服务，为什么要先进入对应的文件夹呢？因为我们配置了`cluster-config-file nodes.conf` 这个会在你启动目录下生成一个node.conf 文件，用来存放当前节点信息。所以你不先进入对应目录的话，很很可能就启动不成功哟。下图可以看到生成了 node.conf 以及里面的内容。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829203246860.png)
我们在来通过线程的方式查看一下

```
ps -ef |grep redis
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829203446966.png)
可以发现我们启动的6个节点都已经启动了，并且后面都带有 cluster 的标识。和 6379 单机模式有区别。

### 节点互通
到此准备工作算是真正的做完了，我们启动了六个节点，但是现在这六个节点是相互独立的，没有任何关联，那我们怎么将它们关联起来呢？
我们先用客户端进入到8000节点。查看一下节点信息

```
redis-cli -p 8000
cluster nodes
```
可以看到只有自身这个节点的信息。现在我们和其他节点建立连接

```
cluster meet  ip  port
```
下图可以看我们和8001节点建立连接，这个时候再查看nodes节点信息，发现就有两个节点信息啦，说明这个集群中现在存在了8000 和8001 两个节点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830150256224.png)
我们现在把8002 节点也加进来。这样原本的各自独立的节点就在同一个集群中啦，大概就是下面这张图（有点丑，将就看）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902113443323.gif)
### 槽指派
上面已经进行了节点互通了，多个节点在同一个集群中了，那我们是不是就可以使用集群了呢？
其实不行，我们来看一下`cluster info`![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902131512152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
发现 cluster_status 还是fail。表明还是不可用的。因为我们还没有进行卡槽的分派。
Redis 集群是通过分片的方式来保存数据库中的键值对的，集群整个数据库被分成16384个槽（slot）。也就是说所有数据的key都会映射到对应的 slot 中。只有当数据库中16384 个槽都在节点上有分派，集群才会上线，否则集群的状态就是 fail。
所以接下来开始槽指派吧

```
cluster addslots  slots[slots]
样例：
cluster addslots 0 1 2 
cluster addslots 0 1 2 ... 5000
cluster addslots {0..5000}
```
网上说的可以支持范围分配的，但是我电脑上试了 N 种方法都不行，我的 redis 版本是4.0.9 的。最后没办法，写了一个脚本跑的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902180531659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
如果大家也不行的话，可以用脚本跑吧。

```
start=$1
end=$2
port=$3
for slot in `seq ${start} ${end}`
do
           echo "slot:${slot}"
              redis-cli -p ${port} cluster addslots ${slot}
      done
~              
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902181223836.png)
然后执行：

```
sh add-slots.sh 0 5000 8000
sh add-slots.sh 5001 10000 8001
sh add-slots.sh 10000 16384 8002
```
这样就可以把16384个卡槽分配到三个节点上啦，如果有多个节点，自己调整分配哈，我这里是以三个节点为例的。

卡槽分配完成以后，我们在来看看 cluster  的状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902181442458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
发现 cluster_status 变成 ok 了。说明我们的卡槽分配是成功的。
### 验证
那现在集群上线了，是不是就可以用了呢?
答案是是的，但是配到现在为止还有点瑕疵，瑕疵我们待会再说，先来看看集群能不能操作命令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903083508917.png)
可以看到，进行槽指派之后是可以进行正常的操作的，这里的`set a 123`提示我移动到8002端口执行。因为a 对应的卡槽为15495.
这里有一个命令和可以查看key值在哪一个卡槽，从而属于哪一个节点。

```
127.0.0.1:8000> CLUSTER KEYSLOT a
(integer) 15495
127.0.0.1:8000> 
```
那假如我就想set a 呢，难道要先切换到8002端口，那岂不是很麻烦，还有另一种方法：

```
redis-cli -c -p 8000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903084627953.png)
可以看到这样启动客户端，会自动的将数据存入到对应的节点上，并切换到这个节点，并且之前我在8000 端口上`set data 123`,我现在在8002端口上`get data ` 会自动的找到key值并切换到8000端口上，这样在客户端就感觉这三个节点是一个整体啦，是不是很方便。
### 配置主从
前面为止，集群模式已经搭建好了，但是呢前文说的还有点瑕疵，现在就来说说，我们现在搭建的集群只有三个主节点，任何一个主节点挂掉了，就会导致集群不可用，因为集群可用的标志是 16384 个卡槽全部都分配到可用的节点上。所以我们现在搭建的集群还是不稳定的。所以为了解决这个问题，我们需要为每一个主节点配置一个从节点。
从节点的作用是数据热备份和当主节点出现故障时可以替代主节点进行工作。

好了，我们来搭建主从吧，前文中我们创建了6个节点，还有三个没用，其实就是用来搭建主从的哈哈。还没有用到的这三个节点 和搭建好的这个集群是独立的现在。
第一步，将这三个节点加入到集群中来。

```
cluster meet  127.0.0.1 8003
cluster meet  127.0.0.1 8004
cluster meet  127.0.0.1 8005
```
第二步，查找主节点的 nodesId。通过 `cluster nodes `可以查到到集群中所有节点的信息，第一列就是每个节点的nodesId.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903091254964.png)
第三步，将从节点和主节点关联起来。

```
redis-cli -p 8003 cluster replicate 9a86d899be55a37d3ac1c8be6c342c7f59513076
redis-cli -p 8004 cluster replicate 03efbb8782a705d5978f5c1b14d4dcba14a167e8
redis-cli -p 8005 cluster replicate 01f6c1888b7a103c10fffe9cf6be8a5fff8ae985
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903092754288.png)
我们再来看看节点信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903093105713.png)
可以看到从节点已经搭建成功啦，那到底有没有成功呢，我们再来试试，我们手动模拟主节点8000故障了。我们用`shutdown`可以关闭当前节点的服务。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903093408249.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
看一下进程确定8000已经关闭了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903093544690.png)
我们现在进入8001端口看一下，`get data`可以看到切换到8003端口上了，8003端口是8000端口的从节点，现在8000端口挂掉了，8003端口会代替8000端口继续工作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903100959947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
我们看一下`cluster nodes ` 发现8000端口节点是 fail 的，8003端口升级成master了。我们现在修复了8000端口问题，将其重启起来看看。

```
quellanan@quellanan-Lenovo-G400:/usr/local/redis/etc/cluster2/8000$ redis-server ./redis.conf 
26401:C 03 Sep 10:13:31.777 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
26401:C 03 Sep 10:13:31.777 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=26401, just started
26401:C 03 Sep 10:13:31.777 # Configuration loaded

```
然后看看节点信息，发现8003端口变成了主节点，8000端口变成了8003端口的从节点。这和我们之前哨兵模式是一样的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903101643703.png)
好啦，到此为止，一个集群算是真正的搭建好啦。一步一步的来，不难，就几个命令而已，可能我写得比较啰嗦嘿嘿。

## 方式二
### 准备工作
第二种方法搭建集群就简单讲啦，准备工作和启动都是一样的，只是不用我们自己进行节点互通和分配卡槽啦。如下图，我已经启动 7000~7005 六个节点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903103718865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
### 安装软件

```
yum install ruby
yum install rubygems
gem install redis 
```
我这里已经安装好了，就不演示啦。
### 配置
执行以下命令，就会自动的帮我们进行节点互通，分配卡槽以及设置从节点。
```
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

```
**特别提醒，这里的IP用主机的IP，如果使用127.0.0.1的话，在我们代码中访问会出错，我也是在项目中使用的时候碰到的**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903112941564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
上面就已经搭建好集群啦，简单吧。
### 验证
我们现在简单验证一下，进入7000节点；

```
redis-cli -c -p 7000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019090311331399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
可以看到集群是上线状态，可以正常使用啦，我们简单操作一波
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903113444307.png)
好啦，集群搭建的实例就说这么多吧，两种方式，第一种是原生的，第二种是借助工具的，效果是一样的。还有一个内容就是重新分片。这个用到的不多，我们前期在搭建集群的时候先预留多个节点就好，不然后面要扩容，就需要用到重新分片，感兴趣的可以在讨论区讨论下吧，也不难，这里就不写了(文章太长啦)。
# 在项目中使用集群
在项目中使用集群，我这里就简单的给一个样例给大家参考。
## 创建一个springboot 项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903171721569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
基本上一直next 就好了。

## 修改pom.xml文件

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!--Redis使用starter-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
      
        <!--注解日志/get/set-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
        
    </dependencies>
```

## 修改application.yml

```
server:
  port: 9090

# redis 集群配置
spring:
  redis:
    cluster:
      nodes: 192.168.252.53:7000,192.168.252.53:7001,192.168.252.53:7002,192.168.252.53:7003,192.168.252.53:7004,192.168.252.53:7005
      timeout: 6000ms # 连接池超时时间（毫秒）
    # 密码没有可以不填
    password:
    database: 0 # 数据库索引
    lettuce:
      pool:
        max-active: 8 # 连接池最大活跃连接数（使用负值表示没有限制）
        max-idle: 8 # 连接池最大空闲连接数
        max-wait: -1ms # 连接池最大阻塞等待时间 毫秒（使用负值表示没有限制）
        min-idle: 0  #最小空闲连接数
```

## 配置dao层
这个样例是我写的另一篇博客样例改编过来的，大家不知道项目结构的可以看一下我这篇博客中样例的项目结构。
[Redis在SpringBoot中使用案例](https://blog.csdn.net/qq_27790011/article/details/98344732)
创建dao 包，创建一个User 类,这里使用了lombok提供的@Getter 和@Setter 非常方便，代码看着也很简洁。

```
@Getter
@Setter
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private Long id;
    private String userName;
    private String password;
    private String email;
    private String nickname;
    private String regTime;

    public User(String email, String nickname, String password, String userName, String regTime) {
        super();
        this.email = email;
        this.nickname = nickname;
        this.password = password;
        this.userName = userName;
        this.regTime = regTime;
    }
}
```
## 创建service层
创建一个service 包，创建一个RedisService类，代码如下：

```
@Service
@Slf4j
public class RedisService {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
  
    public boolean setUserBystringRedisTemplate(User user){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        ops.set(user.getNickname(),JSONObject.fromObject(user).toString());
        return true;
    }
    
    public String getUserBystringRedisTemplate(String name){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        return JSONObject.fromObject(ops.get(name)).toString();
    }

    public boolean setString(String key,String value){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        ops.set(key,value);
        return true;
    }
    
    public String getString(String key){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        return (String)ops.get(key);
    }
}
```

## 创建Controller层
创建一个controller 包，创建一个RedisController类代码如下：

```

@RestController
public class RedisController {

    @Autowired
    private RedisService redisService;

    @RequestMapping("/setUser")
    public String setUser(){
        User user=new User("aa@qq.com","quellan","123456","朱",new Date().getTime()+"");
        redisService.setUserBystringRedisTemplate(user);
        return "添加成功";
    }
   
    @RequestMapping("/getUserByStringRedisTemplate")
    public String  getUserByStringRedisTemplate(){
        String name="quellan";
        return redisService.getUserBystringRedisTemplate(name);
    }
    
    @RequestMapping("/setString")
    public String setString(String key ,String value){
        redisService.setString(key,value);
        return "添加成功";
    }
    
    @RequestMapping("/getString")
    public String setString(String key){
        return redisService.getString(key);
    }
}
```

## 测试
到此为止，代码就已经写完啦，我们来启动项目测试一下。
项目启动成功之后，我们调接口看看

```
http://localhost:9090/setString?key=a&value=123
http://localhost:9090/setString?key=b&value=1qaz
http://localhost:9090/setString?key=data&value=wsxcd
http://localhost:9090/getString?key=a
http://localhost:9090/getString?key=data
http://localhost:9090/setUser
http://localhost:9090/getUser
```
我就截图看其中两个
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903174849583.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903174857678.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903174943654.png)
界面上没有问题啦，我们再用客户端看一下。

```
redis-cli -c -p 7000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019090317530193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
数据在集群中正常的读取是没有问题哒。

好了，在项目中简单使用集群就讲到这里啦，源代码我github上了，感兴趣的同学可以看下。
[demo源代码](https://github.com/QuellanAn/SpringBootLearning)

总算写完了，可能写的不是很好，大家有疑问的，可以提出来，我知道的尽量给大家解答，谢谢大家！

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)










