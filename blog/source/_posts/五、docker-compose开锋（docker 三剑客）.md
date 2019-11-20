layout: post
title: 五、docker-compose开锋（docker 三剑客）
author: QuellanAn
categories: 
  - docker
tags:
  - docker
  - linux
---
# 前言
终于写到docker-compose了，其实我最开始接触docker的时候，是因为一个开源项目需要用docker 环境和docke-compose 所以我最先接触的是docker-compse 后面才恶补的docker的一些基础知识。

可以看到docker-composer 和docker 有关系，但是你也了解docker-compose 的命令 简单的操作docker 容器。

说了这么多，还没有说docker-compose 有什么作用，为什么要使用docker-coompose。
其实我们都知道，在我们实际的项目中，一个项目一般都是前端服务端数据库都进行分离的。所以一个项目一般都是有多个镜像组成的。那怎么将这一组镜像管理起来呢？就是通过docker-compose 啦

docker-compose 中有两个重要呢的概念
服务(service ): 就是我们上面说的一个应用容器，仅仅负责真个项目的中的一部分，比如数据库mysql.

项目(project)：就是我们上面说你的项目啦，包含一组容器。

docker-compose 通过 docker-compose.yml 文件对这一组容器进行配置。 

好了，正式开始接触 docker-compose吧

# 安装
docker-compose 安装很简单，windos 版本的已经自带了。我们可以通过
```
docker-compose  -v
```
查看我们本机安装的docker-compose 版本。
Linux 安装也很简单。
在官网上也有：https://github.com/docker/compose/releases
```
sudo apt-get update

#安装最新的docke-ce
sudo apt-get install  docker-ce

# 下载最新的docker-compose
curl -L https://github.com/docker/compose/releases/download/1.25.0-rc4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 修改docker-compose 权限
chmod  x /usr/local/bin/docker-compose
```

卸载 docke-compose
```
sudo rm /usr/local/bin/docker-compose
```

## 本地安装
如果上面安装不行的话，或者报错，可以用下面的方式进行安装。

在上面的官网上找到对应版本的Assests 选择对应的文件下载。

![file](https://img-blog.csdnimg.cn/2019110815595186.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

下载下来后，我们放到 /usr/local/bin/  目录下。执行下面操作
```
#改名
sudo mv docker-compose-Linux-x86_64 ./docker-compose
#增加执行权限
sudo chmod  x /usr/local/bin/docker-compose
```
这样就和上面的效果是一样的啦，我们可以通过```docker-compose -v ```查看安装成功没有。


# docker-compose.yml
知道了docker-compose  那最重要的就是docker-compose.yml 文件啦，通过这个文件就可以管理项目的镜像了，那我们怎么写docker-compose.yml 文件呢？
官方提供了很多模版，我们按照模版来写就可以了。

## 主模版
```
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```
可以看到格式就是我们熟悉的yml 格式，和我们springboot 项目中配置是差不多的。
我们前面知道的一个项目是由一组服务组成的，也就是你对应文件中的services。webapp 就是我们为服务起的一个名字，image 对应的镜像名，ports 镜像暴露的端口，volumes 镜像的数据卷。可以看到，里面的命令和docker run 的指令是差不多的。

## depends_on
解决容器的依赖，表示一个容器依赖其他的其他容器，比如说
```
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
    depends_on: 
      - redis
      - mysql

  redis:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"

  mysql:
    image: mysql:latest
    restart: always
    ports:
      - "3306:3306"
```
还是上面的例子，只不过我多加了两个 service 。表示这个项目中用到了mysql  和redis  并且在webapp 中使用depends_on 表示redis 和mysql 先webapp 启动。

更多的模版，大家用的时候可以参考官网上就可以了我感觉。知道是什么意思就可以，不用都记下来。

https://yeasy.gitbooks.io/docker_practice/content/compose/compose_file.html

# docker-compose 指令

我们可以通过帮助指令来查看docker-compose 怎么使用。
```
docker-compose --help 
```

基本语法格式：
```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

我这里也就将一下常见的，因为通过```--help```都可以查到。

## docker-compose config
用于检查我们的docker-compose.yml 文件的内容格式是否正确，在我们运行之前先检测一下比较好。

## docker-compose up
用来启动项目，比如我们现在有一个docker-compose.yml 文件，那我们进入到这个文件目录，执行```docker-compose up```就可以将项目依赖的镜像下载下来，并启动相应的容器服务。整个项目都启动起来了，直接使用就好了，可谓是相当强大了。

docker-compose up -d  表示后台启动。

## docker-compose down 
和 up 对应，用来停止我们的项目。

## docker-compose restart 
重启我们项目

其他的也不说了，可以查看官网：

https://docs.docker.com/compose/reference/overview/



# demo 
光说不练假把式，我们上面说的一堆基础的知识，还是需要我们实践才行，不然我们不会有什么实质性的收获。所以接下来我们就搭建一个简单的demo。

我们还是用前面的的hello的项目，我们对项目进行一些修改，增加 redis。
这里我就不具体的讲啦，有不会的可以看我这篇文章，写的很简单明了：

[三、Redis在SpringBoot中使用案例](https://blog.csdn.net/qq_27790011/article/details/98344732)

我们这里先在在pom.xml 中增加redis 依赖：
```
<!--Redis使用starter-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

在application.properties 中增加redis 配置
```
#配置redis
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=192.168.252.53
# Redis服务器连接端口
spring.redis.port=6389  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0

```

在controller 包中创建一个redisController 类
```
@RestController
@RequestMapping("/redis")
@Slf4j
public class RedisController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping(value = "/add",method = RequestMethod.GET)
    public String add(@RequestParam(value="key")String key,@RequestParam(value = "value") String value){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        ops.set(key,value);
        return "success";
    }

    @RequestMapping(value = "/get",method = RequestMethod.GET)
    public String get(@RequestParam(value = "key")String key){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        return (String) ops.get(key);
    }
    
}
```

好了，我们将项目打包成镜像，至于怎么打包成镜像上一篇我已经讲了，不会的可以查看:

[三、DockerFile 定制属于自己的专属镜像](https://blog.csdn.net/qq_27790011/article/details/102729974#docker_162)

## 准备我们的redis.conf
我们使用 redis 镜像，但是我们不想使用默认的配置，想要使用自己的配置启动redis。所以我们来复制一份redis.conf 。我就修改了
```
#设置redis 可以远程访问
bind 0.0.0.0
#后台启动
daemonize yes
```
![file](https://img-blog.csdnimg.cn/20191108155950758.jpeg)
redis.conf 放在我们上图的redis目录下。

## docker-compose.yml
我们来编写docker-compose.yml ，直接套用上面的模版。
```

version: "3"
  
services:
  webapp:
    image: quellanan/hello:1.0.0
    ports:
      - "9000:9000"
    volumes:
      - "/data"
    depends_on:
      - redis

  redis:
    image: redis:latest
    restart: always
    ports:
      - "6389:6379"
    volumes:
      - /redis/redis.conf:/etc/redis/redis.conf
    command: redis-server /etc/redis/redis.conf

```

可以看到基本上是根据模版来的，指定我们镜像，端口，数据卷。
这里webapp 的没有什么好说的 ，上面都说了，一看就能懂，说一下redis的。
images 指定的镜像为redis:latest ，如果你本地没有这个镜像，就会从官网上下载。
restart:always 表示自动重启。
```
 ports:"6389:6379"
```
表示镜像启动redis容器的端口是6379，映射到服务器的6389 端口，所以我们在项目配置的redis 端口应该是6389.
```
volumes: /redis/redis.conf:/etc/redis/redis.conf
```
表示将 ./redis/redis.conf 文件加载到 容器中的 /etc/redis/redis.conf 位置。
说明第一个路径是相对路径，第二个路径是绝对路径。

```
command: redis-server /etc/redis/redis.conf
```
表示在启动redis 容器的时候会执行的命令。这样就可以实现启动redis镜像加载我们自己的配置文件了。

## docker-compose up

准备工作都做好了，开始我们大展拳脚，哈哈，其实不然，我们准备工作做好了，就已经成功一大半了，我们接下来要做的就是 就是通过docker-compose 启动镜像。我们直接在存放docker-compose.yml 目录下执行：
```
docker-compose up
```
![file](https://img-blog.csdnimg.cn/20191108155951768.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)
这样我们就启动成功了。

如果想后台启动的话输入：
```
docker-compose up -d 
```

## 测试
我们项目启动，现在来测试一下到底成功没有。
```
http://192.168.252.53:9000/
```
这个是测试项目是正常启动了。
![file](https://img-blog.csdnimg.cn/20191108155951218.jpeg)
我们接下来看看我们配置的redis 有没有成功。

```
http://192.168.252.53:9000/redis/add?key=a&value=123qaz
http://192.168.252.53:9000/redis/get?key=a
```
![file](https://img-blog.csdnimg.cn/20191108155952431.jpeg)
![file](https://img-blog.csdnimg.cn/20191108155952729.jpeg)

可以看到界面上接口没有问题了，redis已经已经生效了，我们还不太确定，可以去服务器上看下。
![file](https://img-blog.csdnimg.cn/20191108155952359.jpeg)。本地没有装redis ，我们可以进入到redis容器中去查看。
操作如下：
先通过```docker ps ```查看redis 容器id
然后通过下面命令进入容器。
```
docker exec -it 容器id  /bin/bash
```
最后连接redis
```
redis-cli 
```
![file](https://img-blog.csdnimg.cn/20191108155953281.jpeg)


# 番外
好了，就说这么多啦

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤


![file](https://img-blog.csdnimg.cn/20191015213334732.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)













