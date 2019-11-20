layout: post
title: 四、docker 仓库(让我们的镜像有处可存)
author: QuellanAn
categories: 
  - docker
tags:
  - docker
  - linux
---
# 前言
前面讲完了docker 镜像和容器，以及通过Dockerfile 定制属于我们自己的镜像，那那现在就是需要将我们自己定制的镜像存放到仓库中供他们使用。这一套流程才算是正式走完了。从获取镜像，操作镜像容器，定制镜像，上传镜像。会了这些，也算是docker 正式入门了。

# 上传到共有仓库
docker 官网有一个共有的仓库，大家应该都知道，和github 类似。dockehub可以管理你自己的镜像。我们需要创建一个账号用来管理。

官网：https://hub.docker.com/

我们创建好账号后，就可以在我们本机的电脑上登录到官网了。
```
docker login 用户名 网址
```
网址可以不填，默认的就是去登录官网，登录官网之后就可以上传我们自己的镜像了
```
 docker push [OPTIONS] NAME[:TAG]
 
 eg:
 docker push quellanan/hello:1.0.0
```
![file](https://img-blog.csdnimg.cn/20191105083653418.jpeg)

我这截图是上传过一次，再上传的时候提示已经存在，说明是上传成功的。
我们可以查看一下：
```
docker search quellanan
```
![file](https://img-blog.csdnimg.cn/20191105083654133.jpeg)

# 私有仓库
docker 官方提供了一个私用仓库的镜像，我们可以直接使用。docker-registry.

## 下载
我们先下载registry 镜像
```
docker pull registry
```
![file](https://img-blog.csdnimg.cn/20191105083654382.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## 容器运行
```
docker run -d -p 5000:5000 --restart=always --name registry registry
```
![file](https://img-blog.csdnimg.cn/20191105083654175.jpeg)

到现在我们私有仓库已经有了，现在我们如何将自己本地镜像上传私有仓库呢？

## 上传
首先我们需要使用docker tag 将镜像重命名，前缀需要和私用仓库一致，才能上传成功。
```
docker tag java:8 127.0.0.1:5000/java:8
docker push 127.0.0.1:5000/java:8 
```
![file](https://img-blog.csdnimg.cn/20191105083654527.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

通过下面命令查看是否成功
```
docker push 127.0.0.1:5000/java:8
```
![file](https://img-blog.csdnimg.cn/20191105083653172.jpeg)

上面证明我们已经将镜像上传到我们的私有仓库了。

## 下载
那现在我们先将本地的镜像删除掉，然后从私服上下载镜像，看是否能够下载下来。
```
 docker image rm 127.0.0.1:5000/java:8
 
 docker pull 127.0.0.1:5000/java:8
```
![file](https://img-blog.csdnimg.cn/20191105083655119.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

证明我们创建的私服是可以用的，但是有没有感觉有点别扭，不能想dockerhub 那样直观的查看我们私有仓库的镜像，没有可视化界面。所以接下来我们用另一个镜像来搭建我们私有仓库。


# Nexus3
Nexus 是管理maven 的jar 包工具，Nexus3 支持对镜像的管理。

## 下载
我们先下载nexus3的镜像
```
docker pull sonatype/nexus3
```

## 启动
下载成功后，我们来启动对应的容器。
```
docker run -d --name nexus3 --restart=always -p 8081:8081 -p 8082:8082 -p 8083:8083 --mount src=nexus-data,target=/nexus-data sonatype/nexus3
```
这里说明一下为什么要启动三个端口。8082是私有仓库，不启动的话，好像我们本地根本连不上去，一直报超时。8083为后面代理dockerhub 做准备。
![file](https://img-blog.csdnimg.cn/20191105083653835.jpeg)

容器启动之后我们在页面上访问
```
192.168.252.53:8081
```
可以看到我们的 nexus3的镜像已经启动成了，我们需要登录才能进行配置。网上说的用户名为admin，密码为admin123 我试了发现登录不上去。
![file](https://img-blog.csdnimg.cn/20191105083655644.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

然后看提示说密码存放在这个位置，所以我们进入到容器。查看我们的密码。
```
docker ps
docker exec  -it /bin/bash
cat /nexus-data/admin.password
```

![file](https://img-blog.csdnimg.cn/20191105083656432.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

找到密码后，我们在界面登录后，会让我们修改密码。
![file](https://img-blog.csdnimg.cn/20191105083654721.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## 配置
登录成功后，我们开始配置我们docker的私有仓库。选择Create Repostory
![file](https://img-blog.csdnimg.cn/2019110508365788.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

选择docker(hosted)
![file](https://img-blog.csdnimg.cn/20191105083656948.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

配置仓库名和端口
![file](https://img-blog.csdnimg.cn/20191105083657658.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

这些都配置好了，现在我们怎么使用这个私有仓库呢，我们在/etc/docker/daemon.json 文件中加上私有仓库的地址。
```
{
        "registry-mirrors": [
                "https://registry.docker-cn.com",
                "https://dockerhub.azk8s.cn"
        ],
        "insecure-registries":["192.168.252.53:8082","192.168.252.53:8083"]
}
```
registry-mirrors 是配置国内镜像，不需要的可以不配置。insecure-registries 就是设置我们自己的私有仓库地址。

重启
```
systemctl daemon-reload
systemctl restart docker
```

## 测试
现在我们来登录上我们私有仓库(密码我改成了admin123)
```
docker login -u admin -p admin123 192.168.252.53:8282
```
![file](https://img-blog.csdnimg.cn/20191105083657899.jpeg)

一样的我们打标签。
```
docker tag java:8 192.168.252.53:8082/java:8
```

上传
```
docker push 192.168.252.53:8082/java:8
```
![file](https://img-blog.csdnimg.cn/20191105083658156.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

可以看到我们已经将镜像上传的nexus 上了，我们现在在界面上看下。整个的界面就是这样的。
![file](https://img-blog.csdnimg.cn/201911050836583.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

说明我们用 nexus3 搭建的私有仓库是没有问题的。

# Nexus3 代理仓库
上面我们只是配置了docker(host)，这个相当于我们的私有仓库，但是我们现在使用docker login 我们自己的仓库，如果我们需要的镜像我们仓库没有，就会很麻烦，需要重新登录到共有仓库上下载下来，再上传到我们的私有仓库，那有没有办法可以一步到位呢？

下面我们就来操作一波。

## docker(proxy)
上面我们已经配置好了私有仓库的不用动，下面我们来配置代理仓库，
![file](https://img-blog.csdnimg.cn/20191105083658291.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

选择docker(proxy),name 自定义。主要的Proxy 这里需要注意一下。
```
https://registry-1.docker.io
```
![file](https://img-blog.csdnimg.cn/20191105083658972.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## docker(group)
端口设置8083
![file](https://img-blog.csdnimg.cn/20191105083659263.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

将代理的和个人仓库加到group中
![file](https://img-blog.csdnimg.cn/20191105083659513.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9xdWVsbGFuYW4uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

这样上面就配置好了。

# 番外

这篇算是马马虎虎的写完了吧，但总感觉不经如意，又不知道怎么修改，就先这样发出来吧，后续调整。

好了，就说这么多啦

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤


![file](https://img-blog.csdnimg.cn/20191015213334732.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)






