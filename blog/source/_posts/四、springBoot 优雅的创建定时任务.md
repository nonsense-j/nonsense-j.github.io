layout: post
title: 四、springBoot优雅的创建定时任务
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
date: 2019-09-26 16:12:02
---

# 前言
好几天没写了，工作有点忙，最近工作刚好做一个定时任务统计的，所以就将springboot 如何创建定时任务整理了一下。
总的来说，springboot创建定时任务是非常简单的，不用像spring 或者springmvc 需要在xml 文件中配置，在项目启动的时候加载。spring boot 使用注解的方式就可以完全支持定时任务。
不过基础注解的话，可能有的需求定时任务的时间会经常变动，注解就不好修改，每次都得重新编译，所以想将定时时间存在数据库，然后项目读取数据库执行定时任务，所以就有了基于接口的定时任务。下面就分基于注解和基于接口详细讲解。

# 基于注解
pom.xml 文件不用修改，我们原本的项目就支持，其实定时器是springboot框架自带的，不用引入什么依赖。我们直接创建一个autotask 包，创建一个AutoTask类。
```
@EnableScheduling
@Component
@Slf4j
public class AutoTask {
    @Scheduled(cron="*/6 * * * * ?")
    private void process(){
        log.info("autoTask ");
    }
}
```

这样一个定时器就创建啦，在项目启动后，会每隔6s 打印“autoTask”的日志。是不是很简单。主要用到的两个注解就是@EnableScheduling 和 @Scheduled。
注解@EnableScheduling 就是开启定时任务的。哪个类的中的方法想要定期执行，就在这个类上加入这个注解。当然这个这个注解也可以加在启动类上。加在启动类上表示项目中所有的类都可以创建定时任务。
![file](https://img-blog.csdnimg.cn/20190926161157924.jpeg)

@Scheduled 注解就是我们常见的定时器啦，后面的cron 就是定时任务表达式。在方法上注解，表示这个方法定期执行。
![file](https://img-blog.csdnimg.cn/20190926161158434.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
不过@Scheduled 可以进行两种配置，我们熟悉的cron ,还有一种是fixedRate。比如fixedRate=6000 表示方法每6秒钟执行一次。
我们来启动项目看一下，可以看到两个方法都在定期执行。
![file](https://img-blog.csdnimg.cn/20190926161158769.jpeg)


# 基于接口
上面可以看到springboot 基于注解是非常方便的。但是对于频繁变动或者一个项目中有很多的定时器那就不方便管理了。所以统一将定时器信息存放在数据库中。
```

DROP TABLE IF EXISTS `scheduled`;
CREATE TABLE `scheduled`  (
  `cron_id` varchar(30) NOT NULL PRIMARY KEY,
  `cron_name` varchar(30) NULL,
  `cron` varchar(30) NOT NULL
);
INSERT INTO `scheduled` VALUES ('1','定时器任务一','0/6 * * * * ?');
```
![file](https://img-blog.csdnimg.cn/20190926161159239.jpeg)

在dao 层mapper1包下创建一个CronMapper接口，很简单的就获取cron
```
public interface CronMapper {

    @Select("select cron from scheduled where cron_id = #{id}")
    public String getCron(int id);
}
```
![file](https://img-blog.csdnimg.cn/20190926161159580.jpeg)


这里我们就不写service 层了。直接在autotask 包下创建一个AutoTaskFromDB类
```
@Slf4j
@Component
public class AutoTaskFromDB implements SchedulingConfigurer {

    @Autowired
    protected CronMapper cronMapper;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {

        scheduledTaskRegistrar.addTriggerTask(() -> process(),
                triggerContext -> {
                    String cron = cronMapper.getCron(1);
                    if (cron.isEmpty()) {
                       log.info("cron 为空");
                    }
                    return new CronTrigger(cron).nextExecutionTime(triggerContext);
                }
        );
    }

    private  void process(){
        log.info("formDB ");
    }
}
```

可以看到也很简单，就是实现SchedulingConfigurer 这个吧接口，addTriggerTask（）是添加一个定时器。
process（）方法是我们需要定时执行的方法体。
CronTrigger(cron).nextExecutionTime(triggerContext) 就是从数据库读取的cron 创建定时器。

这个类我没有加上@EnableScheduling 注解，因为我在启动类上加上了，如果你们启动类上没有加，这里记得加上。

测试一下;下图可以看到三个定时任务都执行了，fromDB 是从数据库读取的。
![file](https://img-blog.csdnimg.cn/20190926161159969.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)



# cron 
cron 用法网上有很多，也没有什么讲的这里就附带记录下
## 结构
cron表达式是一个字符串，分为6或7个域，每两个域之间用空格分隔，
其语法格式为："秒域 分域 时域 日域 月域 周域 年域"
## 取值范围

|域名|	可取值|	可取符号（仅列部分常用）|
|--|--|
|秒域|	0~59的整数|	  *    -    ,    /
|分域|	0~59的整数|	  *    -    ,    /
|时域	|0~23的整数|	  *    -    ,    /
|日域	|1~31的整数|	  *    -    ,    /    ?    L
|月域	|1~12的整数或JAN~DEC|	  *    -    ,    /
|周域	|1~7的整数或SUN~SAT|  *    -    ,    /    ?    L    # 
|年域	|1970~2099的整数	|  *    -    ,    /


## 常例
| 表达式                       | 意义                      |
| ------------------------- | ----------------------- |
| 每隔5秒钟执行一次                 | \*/5  \*  \* \*  \*  ?  |
| 每隔1分钟执行一次                 | 0  \* /1  \* \* \*  ?   |
| 每天1点执行一次                  | 0  0  1  \*  \*  ?      |
| 每天23点55分执行一次              | 0  55  23 \*  \* ？      |
| 每月最后一天23点执行一次             | 0  0  23  L  \*  ？      |
| 每周六8点执行一次                 | 0  0  8  ?  \*  L       |
| 每月最后一个周五，每隔2小时执行一次        | 0  0  \*/2  ?  \*  6L   |
| 每月的第三个星期五上午10:15执行一次      | 0  15  10  ? \*  5#3    |
| 在每天下午2点到下午2:05期间的每1分钟执行   | 0  0-5  14  \*  \*  ?   |
| 表示周一到周五每天上午10:15执行        | 0  15  10  ? \*  2-6    |
| 每个月的最后一个星期五上午10:15执行      | 0  15  10  ?  \*  6L    |
| 每天上午10点，下午2点，4点执行一次       | 0  0  10,14,16  \* \* ? |
| 朝九晚五工作时间内每半小时执行一次         | 0  0/30  9-17  *  * ?   |
| 每个星期三中午12点执行一次            | 0  0  12  ?  *  4       |
| 每年三月的星期三的下午2:10和2:44各执行一次 | 0  10,44  14  ?  3  4   |
| 每月的第三个星期五上午10:15执行一次      | 0  15  10  ?  *  6#3    |
| 每月一日凌晨2点30执行一次            | 0  30  2  1  *  ?       |
| 每分钟的第10秒与第20秒都会执行         | 10,20  \*  \*  \*  \* ? |
| 每月的第2个星期的周5，凌晨执行          | 0  0  0  ?  *  6#2      |


# 番外
本来这个知识点不应该放在这里讲的，但是不多，顺带写了，刚好也能做定时器。我们项目中往往有一些需求需要在项目启动的时候就执行，那这个我们怎么实现了。其实spring boot 使用起来也非常简单，只用实现 ApplicationRunner  就好了。
我们在autotask 包下创建一个AutoTaskFromSpringRunner类

```
@Slf4j
@Component
public class AutoTaskFromSpringRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        process();
    }

    private void process(){
        log.info(" run ApplicationArguments");
    }
}

```
启动项目看一下，可以发现这个会在项目启动后执行，但是只会执行一次。

![file](https://img-blog.csdnimg.cn/20190926161200531.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
那这个怎么用来做定时器呢？当然是结合线程来做啦，但是这个方法其实不建议，b毕竟线程很容易出问题，但是提供一种思路：

```
@Slf4j
@Component
public class AutoTaskFromSpringRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        process();
        new Thread(() -> {
            while (true) {
                process2();
                try {
                    Thread.sleep(6000);
                } catch (InterruptedException e) {
                    log.error("{}",e);
                }
            }
        }).start();
    }
    private void process(){
        log.info(" run ApplicationArguments");
    }
    private void process2(){
        log.info("线程定时器");
    }
}

```
启动项目看下，发现也是可以起到定时器的作用的。
![file](https://img-blog.csdnimg.cn/20190926161201238.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

好了，就说这么多啦，今天项目的代码也同步到github 上啦。
github地址：https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019092616120288.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)









