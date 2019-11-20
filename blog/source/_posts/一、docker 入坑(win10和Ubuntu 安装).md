layout: post
title: 一、docker 入坑(win10和Ubuntu 安装)
author: QuellanAn
categories: 
  - docker
tags:
  - docker
  - linux
date: 2019-10-11 16:12:02
---

# 前言
终究还是绕不过去了，要学的知识真的是太多了，好在我们还有时间，docker 之前只闻其声，不曾真正的接触过，现在docker 越来越火，很多公司也都开始使用了。所以对于我们程序员而言，又得修炼一项必备技能了。
所以让我们勇敢的踏出第一步，学海无涯，让我们一步一个脚印。从安装开始讲起吧。

# windows10安装
参考：https://yeasy.gitbooks.io/docker_practice/content/install/windows.html

## 开启Hyper-V
win10 安装需要先开启 Hyper-V。
控制面板-->所有控制面板项-->程序和功能-->启用或关闭 Windows 功能
![file](https://img-blog.csdnimg.cn/20191015213331400.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

## 下载安装
然后下载安装程序：
[Stable](https://download.docker.com/win/stable/Docker Desktop Installer.exe)
 或者
[Edge](https://download.docker.com/win/edge/Docker Desktop Installer.exe)

下载下来之后直接双击运行完成后的截图。
![file](https://img-blog.csdnimg.cn/20191015213331663.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

点击close and log out 会重启电脑。

## 设置
重启完电脑后，在我们的导航栏会有docker 的图标，点击图标，选择setting ,genneral 勾选最后一个选项。

![file](https://img-blog.csdnimg.cn/2019101521333212.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

设置镜像，我们使用国内的镜像，会让我们下载速度提升，在setting的daemon中设置
```
https://registry.docker-cn.com
https://dockerhub.azk8s.cn
```

![file](https://img-blog.csdnimg.cn/20191015213332342.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

## 测试
在cmd 控制台查看docker  版本
```
docker version
```
![file](https://img-blog.csdnimg.cn/20191015213332615.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

运行hello-world 镜像
```
docker run hello-world
```
![file](https://img-blog.csdnimg.cn/20191015213332178.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

证明docker在win 10 上安装成功啦。至于接下来怎么使用，我们下篇再讲。

# Ubuntu 安装
我的是Ubuntu18.0.4 的，安装方法也很简单。
```
#卸载旧版本
sudo apt-get remove docker docker-engine docker.io

# 安装包更新
sudo apt-get update

# 安装依赖
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# 加Docker官方GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#设置稳定版的Docker仓库
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#安装 docker-ce
sudo apt-get install docker-ce
```
查看安装docker版本 
```
docker version 
```
![file](https://img-blog.csdnimg.cn/20191015213333222.jpeg)


运行hello-world
```
docker run hello-world
```
发现并没有出现下面错误

![file](https://img-blog.csdnimg.cn/20191015213333511.jpeg)

docker进程使用Unix Socket而不是TCP端口。而默认情况下，Unix socket属于root用户，需要root权限才能访问。
所以使用
```
sudo docker run hello-world
```
或者并将当前用户加入到docker用户组中。
默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。
```
#建立 docker 组：
sudo groupadd docker

# 将当前用户加入 docker 组：
sudo usermod -aG docker $USER

#更新用户组
newgrp docker     

#测试docker命令是否可以使用sudo正常使用
docker ps    
```
![file](https://img-blog.csdnimg.cn/20191015213334263.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 番外
到此为止，我们的win10 环境和ubuntu 环境都已经搭建好docker 啦。下篇就让我们继续学习怎么使用docker 吧。

好了，就说这么多啦

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/20191015213334732.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

