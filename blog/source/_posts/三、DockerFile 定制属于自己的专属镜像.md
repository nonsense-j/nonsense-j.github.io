layout: post
title: 三、DockerFile 定制属于自己的专属镜像
author: QuellanAn
categories: 
  - docker
tags:
  - docker
  - linux
---
# 前言
上篇文章我们知道了怎么操作镜像和容器，到基础都是从已经存在的镜像开始的，那我们自己怎样搭建一个镜像并使用它呢？接下来就让我们学习使用dockerfile 创建属于自己的镜像吧。

# dockerfile
在这之前，我们需要知道dockerfile ,因为我们就是通过dockerfile 来创建镜像的。那dockerfile 是什么呢？
dockerfile 是一个文件，文件里面是我们写的一条条的指令，然后通过```docker build ```
命令来构建一个镜像。
 现在难就难在这个指令怎么写，所以接下让我们一起看看dockfile 指令吧。
 
 ## dockerfile 指令
 ### FROM 
 ```
FROM <image>
FROM <image>:<tag>
#tag是可选的,默认会使用latest版本的基础镜像
 ```
 from 指令是依赖的基础镜像，所谓的定制镜像，是在其他的镜像上添加一些我们自己东西，定制成我们自己的镜像。当然我们也可以不依赖任何镜像，自己从头开始搭建。那就使用

 ```
FROM  scratch
 ```
 scratch 其实也是一个docker 镜像，但是这个镜像比较难特殊，它是一个虚拟镜像，里面什么都没有，是一个空白的镜像，所以如果想不依赖任何镜像，可以使用```from scratch```。
 
那现在又有一个问题了，dockfile 文件中可以出现多个From 么？

在docker 17.05 版本之前是不支持出现多个From 的，一个dockefile只能有一个From 指令，且必须放在文件中的第一行。因为作为基础镜像使用。在docker17.05 后支持多From 。表示构建的多重阶段，不过最终生成的镜像还是以最后一个From 基础镜像为基础的。

### RUN
run 指令 是表示在镜像构建时运行的指令。
两种格式：
```
#shell格式
run <命令>
eg: run apt-get update

#exec 格式
run ["可执行文件"，“参数1”，“参数2”...]
```

### COPY
复制文件的指令
```
copy 源路径  目标路径
#支持通配符
eg:copy hom?.txt /mydir/
```

### ADD
add 是更高级的复制。copy 有的功能它都用，它还能访问网络资源，源路径可以是一个URL。源路径文件也可以是一个压缩文件，可以直接解压。
所以如果想要直接复制一个压缩包进去的话，就要使用COPY 而不能只用ADD了。
官方建议是能使用COPY 的就使用COPY ,因为COPY 命令语义比较明确就是复制文件，并且ADD 指令会使得镜像构建缓存失效，使得镜像构建比较缓慢。

### CMD
cmd 指令是表示在运行容器时执行的指令。

```
#shell 格式
cmd  <命令>
eg:cmd echo $HOME

#exec 格式
cmd ["可执行文件"，“参数1”，“参数2”]
eg: cmd ["sh","-c","echo $HOME"]
```

### ENTRYPOINT
entrypoint 入口点
```
entrypoint <命令>

entrypoint ["可执行文件","参数1","参数2"]
```
entryPoint 指令和 cmd 指令功能类似，不过entrypoint 可以让镜像变成像命令一样使用，可以做应用运行前的准备工作。这个具体的后面讲。

### ENV 
env 是设置环境变量的指令，
```
env MY_VERSION 1.0.0
```

### ARG
arg 用于构建时传递的参数
```
arg <参数名>[=<默认值>]

eg: 
arg version
arg myversion=1.0.0
```

### VOLUME
定义匿名卷
```
volume <路径>
volume ["<路径1>"，["<路径2>"...]

eg： volume  /etc/docker/log
```

### EXPOSE
申明端口
```
expose <端口1>  [<端口2>...]
```
这里需要注意的是，expose 是申明容器应用端口，但是容器运行是并不一定就是开启这个端口提供服务。在dockerfile 中写入端口申明有两个好处，一是当做镜像服务的守护端口，方便映射，二是在运行时使用随机端口映射时，就会映射的expose设置的端口上。

好了，指令当然不止这些，更多的想了解的查看：

https://docs.docker.com/engine/reference/builder/

## 简单测试
之前这篇文章写到一半放下了，因为中间docker出了一点问题，下载镜像一直提示超时，然后设置了国内加速，才弄好。
上面我们了解了Dockerfile 指令，接下来就让我们先做一个简单的测试吧。
我们穿件一个springboot项目。创建一个HelloController 类。
```
@RestController
public class HelloController {
    @RequestMapping("/")
    public String hello(){
        return "hello docke 我的简单测试 ";
    }
}
```
然后打成jar 包。放到我们服务器的文件夹下。
并且在文件下创建Dockerfile文件
```
vim Dcokerfile

#文件内容
FROM java:8
VOLUME /tmp
ADD hello-1.0.0.jar hello-1.0.0.jar
ENTRYPOINT ["java","-jar","/hello-1.0.0.jar"]
```
可以看到用到的命令都是我们上面介绍的。java8作为基础镜像，/tmp作为数据卷, add 将本地jar包添加到镜像中，entrypoint 运行我们的jar包。

![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyMzhlNWUzODJmMWY0NzE0Nzk5MDE1NzIzMzYzNjYucG5n?x-oss-process=image/format,png)

在该目录下构建镜像现在，最后面的点别忘记了。
```
docker build -t helle:v1 .
```
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyMTJjYzQ2ZmUxZDE0YWQ3MGU2MDE1NzIzMzY2ODAucG5n?x-oss-process=image/format,png)
可以看到我们的镜像分4不构建，也就是构建四个镜像，因为我们Dockerfile 中有四条指令。前面说了后一条指令是在前一条指令的基础上构建镜像的。所以这四个镜像中前面三个就是中间镜像了。

我们现在看看我们创建的镜像。
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyZDk0NTYzNTY5Y2ZlMGRiNTUyMDE1NzIzMzY5OTcucG5n?x-oss-process=image/format,png)

我们接下来启动镜像
```
docker run -d -p 8090:8080 hello:v1 
```
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyNWVlYmU2YjM0NTExMjRlNWYyMDE1NzIzMzczNDIucG5n?x-oss-process=image/format,png)
其中 -d 是后台启动，-p 是映射端口，前面的是我们设置的端口，后面是项目运行的端口。
启动后我们在浏览器上访问下。
```
http://192.168.252.53:8090
```
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyZTkyN2JlYmI4MTllMGIyMDE2MDE1NzIzMzczNzgucG5n?x-oss-process=image/format,png)

这样我们通过docker 构建我们springboot 的项目，创建属于我们自己的镜像就完成了。

# 配置docker远程访问
我们现在要做的是，直接通过idea打包生成docker镜像。所以，第一步开启docker的远程访问，我的docker 是安装到服务器上的。我先在本地检测一下，服务器上的docker 是否开启的远程访问。
```
docker -H 192.168.252.53 info
```
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyNmI5ZWE5NDUyNjEwMDEyYTU3MDE1NzIzNDA2MjMucG5n?x-oss-process=image/format,png)
说明是没有开启docker的远程服务的。所以进入服务器。
执行如下操作,在docker.service. 文件夹下创建一个http-proxy.conf文件.
```
sudo mkdir /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```
文件内容
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```
然后重启daemon和docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
然后我们再 在本地测试一下。说明docker 的远程访问已经配置好了。
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyOTViZGRmZDAxNzI3YmE2NmFkMDE1NzIzNDExMTYucG5n?x-oss-process=image/format,png)


# idea配置
我们打开我们的hello 项目，在pom.xml 中增加配置
```
<properties>
        <java.version>1.8</java.version>
        <docker.image.prefix>quellanan</docker.image.prefix>
</properties>
```
在build 中增加。
```
<!-- Docker -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <!-- 将插件绑定在某个phase执行 -->
                <executions>
                    <execution>
                        <id>build-image</id>
                        <!-- 用户只需执行mvn package ，就会自动执行mvn docker:build -->
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!-- 指定生成的镜像名 -->
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <!-- 指定标签 -->
                    <imageTags>
                        <imageTag>${project.version}</imageTag>
                    </imageTags>
                    <!-- 指定 Dockerfile 路径 -->
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <!-- 指定远程 docker api地址 -->
                    <dockerHost>http://192.168.252.53:2375</dockerHost>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <!-- jar包所在的路径此处配置的对应target目录 -->
                            <directory>${project.build.directory}</directory>
                            <!-- 需要包含的jar包,这里对应的是Dockerfile中添加的文件名　-->
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>

```


在src/main/docker 中创建Dockerfile 文件，文件内容上面Dockerfile 内容一样
```
FROM java:8
VOLUME /tmp
ADD hello-1.0.0.jar hello-1.0.0.jar
ENTRYPOINT ["java","-jar","/hello-1.0.0.jar"]
```
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyNjI3MzNmODgyMGJiMTI0OGJkMDE1NzI1MDU1NzAucG5n?x-oss-process=image/format,png)

# mvn package
因为我们配置在构建的时候就会进行docker 打包。所以我们知己运行mvn package

![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyNzllOGUzZTIyZjExOWU2MDAzMDE1NzI1MDYyMDYucG5n?x-oss-process=image/format,png)
控制台查看是打包成功的。
我们去服务器上看下，有没有
![file](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ncmFwaC5iYWlkdS5jb20vcmVzb3VyY2UvMjEyNzBmZTZiOGQ0NThmODZmOTQxMDE1NzI1MDYyNjUucG5n?x-oss-process=image/format,png)
可以看到已经成功了。

# 番外
这篇总算写完啦算是，中间自己亲自试验，踩了一路坑，也算是初步弄好了。以后我们项目不用将jar 包放到服务器上再来创建镜像了，可以直接在我们项目中打包构建镜像就想构建jar 包一样简单。还是可以的吧。

好了，就说这么多啦

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤


![file](https://img-blog.csdnimg.cn/20191015213334732.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)


 













 
