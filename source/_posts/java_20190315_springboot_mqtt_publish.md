---
layout:     post
title:      SpringBoot之MQTT消息发送
date:       2019-03-15 09:30:12
author:     liangtong
catalog: true
categories: Java
tags: mq

---


实现MQTT协议的中间件，目前使用的是Apache-Apollo服务器。
本文使用Gateway绑定的方式，进行消息发送。

#### SpringBoot

使用`idea`新建`Spring Initializr`工程，过程省略，使用`Maven`对项目依赖进行管理，配置过程省略。
完成后，在`pom`文件中加入以下依赖

```xml
        <!--MQTT Start-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-integration</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-mqtt</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--MQTT End-->
```

然后配置SpringBoot的文件`application.yml`

```yml
spring:
  mqtt:
    # publish 发布
    publish:
      #发布 - 用户名
      username: admin
      #发布 - 密码
      password: password
      #发布 - 服务器连接地址，如果有多个，用逗号隔开，如：tcp://127.0.0.1:61613,tcp://192.168.2.133:61613
      url: tcp://127.0.0.1:61613
      client:
        #发布 - 连接服务器默认客户端ID
        id: mqttId
      default:
        #发布 - 默认的消息推送主题，实际可在调用接口时指定
        topic: topic
```



 <!-- more -->



#### SpringBoot配置类

配置MQTT的消息推送配置类

```Java
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.IntegrationComponentScan;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.outbound.MqttPahoMessageHandler;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;

import java.util.Arrays;
import java.util.List;

/**
 * mqtt publish config
 */
@Configuration
@IntegrationComponentScan
public class MqttPublishConfig {

    @Value("${spring.mqtt.publish.username}")
    private String username;

    @Value("${spring.mqtt.publish.password}")
    private String password;

    @Value("${spring.mqtt.publish.url}")
    private String hostUrl;

    @Value("${spring.mqtt.publish.client.id}")
    private String clientId;

    @Value("${spring.mqtt.publish.default.topic}")
    private String defaultTopic;

    @Bean
    public MqttConnectOptions getMqttConnectOptions(){
        MqttConnectOptions mqttConnectOptions=new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        mqttConnectOptions.setKeepAliveInterval(2);
        List<String> hostList = Arrays.asList(hostUrl.trim().split(","));
        String[] serverURIs = new String[hostList.size()];
        hostList.toArray(serverURIs);
        mqttConnectOptions.setServerURIs(serverURIs);
        return mqttConnectOptions;
    }
    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setConnectionOptions(getMqttConnectOptions());
        return factory;
    }

    @Bean
    @ServiceActivator(inputChannel = "mqttOutboundChannel")
    public MessageHandler mqttOutbound() {
        MqttPahoMessageHandler messageHandler =  new MqttPahoMessageHandler(clientId,
                mqttClientFactory());
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(defaultTopic);
        return messageHandler;
    }
    @Bean
    public MessageChannel mqttOutboundChannel() {
        return new DirectChannel();
    }
}
```

#### Service
配置MqttGateway消息推送接口类，提供发布特定主题消息的能力。

```Java
import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.handler.annotation.Header;

@MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
public interface MqttGateway {
    void publishMqttMessageWithTopic(String data,
                                     @Header(MqttHeaders.TOPIC) String topic);
}
```

#### RestController
配置http接口，用于(或者测试)触发消息的发送。

```Java
import com.xxxxxxxxxxxxxxxxxxxxx.mqtt.service.MqttGateway;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.util.MimeTypeUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/mqtt")
public class MessageApiController {
    @Autowired
    private MqttGateway mqttGateway;

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RequestMapping(value = {"publish"}, method = {RequestMethod.POST}, produces = MimeTypeUtils.APPLICATION_JSON_VALUE)
    public void postPublishMessage(@RequestParam String message,
                                   @RequestParam(value = "topic", required = false) String topic) {
        logger.info("\n----------------------------START---------------------------\n" +
                "接收到发布请求:\ntopic:" + topic + "\nmessage:" + message +
                "\n-----------------------------END----------------------------");

        if (topic == null || topic.isEmpty()) {
            topic = "topic";
        }

        mqttGateway.publishMqttMessageWithTopic(message,
                topic);
    }
}
```

#### 测试
启动`SpringBootApplication` 。
使用`Postman`发送一条`topic`为`test-topic`的消息。

![postman.png](/post/java/20190315/postman.png)


控制台中，打印出以下info

![console.png](/post/java/20190315/console.png)

`Apollo`后台，我们可以看到一个主题为`test-topic`的记录

![apollo.png](/post/java/20190315/apollo.png)



大功告成！下一篇介绍SpringBoot集成MQTT消息订阅功能！


