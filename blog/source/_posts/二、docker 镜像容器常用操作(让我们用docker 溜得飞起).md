layout: post
title: 二、docker 镜像容器常用操作(让我们用docker 溜得飞起)
author: QuellanAn
categories: 
  - docker
tags:
  - docker
  - linux
---

# 前言
上篇讲了我们如何安装docker，现在该我们一展拳脚的时候了。接下来让我们一起学习一下docker常见的操作，让我们能够会使用 docker。

# 基本概念
在讲使用之前，还是先将一下docker 的基本概念，毕竟上篇就讲了docker 的安装。一些基本的名词还是需要了解一下的。
docker 最重要的就是镜像和容器了，还有一个仓库。

那什么是docker 镜像呢？ 

docker 镜像就相当于一个 root 文件系统，不仅包含容器运行的程序和资源，还包含运行依赖的配置。但是镜像不包含任何动态的数据。

通俗的来讲就像是我们项目运行需要各种依赖和配置以及各种部署。然后我们将这些环境和程序都打包在一起，形成一个可以直接运行的包。就相当于是docker镜像，将所有需要的环境都集成在一起。在哪都可以运行。

docker 镜像是分层存储的。docker镜像在构建的时候是一层层构建的前一层是后一层的基础，使得镜像在复用、定制变得更加简单。也由于镜像是分层存储的，所以镜像显示的size 大小并不是实际占用的物理内存。因为有很多中间镜像都是公用的。所以实际占用的内存会比显示的size要小。

![file](https://img-blog.csdnimg.cn/20191022172419183.jpeg)

查看容器实际的占用的内存使用
```
docker  system df
```
![file](https://img-blog.csdnimg.cn/20191022172419421.jpeg)

现在知道镜像了，那镜像怎么使用呢？

那就是通过容器啦，容器和镜像的关系就像是 对象和实例的关系。也就是说根据镜像创建一个可以直接运行的容器。容器是镜像的具体体现，所以容器就有创建，启动，停止，删除等操作。

# 镜像的使用
好了，前面知道了什么是docker 镜像和容器，那现在就我们来看看怎么使用他们吧。

## 下载镜像
我们安装好docker 后，怎么获取镜像呢？和git 拉取一样也是使用pull.
```
docker pull 
```
详细的参数使用可以通过```docker pull --help ```来查看

比如我们现在下载一个nginx的景象
```
docker pull nginx
```
默认会下载latest 的镜像，表示下载最新的镜像。也可以下载稳定版本的，或者下载指定版本的。
```
docker pull nginx:stable

docker pull nginx:1.16
```
![file](https://img-blog.csdnimg.cn/20191022172419766.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

## 查询镜像
我们镜像下载下来了，我们怎么查看我们电脑上有哪些镜像呢？
其实上面我已经用了
```
docker image ls 
或者
docker images 两者的效果是一样的。
```
具体使用一样的可以使用```docker image --help```。我们接下来将我们常用的。
查询显示虚悬镜像
```
docker images -f dangling=true
```
虚悬镜像是没有作用的，占用内存空间，虚悬镜像怎么来呢？一般是我们下载镜像，依赖一些中间镜像，然后我们删除了下载的镜像，但是只是删除了上层镜像，依赖的镜像没有删除。这样没有依赖的中间镜像就成了虚悬镜像，是可以删除的。

删除虚悬镜像

```
docker image prune
```
其他的一些查询操作。
```
#列出中间层镜像
docker images -a
#列出部分镜像
docker images 仓库名

#过滤
docker images -f since=仓库名
docker images -f before=仓库名
```


## 删除镜像
我们现在知道怎么拉取镜像，以及在本地查看镜像，那我们想要删除镜像怎么删除呢？
```
docker image rm 镜像id
```
我们可以通过镜像id 来删除镜像，并且不用完整的镜像id ,只要可以做唯一区分就好了。
![file](https://img-blog.csdnimg.cn/2019102217242068.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

除了通过镜像id 来删除镜像，还可以通过以下的几种方式来删除，更过的可以通过```docker image rm --help```来查看

```
# 删除所有仓库名为***的镜像
docker image rm $(docker images -q 仓库名)

# 删除仓库名在***之前的镜像
docker image rm $(docker images -q -f before=仓库名)
```

# 容器的使用
上面讲了镜像的获取查看删除操作，那我们怎么根据镜像来操作相关的容器呢？

## 创建和启动

前面说了镜像和容器的关系就像是对象和实例的关系。我们一般使用都是使用实例，一样的我们docker使用也是使用docker容器。
那我们怎么根据镜像来创建容器并使用它呢？
使用
```
docker run  
```
比如我们前面下载了那么多Nginx，我们现在启动你nginx 试试。
```
docker run -p 8080:80 nginx:stable
```
-p 是用来指定映射端口的，8080是我们设置访问那个端口，80 是Nginx本身的端口。也可以后台启动
```
docker run -d -p 8180:80 nginx:stable
```
设置容器name
```
docker run --name myNginx -d -p 8280:80 nginx:stable 
```
![file](https://img-blog.csdnimg.cn/20191022172420349.jpeg)

![file](https://img-blog.csdnimg.cn/20191022172419372.jpeg)

我们现在在浏览器上访问一下8080，8081，8082这几个端口，应该都可以访问的。
![file](https://img-blog.csdnimg.cn/20191022172420923.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

## 终止容器
上面的容器启动了，我们现在想要停止容器，我们该怎么做呢？
如果我们没有后台启动，想要终止容器的话，直接Ctrl C 就可以退出来。如果我们是后台启动的，我们就需要通过
```
docker container stop 容器id
```
![file](https://img-blog.csdnimg.cn/20191022172421213.jpeg)
可以看到，删除的时候一样的不需要完整的id ,只要可以唯一区分就可以。

## 查看容器
其实上面已经用到了如何查询容器。
```
docker container ls
```
这个是查看正运行的容器。查看所有容器使用
```
docker containe la -a 
```
更多的命令可以查看
```
docker container ls --help
```
![file](https://img-blog.csdnimg.cn/20191022172421446.jpeg)
上图可以看到我已经停止了三个Nginx容器。用 -a 才会显示。 

## 重启容器
我们又想将关的容器重新启动，那怎么做
```
docker container start 容器id

#重启运行中的容器
docker container restart 容器id
```
![file](https://img-blog.csdnimg.cn/20191022172421768.jpeg)


## 删除容器
```
docker container rm 容器id

# 删除运行中的容器
docker container -f  容器id

# 删除所有没有运行的容器
docker container prune
```
![file](https://img-blog.csdnimg.cn/20191022172421990.jpeg)


# 番外
到此为止，我们常用的镜像和容器的操作就会使用啦。都是一些命令。忘记的可以--help 查看一下。

好了，就说这么多啦

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyYzc1MjZhNDhlMjBhZTU4YjhjMDE1Njg5MDE3NDYucG5n?x-oss-process=image/format,png)







