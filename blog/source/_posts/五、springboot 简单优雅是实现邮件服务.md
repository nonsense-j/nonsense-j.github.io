layout: post
title: 五、springboot 简单优雅是实现邮件服务
author: QuellanAn
categories: 
  - springBoot
tags:
  - springboot
  - java
---

# 前言
spring boot 的项目放下小半个月没有更新了，终于闲下来可以开心的接着写啦。
之前我们配置好mybatis 多数据源的，接下来我们需要做一个邮件服务。比如你注册的时候，需要输入验证码来校验。这个验证码就可以通过邮件来发送。当然现在验证码大部分都是通过短信，单邮件有时候也是必不可少的。所以我们的spring架手架还是将邮件服务也搭建起来。下一篇将短信服务也整合进来。
好了，言归正传。搭建邮件服务没有接触可能会觉得很麻烦或者单机环境测试环境都实现不了。觉得没有邮件服务。其实我们个人使用的话，是可以做到的。qq邮箱，网易邮箱都可以的。我这里使用的是QQ邮箱。网上有很多相关的教程。

# 邮箱服务器准备
登录QQ邮箱，点击设置 -->账户 可以找到 下图这个。

![file](https://img-blog.csdnimg.cn/20191011182249343.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

需要开通 POP3/SMTP服务。开通这个后，会生成一个秘钥。这个秘钥我们待会会在项目中用到。拿小本本记下来哈哈。

# 添加依赖和配置
邮箱准备好了，我们就开始我们的项目吧。
首先在pom.xml 文件中添加依赖
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

然后在application.proteries 文件中添加配置，改成自己的邮箱。password 就是刚刚生成的那个秘钥。QQ邮箱的服务器地址是：smtp.qq.com  。网易的大家可以搜一下。
```
spring.mail.host=smtp.qq.com
spring.mail.username=1186154608@qq.com
spring.mail.password=abcdefgqazqaz
spring.mail.default-encoding=UTF-8

mail.from=1186154608@qq.com
```

![file](https://img-blog.csdnimg.cn/20191011182249633.jpeg)

# Service 层
配置信息都好了之后，我们就可以来使用啦。这里我们暂时没有涉及到数据库，就直接写Service层和controller 层。
在service 包下创建一个MailService 和MailServiceImpl 

![file](https://img-blog.csdnimg.cn/20191011182249899.jpeg)

MailServiceImpl 中代码
```
@Service
@Slf4j
public class MailServiceImpl implements MailService{
    @Autowired
    private JavaMailSender mailSender;
    @Value("${mail.from}")
    private String mailFrom;
    @Override
    public void sendSimpleMail(String mailTo) {
        SimpleMailMessage message=new SimpleMailMessage();
        message.setFrom(mailFrom);
        message.setTo(mailTo);
        message.setSubject("simple mail");
        message.setText("hello world");
        mailSender.send(message);
        log.info("邮件已经发送");
    }

}
```

这里我们就先简单的测试一下看看邮件能不能发送。mailFrom 是发件人，mailTo 是收件人。message.setSubject()设置邮件主题。message.setText()设置邮件内容。
 mailSender.send(message)是发送短信。
 
 # controller层
 我们创建一个MailController类。代码如下：
 ```
 @RestController
@RequestMapping("/mail")
public class MailController {
    @Autowired
    private MailService mailService;

    @RequestMapping(value = "/send",method = RequestMethod.GET)
    public String sendMail(@RequestParam(value = "userName")String userName){
        mailService.sendSimpleMail(userName);
        return "success";
    }
}
 ```
 可以看到就一个发送的接口。很简单，参数传过来接收人的邮箱就好了。
 
 # 测试
 到此为止，我们邮件服务的demo 就已经搭建好了。我们接下来测试测试一下。我们启动项目。然后调接口
 ```
 http://localhost:9090/zlflovemm/mail/send?userName=1303123974@qq.com
 ```
 
![file](https://img-blog.csdnimg.cn/20191011182250181.jpeg)

提示已经发送成功啦，我们进邮箱看下我们发送情况。可以看到是发送成功了。所以说明我们的邮件服务搭建成功了。

![file](https://img-blog.csdnimg.cn/20191011182250606.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

所以现在看来，springboot 集成邮件服务是非常简单的，配置邮件服务器，就可以直接使用啦。

# 发送附件
有时候我们发送邮件不仅仅发送内容，还需要发送附件，那怎么实现呢。其实也很简单。那些配置还是不变。我们在service 层。写一个sendMail方法。如下
```
@Override
    public void sendMail(String mailTo) {
        MimeMessage message=mailSender.createMimeMessage();
        MimeMessageHelper helper = null;
        try {
            helper = new MimeMessageHelper(message, true);
            helper.setFrom(mailFrom);
            helper.setTo(mailTo);
            helper.setSubject("simple mail");
            helper.setText("hello world", true);
            FileSystemResource file = new FileSystemResource(new File("E:\\myself\\test.xls"));
            String fileName = file.getFilename();
            helper.addAttachment(fileName, file);
            mailSender.send(message);
            log.info("邮件已经发送");
        } catch (MessagingException e) {
            log.error("{}",e);
        }
    }
```
可以看到和我们开始测试的时候，有一点不同。这里先
```
MimeMessage message=mailSender.createMimeMessage();
```
MimeMessage 比 SimpleMailMessage 功能更强大，可以发送附件，也可以将内容转成html 格式发送。所以一般实际使用的时候都使用MimeMessage。
另外发送附件，还需要借助MimeMessageHelper 。MimeMessageHelper是辅助MimeMessage的。
```
helper.setFrom(mailFrom);
helper.setTo(mailTo);
helper.setSubject("simple mail");
helper.setText("hello world", true);
```
这些和前面是一样的，发件人收件人，主题，内容。
helper.addAttachment()是添加附件的。

好了，接下我们测试一下。可以看到发送的邮件是有附件的。证明没问题。

![file](https://img-blog.csdnimg.cn/20191011182250966.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)

# 番外

好了，就说这么多啦，今天项目的代码也同步到github 上啦。
github地址：https://github.com/QuellanAn/zlflovemm

后续加油♡

欢迎大家关注个人公众号 "程序员爱酸奶"

分享各种学习资料，包含java，linux，大数据等。资料包含视频文档以及源码，同时分享本人及投递的优质技术博文。

如果大家喜欢记得关注和分享哟❤
![file](https://img-blog.csdnimg.cn/2019092616120288.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NzkwMDEx,size_16,color_FFFFFF,t_70)
