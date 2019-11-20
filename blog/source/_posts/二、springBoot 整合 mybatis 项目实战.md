layout: post
title: 二、springBoot 整合 mybatis 项目实战
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
date: 2019-09-19 22:11:53
---

# 前言
上一篇文章开始了我们的springboot序篇，我们配置了mysql数据库，但是我们sql语句直接写在controller中并且使用的是jdbcTemplate。项目中肯定不会这样使用，上篇文章也说了，会结合mybatis 或者JPA 使用。我们这篇文章就来结合 mybatis 来使用吧，至于为什么选mybatis 而不是JPA ，这个看个人洗好吧。然后这篇文章会附带一讲一下今天为项目新增的配置。主要是为了保持项目和文章的一致性。

先贴出我们今天项目的结构吧，和昨天贴出来的差不多，在那基础上添加了一些东西，整个框架的模型算是成型了。
![file](https://img-blog.csdnimg.cn/20190919221150687.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 引入mybatis依赖
一般改动都是从pom.xml 开始的，我们在昨天基础上的pom.xml  文件中加上mybatis 依赖,版本自己选吧。
```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
 </dependency>
```
# Entry层
昨天我们创建了一个user表 并插入了一条数据，我们就先用这张表吧，所以我们在entry 包中创建一个UserEntry 的实体类.
代码如下：
```
@Getter
@Setter
public class UserEntry {
    private int id;
    private String userName;
    private String password;
    private String email;
    private String roleCode;
    private String roleName;
    private String gmtCreate;
    private String gmtUpdate;
    private String nickname;
    private String userCreate;
}
```
可以看到这里用的注解和@Getter 和@Setter。这里就是避免代码中写入过多的get和 set 方法，使得代码变得更加简洁。

# Dao 层
Dao是用来处理数据的，这里我们引入了mybatis ,可以使用注解，也可以创建xml 文件，将sql 语句写在xml 文件中。我们这里就采用注解的方式吧，毕竟我们springboot项目，不想再出现xml配置文件了(后期也说不定哈哈)。

在dao包下创建一个userMapper 接口。代码如下：
```

@Mapper
public interface UserMapper {

    @Select("select id,username as userName,password,email,role_code as roleCode,gmt_create as gmtCreate,gmt_update as gmtUpdate,nickname as nickName,user_create as userCreate from sys_user")
    List<UserEntry> findUserList();

    @Insert({"insert into sys_user(username,password,email) values('${user.userName}','${user.password}','${user.email}')"})
    int add(@Param("user") UserEntry user);

    @Delete("delete from sys_user where id = #{id}")
    int delete(int id);

}
```
我们这里就先写一个查询、删除和插入的方法吧。可以看到，在接口上引入@Mapper  注解，然后就可以直接使用@Select 和@Insert 等注解啦，直接把注解sql 语句写在这里面就可以了。

# Service 层
service层我们一般都以一个接口和一个实现接口的具体类。
## service 接口
代码如下：
```

public interface UserService {

    List<UserEntry> findUserList();

    int addUser(String userName,String password,String email);

    int deleteUser(int id);
}
```
## serviceImpl 具体实现类。
在service 包的impl 包创建 UserServiceImpl 类。代码如下：
```

@Service
public class UserServiceImpl implements UserService {


    @Autowired
    protected UserMapper userMapper;

    @Override
    public List<UserEntry> findUserList() {
        return userMapper.findUserList();
    }

    @Override
    public int addUser(String userName, String password, String email) {
        UserEntry user=new UserEntry();
        user.setUserName(userName);
        user.setPassword(password);
        user.setEmail(email);
        return userMapper.add(user);
    }

    @Override
    public int deleteUser(int id) {
        return userMapper.delete(id);
    }
}

```

# controller 层
我们在controller 包下的userinfo  包下创建UserController 类。代码如下：
```

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/list",method = RequestMethod.GET)
    public List<UserEntry> findUserList(){
        return userService.findUserList();
    }

    @RequestMapping(value = "/add",method = RequestMethod.GET)
    public String addUser(@RequestParam(value = "userName")String uaserName,@RequestParam(value = "password")String password,@RequestParam(value = "email")String email){
        int falg=userService.addUser(uaserName,password,email);
        if(falg>0){
            return "success";
        }
        return "error";
    }

    @RequestMapping(value = "/delete",method = RequestMethod.GET)
    public String deleteUser(@RequestParam(value = "id")int id){
        if(userService.deleteUser(id)>0){
            return "success";
        }
        return "error";
    }
}
```

# 测试
好了，万事具备，来测试吧。我们启动项目后，在浏览器输入
```
添加一个用户
http://localhost:9090/zlflovemm/user/add?userName=qaz&password=123456&email=123@qq.com
```
![file](https://img-blog.csdnimg.cn/20190919221150962.jpeg)


```
查询所有用户
http://localhost:9090/zlflovemm/user/list
```
![file](https://img-blog.csdnimg.cn/20190919221151243.jpeg)

```
删除一个用户
http://localhost:9090/zlflovemm/user/delete?id=19
```
![file](https://img-blog.csdnimg.cn/20190919221151558.jpeg)

都没有问题啦，说明基本的增删改查整合 mybatis 都是可行的。

# 配置多环境文件
好了，大头讲完了，我们现在来讲讲项目今天进行了哪些配置。首先我们来看下我们项目中多了好几个配置文件。
![file](https://img-blog.csdnimg.cn/20190919221151978.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
分别对应的是开发环境，测试环境，生产环境。毕竟我们在实际开发过程中，这三个环境都是经常有用到的，总会有数据库连接改来改去的问题。这里直接配置多份，想用哪个用那个就可以了。避免反复改容易出错的问题。
我这里就暂时把连接mysql 的链接放到不同环境了。
先在application.properties中加入
```
spring.profiles.active=dev
表示用的是开发环境，就会读取application-dev.yml 文件中的配置。
```
application-dev.xml文件内容如下，另外两个文件也差不多，就不贴出来了。
![file](https://img-blog.csdnimg.cn/20190919221152316.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 配置日志
项目中怎么能缺乏日志文件呢，我这里用的springboot 自带的日志框架logback 也很方便。我们先在application.properties 中加入
```
#日志配置
logging.level.org.springframework.web=info
logging.config=classpath:logback.xml
debug=true
```
然后在 application.properties  同目录下创建一个 logback.xml.内容如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">

    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="./logs" />
    <property name="INFO_FILE" value="zlflovemm_log" />
    <property name="ERROR_FILE" value="zlflovemm_error" />

    <!--控制台日志， 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度,%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 文件保存日志的相关配置，同步 -->
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <Prudent>true</Prudent>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>
                ${LOG_HOME}/${INFO_FILE}-%d{yyyy-MM-dd}.log
            </FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}[%t][%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!-- 文件保存日志的相关配置，同步 -->
    <appender name="ERROR"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <Prudent>true</Prudent>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>
                ${LOG_HOME}/${ERROR_FILE}-%d{yyyy-MM-dd}.log
            </FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}[%t][%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR" level="error" />
    </root>
</configuration>
```

这样项目日志就配置好了，至于日志的具体配置，修改logback.xml 里面参数就可以了。和log4g差不多。

# 配置banner
最后既然是一个项目，当然得有点标志性的东西，比如logo。springboot 给我们留下了一个彩蛋就是banner。我们可以在项目启动的时候，显示我们独一无二的logo .
在application.properties 同目录下创建 banner.txt 
![file](https://img-blog.csdnimg.cn/20190919221152597.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
艺术字大家在网上自行搜索，这里就不推荐啦哈哈。这样我们在项目启动的时候就会加载我们的logo. 算是给广大的我们一点福利吧。

# 番外
今晚总算是写完了，本来会早点的，但是不知道怎么就弄这么晚了。
今天项目的代码也同步到github 上啦。
github地址：https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)



















