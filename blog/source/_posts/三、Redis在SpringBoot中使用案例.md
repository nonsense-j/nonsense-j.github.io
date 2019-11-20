layout: post
title: 三、Redis在SpringBoot中使用案例
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
date: 2019-08-03 18:08:02
---

# 前言
最初的目的就想要在项目中把Redis用起来，然后最近公司的项目全部需要转成springboot，所以现在的项目都是Springboot的，自己刚好也研究下Springboot的。所以才有了下文的案例。

# 项目结构以及相关配置
先创建一个springboot 项目，目录结构大体如下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803160523904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
**在pom.xml 加入依赖**
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
```
说明一下，第一个依赖starter-web 是创建web应用的依赖。lombok 是我自己添加的一个依赖用来注解日志，属性的get/set方法比较方便，其他的三个依赖就是项目中使用redis的依赖啦，一般项目中想要使用redis引入这三个依赖就可以了。

在application.properties中配置redis

```
#配置redis
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=192.168.252.53
# Redis服务器连接端口
spring.redis.port=6379  
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

# 创建Dao层
创建dao 包，创建一个User 类,这里使用了lombok提供的@Getter 和@Setter 非常方便，代码看着也很简洁。

```
import lombok.Getter;
import lombok.Setter;
import java.io.Serializable;

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

# 创建Service层
创建一个service 包，创建一个RedisService类，代码如下：

```
import com.zlf.learning.Redis.dao.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Service;
@Service
@Slf4j
public class RedisService {
    @Autowired
    private RedisTemplate redisTemplate;
    public boolean setUser(User user){
        ValueOperations ops=redisTemplate.opsForValue();
        ops.set(user.getNickname(),user);
        log.info("{}",user.toString());
        return true;
    }
    public User getUser(String name){
        ValueOperations ops=redisTemplate.opsForValue();
        return (User) ops.get(name);
    }
}
```
这里面的代码也非常的清晰，使用到的RedisTemplate ，类似于JdbcTemplate .
ValueOperations ops=redisTemplate.opsForValue();就是连接了redis数据库。之后就可以从redis 中获取和添加值啦。

# Controller层
创建一个controller 包，创建一个RedisController类代码如下：

```
@RestController
public class RedisController {
    @Autowired
    private RedisService redisService;
    @RequestMapping("/getUser")
    public User  getUser(){
        String name="quellan";
        return redisService.getUser(name);
    }
    @RequestMapping("/setUser")
    public String setUser(){
        User user=new User("aa@qq.com","quellan","123456","朱",new Date().getTime()+"");
        redisService.setUser(user);
        return "添加成功";
    }
}
```
# 测试
到此为止基础的就已经完全搭建好了，可以测试运行下。启动spring boot项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803162239820.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803162301820.png)
在redis查一下，发现redis中的key 值并不是我们设置的quellan ,而是一串。这就很难受啦。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803162405899.png)
查了一下，原来是使用的RedisTemplate ，spring-data-redis的RedisTemplate<K, V>模板类在操作redis时默认使用JdkSerializationRedisSerializer来进行序列化.这个具体的放在下一章讲吧，感觉一会讲不完，先跳过哈哈。
上面的测试说明项目中已经可以正常使用redis啦。

# Session共享
按理说到上面就已经差不多，接下来来点骚操作。
分布式怎么共享session。简单来说就是一个项目部署了多个，怎么确保一个用户访问不同的项目（用户实际是无感知的，通过Nginx转发，实现负载均衡）时确保session一致。盗一张图来展示一下吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803174647376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
这张图就是多个Tomcat，那怎么实现session共享呢，就是把session存到redis中，每次去就从redis中取，这样就保证了session共享啦。

那这样是不是每次存session都需要手动存到redis中呢，常理来说当然是的，但是既然是SpringBoot 当然需要不一样啦，只需要增加一个依赖，人家就能帮你自动的加载到redis中。下面来看

## 增加依赖

```
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
```
配置上面已经配置好了

## 增加SpringSession 类
在controller 包中加一个SpringSession 类，命名可能不太规范，见谅哈

```
@RestController
public class SpringSession {
    @Value("${server.port}")
    Integer port;

    @RequestMapping("/setSession")
    public String setSession(HttpSession session){
        session.setAttribute("key","quellanAn");
        return String.valueOf(port);
    }
    @RequestMapping("/getSession")
    public String getSession(HttpSession  session){
        return session.getAttribute("key")+":"+port;
    }
}
```
代码很简单，就是session存一个值，get获取。这里可以看到没有任何操作redis数据库的对吧。

## 测试场景1
先运行项目，查看一下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803175932260.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803175950815.png)

这些都没有什么，我们去redis中看一下，redis中是有session值的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803180040168.png)

测试场景2
好的，接下来继续，因为上面还看不出来共享session。我们将项目打包成jar包运行，这样我们就可以多个端口运行啦，模拟分布式。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803180304303.png)
run.bat 中代码：

```
title learingPorject8090
chcp
java -jar learningproject-1.0.0.jar --server.port=8090
```
run2.bat 改一下端口号就好了。
然后运行jar包，在界面访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803180500323.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190803180520136.png)

这样就实现session共享啦。

## 番外
再多说一句，设置session的过期时间
在启动类中加上注解
设置过期时间1分钟
```
@EnableRedisHttpSession(maxInactiveIntervalInSeconds=60)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080318072784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)



后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)









