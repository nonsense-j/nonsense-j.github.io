layout: post
title: 三、SpringBoot 整合mybatis 多数据源以及分库分表
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
date: 2019-09-21 18:41:27
---

# 前言
说实话，这章本来不打算讲的，因为配置多数据源的网上有很多类似的教程。但是最近因为项目要用到分库分表，所以让我研究一下看怎么实现。我想着上一篇博客讲了多环境的配置，不同的环境调用不同的数据库，那接下来就将一个环境用到多个库也就讲了。所以才有了这篇文章。
我们先来看一下今天项目的项目结构，在上篇博客的基础上进行了一定的增改，主要是增加了一个 config 文件，在dao 中分了两个子包mapper1 和mapper2 将原先的UserMapper 移入到了 mapper1 中。好了，开始正文
![file](https://img-blog.csdnimg.cn/20190921184125101.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 多数据源配置
##  背景
在这之前，还是先说一下为什么会存在多数据源。如果项目小的话，当然是所有的数据以及逻辑处理都操作同一个库。但是当业务量大的话，就会考虑到分库了。比我会将也日志入库数据存放到单独的数据库。或者用户权限信息单独的一个库存放。这种如果只是简单的分库，一个项目中就用到2~4 个数据库的话，这种多数据源配置就有意义啦。在配置文件中配置好这几个数据源，都有唯一标识。项目在启动加载的时候都进行初始化，然后在调用的时候，想用哪个库就哪个数据源的连接实例就好了。

如果不整合 mybatis 的话，直接使用使用spring 自带的jdbcTemplate ，那配置多数据源，以及使用都比较简单，但是整合 mybatis 的话，就相对复杂点。我们一步一步来将讲解。  

## 修改配置文件
打开application-dev.yml 文件，添加数据源。
```
#开发环境
spring:
  # 数据源配置
  datasource:
    one:
      driver-class-name: com.mysql.jdbc.Driver
      jdbc-url: jdbc:mysql://192.168.252.53:3306/zlflovemm?characterEncoding=utf-8&useSSL=false&zeroDateTimeBehavior=CONVERT_TO_NULL
      username: root
      password: 123456
      max-idle: 10
      max-wait: 10000
      min-idle: 5
      initial-size: 5
    two:
      driver-class-name: com.mysql.jdbc.Driver
      jdbc-url: jdbc:mysql://192.168.252.53:3306/zlfdb?characterEncoding=utf-8&useSSL=false&zeroDateTimeBehavior=CONVERT_TO_NULL
      username: root
      password: 123456
      max-idle: 10
      max-wait: 10000
      min-idle: 5
      initial-size: 5
```

这里需要注意的是如果使用的是springboot 2.0 以上的，那么注意是 driver-class-name 和
 jdbc-url 而不是driverClassName和url.这里是一个坑，提醒大家一下。
## 配置数据源
接下来就需要我们手动的加载什么什么数据源了，我们在config中创建 DataSourcesConfig 类
```
@Configuration
public class DataSourcesConfig {

    @Bean(name="dbOne")
    @ConfigurationProperties(prefix = "spring.datasource.one")
    @Primary
    DataSource dbOne(){
        return DataSourceBuilder.create().build();
    }

    @Bean(name="dbTwo")
    @ConfigurationProperties(prefix = "spring.datasource.two")
    DataSource dbTwo(){
        return DataSourceBuilder.create().build();
    }

}
```
这里定义了两个数据源的DataSource。分别是我们在配置文件中配置的one 和two 。注解@Primary 表示默认使用的数据源。

MyBatisConfigOne 类
```
@Configuration
@MapperScan(basePackages = "com.quellan.zlflovemm.dao.mapper1",sqlSessionFactoryRef = "sqlSessionFactory1",sqlSessionTemplateRef = "sqlSessionTemplate1")
public class MyBatisConfigOne {
    @Resource(name = "dbOne")
    DataSource dbOne;

    @Bean
    @Primary
    SqlSessionFactory sqlSessionFactory1()throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dbOne);
        return bean.getObject();
    }
    @Bean
    @Primary
    SqlSessionTemplate sqlSessionTemplate1() throws Exception{
        return new SqlSessionTemplate(sqlSessionFactory1());
    }
}
```
MyBatisConfigTwo 类
```
@Configuration
@MapperScan(basePackages = "com.quellan.zlflovemm.dao.mapper2",sqlSessionFactoryRef = "sqlSessionFactory2",sqlSessionTemplateRef = "sqlSessionTemplate2")
public class MyBatisConfigTwo {
    @Resource(name = "dbTwo")
    DataSource dbTwo;

    @Bean
    SqlSessionFactory sqlSessionFactory2()throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dbTwo);
        return bean.getObject();
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate2()throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory2());
    }
}
```
注意连个文件的区别：
![file](https://img-blog.csdnimg.cn/20190921184125425.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

## dao 层
在dao 层创建了两个包mapper1 和mapper2 .包里面的UserMapper类的内容是完全一样，放在不同的包中只是区分使用哪个数据源。和昨天是一样的。
```
public interface UserMapper {

    @Select("select id,username as userName,password,email,role_code as roleCode,gmt_create as gmtCreate,gmt_update as gmtUpdate,nickname as nickName,user_create as userCreate from sys_user")
    List<UserEntry> findUserList();


    @Insert({"insert into sys_user(username,password,email) values('${user.userName}','${user.password}','${user.email}')"})
    int add(@Param("user") UserEntry user);

    @Delete("delete from sys_user where id = #{id}")
    int delete(int id);
}
```

## service 层
UserService接口
```
public interface UserService {

    List<UserEntry> findUserList();

    int addUser(String userName,String password,String email);

    int deleteUser(int id);

    List<UserEntry> findUserList2();

    int addUser2(String userName,String password,String email);

    int deleteUser2(int id);
}

```

UserServiceImpl类：
```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    protected UserMapper userMapper;

    @Autowired
    protected UserMapper2 userMapper2;


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

    @Override
    public List<UserEntry> findUserList2() {
        return userMapper2.findUserList();
    }

    @Override
    public int addUser2(String userName, String password, String email) {
        UserEntry user=new UserEntry();
        user.setUserName(userName);
        user.setPassword(password);
        user.setEmail(email);
        return userMapper2.add(user);
    }

    @Override
    public int deleteUser2(int id) {
        return userMapper2.delete(id);
    }
}
```

## controller 层
userController
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

    @RequestMapping(value = "/list2",method = RequestMethod.GET)
    public List<UserEntry> findUserList2(){
        return userService.findUserList2();
    }

    @RequestMapping(value = "/add2",method = RequestMethod.GET)
    public String addUser2(@RequestParam(value = "userName")String uaserName,@RequestParam(value = "password")String password,@RequestParam(value = "email")String email){
        int falg= userService.addUser2(uaserName,password,email);
        if(falg>0){
            return "success";
        }
        return "error";
    }

    @RequestMapping(value = "/delete2",method = RequestMethod.GET)
    public String deleteUser2(@RequestParam(value = "id")int id){
        if(userService.deleteUser2(id)>0){
            return "success";
        }
        return "error";
    }
}
```

## 测试
![file](https://img-blog.csdnimg.cn/20190921184125702.jpeg)
![file](https://img-blog.csdnimg.cn/20190921184125970.jpeg)
可以看到是从不同的库中调出来的。这样就说明我们springboot配置多数据源整合mybatis 已经成功了。其实最主要就是config 包下的那三个配置类。其他的都是常见的业务逻辑，所以后面我就没有怎么讲了，代码会同步到github  上，想要实践的可以拿源码下来实践。


到此我们springboot整合mybatis  多数据源已经配置好了，但是我们配置下来可以发现，我们如果想要配置几个数据源就得在 dao 层创建多少个子包用来区分。那如果我们数据量足够大，要分库分表而不是几个库呢？

# 分库分表
## 背景
其实分库分表和多数据源是一样的，只不过是数据源更多了，多到在配置中配置所有的连接显得很臃肿，所以不得不另觅它法。分库分表就是 在项目中配置连接主库的连接，从主库中读取各个分库的连接，然后进行动态的加载，那个接口想调用那个分库就加载这个分库的连接。
我现在项目做的由于不用整合mybatis 直接使用jdbcTemplate ，所以实现起来不是很麻烦。

## 思路
主要就两个类；
GetDynamicJdbcTemplate类：手动的创建连接。
```
/**
 * @ClassName GetDynamicJdbcTemplate
 * @Description 获取动态的jdbcTemplate
 * @Author zhulinfeng
 * @Date 2019/9/20 14:35
 * @Version 1.0
 */
public class GetDynamicJdbcTemplate {

    private  String driverClassName;
    private  String url;
    private  String dbUsername;
    private  String dbPassword;
    private JdbcTemplate jdbcTemplate;

    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }

    public GetDynamicJdbcTemplate(String driverClassName, String url, String dbUsername, String dbPassword){
        this.driverClassName=driverClassName;
        this.url=url;
        this.dbUsername=dbUsername;
        this.dbPassword=dbPassword;
        this.jdbcTemplate=new JdbcTemplate(getDataSource());
    }

    public DriverManagerDataSource getDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(dbUsername);
        dataSource.setPassword(dbPassword);
        return dataSource;
    }
}
```

GetJdbcTemplateMap类在项目启动的时候，会读取主库中的配置，将所有分库的连接都创建好放到map中。我们是按照地市分表的，接口在调用的时候根据前端传过来的地市就可以知道使用哪个数据库连接了。
```

@Component
@Slf4j
    public class GetJdbcTemplateMap implements ApplicationRunner {

    @Autowired
    @Qualifier("baseTemplate")
    private JdbcTemplate jdbcTemplate;

    public static Map<String,JdbcTemplate> JdbcTemplateMap=new HashMap<>();

    @Override
    public void run(ApplicationArguments args) throws Exception {
        String sql="CALL proc_baseinfo_cfg_dbsetting_query()";
        List<Map<String, Object>> list = jdbcTemplate.queryForList(sql);
        if(list!=null && !list.isEmpty()){
            insertMap(list);
        }
    }

    private void insertMap(List<Map<String, Object>> list){
        for(Map<String, Object> map :list){
            String url="jdbc:mysql://" map.get("serverip") ":" map.get("dbport") "/" map.get("dbname") "?allowMultiQueries=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull";
            log.info(url);
            String  dbUsername=  map.get("user").toString();
            String  dbPassword=  map.get("password").toString();
            GetDynamicJdbcTemplate getDynamicJdbcTemplate=new GetDynamicJdbcTemplate(ConstantClass.DRIVERCLASSNAME,url,dbUsername,dbPassword);
            JdbcTemplate jdbcTemplate=getDynamicJdbcTemplate.getJdbcTemplate();
            JdbcTemplateMap.put(map.get("cityid").toString(),jdbcTemplate);
        }
    }
}
```
![file](https://img-blog.csdnimg.cn/20190921184126325.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

在接口中调用也很方便。
![file](https://img-blog.csdnimg.cn/20190921184126584.jpeg)

但是上面讲的只适合我们自己特有的业务，并且也没有整合mybatis ,所以我就没有写在我自己的项目中，这里提供出来是给大家一个思路。


# 番外
也算是写完了这篇，感觉写的不是很好，但是有不知道怎么修改，暂时就先这样吧，后续有思路了再进行修改。又问问我为什么不先整合Thymeleaf  弄出页面来。之所以没有弄，是因为我想后期做前后端分离都是以接口的形式调用。所以想先将后端的部分都搭建好，再来整合前端的。
好了，就说这么多啦，今天项目的代码也同步到github 上啦。
github地址：https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019092118412737.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)





