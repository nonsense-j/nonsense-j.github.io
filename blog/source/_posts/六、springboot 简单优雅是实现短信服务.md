layout: post
title: 六、springboot 简单优雅是实现短信服务
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
---

# 前言
上一篇讲了 springboot 集成邮件服务，接下来让我们一起学习下springboot项目中怎么使用短信服务吧。
项目中的短信服务基本上上都会用到，简单的注册验证码，消息通知等等都会用到。所以我这个脚手架也打算将短息服务继承进来。
短息服务我使用的平台是阿里云的。网上有很多的短信服务提供商。大家可以根据自己的需求进行选择。

# 准备工作
在阿里云上开通服务，以及进行配置。这些阿里云官方文档都写的很清楚，怎么做就不细说的，大家可以参考一下这篇文章:
https://blog.csdn.net/qq_27790011/article/details/78339856

配置好之后你需要获取如下信息：

accessKeyId 、accessSecret 这两个是秘钥。在用户AccessKey 中可以找到。

signName 是签名名称。

![file](https://img-blog.csdnimg.cn/20191014132438633.jpeg)

templateCode 是模版code
![file](https://img-blog.csdnimg.cn/20191014132438907.jpeg)


# 添加依赖和配置
有了上面的准备工作，我们接下来开始在我们的项目中开发吧。一样的先在pom.xml 文件中加入依赖：
```
<!--阿里云短信服务-->
        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>aliyun-java-sdk-core</artifactId>
            <version>4.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.61</version>
        </dependency>
```
这个fastjson 不是必须的，就看你项目中有没有用到啦，没有用到的话，添加第一个依赖就好了。

然后在application.properties文件中加入配置，这四个参数，就是准备工作中我们获取的四个参数。
![file](https://img-blog.csdnimg.cn/20191014132439216.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# service 层
和邮件服务一样，我们这里没有涉及到数据库，就先直接写service 层，创建SmsService 接口和 SmsServiceImpl 类。

![file](https://img-blog.csdnimg.cn/20191014132439504.jpeg)

SmsServiceImpl的代码如下：
```
@Service
@Slf4j
public class SmsServiceImpl implements SmsService {

    @Value("${sms.accessKeyId}")
    private String accessKeyId;

    @Value("${sms.accessSecret}")
    private String accessSecret;

    @Value("${sms.signName}")
    private String signName;

    @Value("${sms.templateCode}")
    private String templateCode;

    @Override
    public boolean sendSms(String iponeNUmber) {
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId, accessSecret);
        IAcsClient client = new DefaultAcsClient(profile);
        CommonRequest request = new CommonRequest();
        request.setMethod(MethodType.POST);
        request.setDomain("dysmsapi.aliyuncs.com");
        request.setVersion("2017-05-25");
        request.setAction("SendSms");
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", iponeNUmber);
        request.putQueryParameter("SignName", signName);
        request.putQueryParameter("TemplateCode", templateCode);
        JSONObject object=new JSONObject();
        String randCode=getRandCode(6);
        log.info("验证码为：{}",randCode);
        object.put("code",randCode);
        request.putQueryParameter("TemplateParam", object.toJSONString());
        try {
            CommonResponse response = client.getCommonResponse(request);
            log.info(response.getData());
            return true;
        } catch (Exception e) {
            log.error("{}",e);
        }
        return false;
    }
    /**
     * 生成随机验证码
     * @param digits
     * @return
     */
    public static String getRandCode(int digits) {
        StringBuilder sBuilder = new StringBuilder();
        Random rd = new Random((new Date()).getTime());

        for(int i = 0; i < digits;   i) {
            sBuilder.append(String.valueOf(rd.nextInt(9)));
        }

        return sBuilder.toString();
    }
}

```

整体的代码逻辑很简单，首先是通过Value注解将配置文件中配置的那四个参数获取到。

sendSms()方法中 :

DefaultProfile 和 IAcsClient  是创建DefaultAcsClient实例并初始化。三个参数分别对应的是:地域ID，RAM账号的AccessKey ID， RAM账号AccessKey Secret。 

DescribeInstancesRequest 是创建API请求并设置参数。request.putQueryParamete()我们修改主要是修改这里面的参数。PhoneNumbers 是接收信息的手机号，这里我发送的是短信验证码。所以我这里生成一个6位的短息验证码。具体需求大家可以根据需求进行调整。

# controller 层
controller 层比较简单，就一个发送短信的接口，在sms包下创建SmsController类，代码如下：
```
@RestController
@RequestMapping("/sms")
public class SmsController {

    @Autowired
    private SmsService smsService;

    @RequestMapping(value = "/send",method = RequestMethod.GET)
    public String sendSms(@RequestParam(value = "userName")String userName){
        smsService.sendSms(userName);
        return "success";
    }
}
```

# 测试
到此为止，短信服务已经搭建好了，现在我们来测试一下，我们首先启动项目，然后调用接口：
```
http://localhost:9090/zlflovemm/sms/send?userName=13265459362
```
然后看下日志

![file](https://img-blog.csdnimg.cn/20191014132439777.jpeg)

看看到我们的手机上收到了短信。

![file](https://img-blog.csdnimg.cn/2019101413244054.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

可以看到短信服务是配置成功了的。整体来说没有我们想象中的那么复杂。

# 番外

好了，就说这么多啦，今天项目的代码也同步到github 上啦。
github地址：https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/20191014132440551.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)



