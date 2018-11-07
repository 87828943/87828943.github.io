---
title: SpringBoot 5.邮件服务
date: 2018-06-22 16:17:38
tags: [springboot]
categories: springboot
---

# SpringBoot 邮件发送

> 目的为了实现忘记密码发送邮件，点击邮件链接验证重置密码(申请邮箱需要开启POP3/IMAP/SMTP服务，如用QQ邮箱密码为授权码不是登录密码)，下面以163邮箱为例。


<!-- more -->

## 配置

1. pom
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2. 配置文件
```
spring.mail.host=smtp.163.com //邮箱服务器地址
spring.mail.username=xxx@163.com //用户名
spring.mail.password=xxx //密码
spring.mail.default-encoding=UTF-8
spring.mail.port=587

spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
spring.mail.properties.mail.smtp.socketFactory.port=465
```

3. 实现
```java
@Autowired
private JavaMailSender javaMailSender;

//发送人
@Value("${spring.mail.username}")
private String fromEmail;

/*
* 发送普通邮件(收件人，主题，内容)
* */
boolean sendEmail(String toEmail,String subject,String content){
    try {
        SimpleMailMessage smm = new SimpleMailMessage();
        smm.setFrom(fromEmail);
        smm.setTo(toEmail);
        /* String[] adds = {"xxx@qq.com","yyy@qq.com"}; //同时发送给多人
        smm.setTo(adds);*/
        smm.setSubject(subject);//设置主题
        smm.setText(content);//设置内容
        javaMailSender.send(smm);//执行发送邮件
        logger.info("html email send success! toEmail:[{}]",toEmail);
    }catch (Exception e){
        logger.error("sendEmail error,toEmail:[{}]",toEmail,e);
        return Boolean.FALSE;
    }
    return Boolean.TRUE;
}

/*
* 发送html邮件(收件人，主题，内容)
* */
boolean sendHtmlEmail(String toEmail,String subject,String content){
    try {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        //true表示需要创建一个multipart message
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
        helper.setFrom(fromEmail);
        helper.setTo(toEmail);
        helper.setSubject(subject);
        helper.setText(content, true);
        javaMailSender.send(mimeMessage);
        logger.info("html email send success! toEmail:[{}]",toEmail);
    }catch (Exception e){
        logger.error("sendEmail error,toEmail:[{}]",toEmail,e);
        return Boolean.FALSE;
    }
    return Boolean.TRUE;
}
```

## 使用
```java
@ResponseBody
@RequestMapping(value = "/forgotPassword",method = RequestMethod.POST)
public void forgotPassword(String email,HttpServletRequest request){
    User user = userService.findByEmail(email);
    String key = MD5Util.encrypt(user.getId()+"_"+UUID.randomUUID().toString()+"_"+user.getEmail()); // 密钥

    //设置秘钥10分钟有效期
    redisUtil.set(String.valueOf(user.getId()),key,10L,MINUTES);

    String url = "http://" + request.getServerName() + ":"+ request.getServerPort()+ request.getContextPath() + "/user/newPassword"+ "?email="+user.getEmail()+"&key="+key;
    String content ="<br/>请点击下方链接重置您的账号密码↓↓↓↓↓ <br/>"+"<a href='"+url+"' ><pre>"+url+"</pre></a><br/>";

    sendHtmlEmail(email,"[重置密码]",content);
}
```

## 其他

除了发送普通邮件html邮件，还能发送带图片，附件，或是通过模板来发送邮件，不一一列举。