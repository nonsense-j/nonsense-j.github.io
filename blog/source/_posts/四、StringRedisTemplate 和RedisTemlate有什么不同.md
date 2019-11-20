layout: post
title: 四、StringRedisTemplate 和RedisTemlate有什么不同
author: QuellanAn
categories: 
  - Redis
tags:
  - redis
  - java
  - linux
date: 2019-08-07 18:03:40
---

# 前言
上一篇文章讲的搭建一个redis+ spring boot 的实例，用到了RedisTemplate，可以成功的访问redis数据库，也可以从中读取数据并显示在页面上，但是呢有瑕疵，那就是其实存在数据库中的Key值是乱码的，类似下面图片这样的。在网上找了一堆的解决办法，看到有StringRedisTemplate 代替RedisTemlate的，所以这边文章就来说说二者到底有什么不同，两者有哪些与缺点，以及在项目中我们如何去使用它。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807170618562.png)
#  二者不同
先来看看StringRedisTemplate ，为什么先看他，因为它实际上是继承RedisTemplate的，并且源码很简单，只有十几行，所以先来看看它。
源码：

```

public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        this.setKeySerializer(RedisSerializer.string());
        this.setValueSerializer(RedisSerializer.string());
        this.setHashKeySerializer(RedisSerializer.string());
        this.setHashValueSerializer(RedisSerializer.string());
    }

    public StringRedisTemplate(RedisConnectionFactory connectionFactory) {
        this();
        this.setConnectionFactory(connectionFactory);
        this.afterPropertiesSet();
    }

    protected RedisConnection preProcessConnection(RedisConnection connection, boolean existingConnection) {
        return new DefaultStringRedisConnection(connection);
    }
}
```
看源码可以看到，就两个构造方法，构造方法中对key和value 进行序列化，这个序列化是使用RedisSerializer.string()序列化的。看看RedisSerializer.string(）的源码可以发现就是将编码格式设置成了UTF-8
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807173140918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
再看看带参数的构造函数，多了一个RedisConnectionFactory 参数，这个参数是是在创建连接的时候，设置连接的信息。在网上copy了一个这个方法的实例，可以参考一下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807172021358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
看到这里，大伙差不多就应该知道StringRedisTemplate和RedisTemlate有什么不同了吧。StringRedisTemplate继承了RedisTemlate,但是又仅仅修改了key和values序列化的方式。那就说明StringRedisTemplate和RedisTemlate实际上就是key和values序列化的方式不同啦。
那接下来再看看RedisTemlate是怎么序列化的。RedisTemlate的源码就比较多了，我们这里就暂时先看其序列化的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807172652895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
可以看到redisTemplate是使用jdk默认编码格式来序列化的。
```
new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader())
```

所以才出现了文章最开始，使用redisTemplate，存的key值在redis数据库中实际上是乱码的。而StringTemplate不会。


# 二者优缺

关于二者优缺点，我们先来看一个例子：
还是上一篇博客的源代码，RedisService层使用的是RedisTemplate，界面上存取，显示都没有问题，这里重点关注一下，getUser()，我这里强转User,在界面上可以正常显示。

```
@Autowired
    private RedisTemplate redisTemplate;
    public boolean setUser(User user){
        ValueOperations ops=redisTemplate.opsForValue();
        ops.set(user.getNickname(),user);
        return true;
    }
    public User getUser(String name){
        ValueOperations ops=redisTemplate.opsForValue();
        return (User)ops.get(name);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807175821733.png)
那我们再使用StringTemplate
修改RedisService层

```
@Autowired
    private StringRedisTemplate stringRedisTemplate;
    public boolean setUser(User user){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        ops.set(user.getNickname(),user);
        return true;
    }

    public User getUser(String name){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        return (User)ops.get(name);
    }
```
再来实行set 和get 就会报错。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807180114742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
set 方法报错，说明不能将一个对象直接当做value值传过去，没有进行转换。而RedisTemplate却可以直接把对象当做value值存进去了。因为RedisTemplate在写入和读出的时候都进行了转换。
被逼无奈的修改了代码如下；
RedisController层

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
        user.setEmail("bb@qq.com");
        redisService.setUserBystringRedisTemplate(user);
        return "添加成功";
    }

    @RequestMapping("/getUserByStringRedisTemplate")
    public String  getUserByStringRedisTemplate(){
        String name="quellan";
        return redisService.getUserBystringRedisTemplate(name);
    }

}
```

RedisService层

```
@Service
@Slf4j
public class RedisService {
    
    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    public boolean setUser(User user){
        ValueOperations ops=redisTemplate.opsForValue();
        ops.set(user.getNickname(),user);
        return true;
    }

    public User getUser(String name){
        ValueOperations ops=redisTemplate.opsForValue();
        return (User)ops.get(name);
    }

    public boolean setUserBystringRedisTemplate(User user){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        ops.set(user.getNickname(),JSONObject.fromObject(user).toString());
        return true;
    }
    
    public String getUserBystringRedisTemplate(String name){
        ValueOperations ops=stringRedisTemplate.opsForValue();
        return JSONObject.fromObject(ops.get(name)).toString();
    }
}
```
server层就是分别使用RedisTemplate和StringRedisTemplate对User对象进行存和读的操作。特别注意一下StringRedisTemplate由于直接对象不能存，所以先转成string才能存进去的，读出来的时候，也是string形式返回的，如果读出来想要变成user类还得进一步转换。
来看看效果现在。先setUser,往redis中插入两条数据，这里可以看到我们代码中设置的key 是一样的，都是quellan
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190808094033782.png)
来看看获取结果getUser是使用RedisTemplate来获取的，邮箱是aa
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080809424355.png)
来看看getUserByStringRedisTemplate,邮箱是bb
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190808094715724.png)
这里是不是有说明了一个问题：使用RedisTemplate和StringRedisTemplate是相互独立的，在代码中使用相同的key值进行存储，不会替换，两份都会存在，具体原因还是刚刚提到的，其实他们真实存在redis数据库的key 是不一样的，所以才会独立。我们看看redis数据库。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190808095154807.png)可以发现redis数据库中有两个key值，其中key值为quellan的是使用stringRedisTemplate来来存储的，可以看到邮箱为bb。还有一个乱码的key值使用RedisTemplate存储的，在控制台怎么获取这个key值我暂时也不知道，有知道的小伙伴希望告知一下，嘿嘿。

上面说的redisTemplate 和StringRedisTemplate 是独立的，这个在项目中很容易出现坑的，所以小伙伴们得多多注意，不要存的时候用StringRedisTemplate 读的时候用redisTemplate 或者相反。这样可能回到导致死活读不出数据。

# 总结
上面说了这么多，总结一下吧:
1、RedisTemplate和StringRedisTemplate存储是分开存的，也就是代码中相同的key实际上在redis数据库中有两个key.原因是RedisTemplate进行了转换，而StringRedisTemplate直接以代码key值存储了。
2、如果我们存一些简单的数据结构，建议使用StringRedisTemplate,因为方便在数据库中查看。如果我们存一些复杂的数据接口，比如对象里面还包含多个对象的，就建议使用RedisTemplate了，系统会帮忙转换，省去我们自己转换的麻烦，上面的代码可以看到直接将取到的value值强转成user都没问题，很方便。
3、二者可以配合使用，但是不能混着用。

# 番外
## redis控制台中文乱码
刚刚我们在控制台 `get quellan `的时候发现userName 是乱码的。
那是因为我们进入的方式不对。需要用

```
redis-cli --raw -a password
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080810083976.png)


后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019091922115335.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)








 
