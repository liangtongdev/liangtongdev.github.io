---
layout:     post
title:      SpringBoot之MQTT消息订阅
date:       2019-03-16 11:30:12
author:     liangtong
catalog: true
categories: Java
tags: mqtt

---


实现MQTT协议的中间件，目前使用的是Apache-Apollo服务器。
接上篇，本文介绍如何在SpringBoot上集成MQTT消息订阅功能。

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
    # subscribe 订阅
    subscribe:
      #订阅 - 用户名
      username: admin
      #订阅 - 密码
      password: password
      #订阅 - 服务器连接地址，如果有多个，用逗号隔开，如：tcp://127.0.0.1:61613,tcp://192.168.2.133:61613
      url: tcp://127.0.0.1:61613,tcp://192.168.2.133:61614
      client:
        #订阅 - 连接服务器默认客户端ID
        id: mqttId-1
      default:
        #订阅 - 默认的消息推送主题，实际可在调用接口时指定
        topic: hello,world
        #订阅 - 连接超时
      completionTimeout: 3000
```

在配置文件中，我们设置两个订阅主题`hello`和`world`

 <!-- more -->



#### SpringBoot配置类

配置MQTT的消息订阅配置类

```Java
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.IntegrationComponentScan;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.support.DefaultPahoMessageConverter;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;

import java.util.Arrays;
import java.util.List;

/**
 * mqtt subscribe
 */
@Configuration
@IntegrationComponentScan
public class MqttSubscribeConfig {

    @Value("${spring.mqtt.subscribe.username}")
    private String username;

    @Value("${spring.mqtt.subscribe.password}")
    private String password;

    @Value("${spring.mqtt.subscribe.url}")
    private String hostUrl;

    @Value("${spring.mqtt.subscribe.client.id}")
    private String clientId;

    @Value("${spring.mqtt.subscribe.default.topic}")
    private String defaultTopic;

    @Value("${spring.mqtt.subscribe.completionTimeout}")
    private int completionTimeout ;   //连接超时

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Bean
    public MqttConnectOptions getReceiverMqttConnectOptions(){
        MqttConnectOptions mqttConnectOptions=new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        List<String> hostList = Arrays.asList(hostUrl.trim().split(","));
        String[] serverURIs = new String[hostList.size()];
        hostList.toArray(serverURIs);
        mqttConnectOptions.setServerURIs(serverURIs);
        mqttConnectOptions.setKeepAliveInterval(2);
        return mqttConnectOptions;
    }
    @Bean
    public MqttPahoClientFactory receiverMqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setConnectionOptions(getReceiverMqttConnectOptions());
        return factory;
    }

    //接收通道
    @Bean
    public MessageChannel mqttInputChannel() {
        return new DirectChannel();
    }

    //配置client,监听的topic
    @Bean
    public MessageProducer inbound() {
        List<String> topicList = Arrays.asList(defaultTopic.trim().split(","));
        String[] topics = new String[topicList.size()];
        topicList.toArray(topics);

        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter(clientId,
                        receiverMqttClientFactory(),
                        topics);
        adapter.setCompletionTimeout(completionTimeout);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        adapter.setOutputChannel(mqttInputChannel());
        return adapter;
    }

    //通过通道获取数据
    @Bean
    @ServiceActivator(inputChannel = "mqttInputChannel")
    public MessageHandler handler() {
        return new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                String topic = message.getHeaders().get("mqtt_receivedTopic").toString();
                String msg = message.getPayload().toString();
                logger.info("\n----------------------------START---------------------------\n" +
                        "接收到订阅消息:\ntopic:" + topic + "\nmessage:" + msg +
                        "\n-----------------------------END----------------------------");
            }
        };
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


#### 测试
启动`SpringBootApplication` 。
使用`Postman`发送一条`topic`为`test`的消息和一条`topic`为`hello`的消息。
控制台中，打印出以下info

![console.png](https://upload-images.jianshu.io/upload_images/16014538-fef38f0ff2548c40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

`Apollo`后台，我们可以看到以下记录

![apollo.png](https://upload-images.jianshu.io/upload_images/16014538-84a7abe4813c9a7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)



#### 多客户端
将`SpringBoot`项目打成`jar`包，然后运行。
之后使用`Postman`发送一条`topic`为`hello`的消息。可以看到两个订阅端都收到了消息

![multi-client-console.png](https://upload-images.jianshu.io/upload_images/16014538-4da86f22c6f16b39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

`Apollo`后台，我们可以看到consumer的数量已经变更为2

![multi-client-apollo.png](https://upload-images.jianshu.io/upload_images/16014538-91b1fa6bbfffc938.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)



大功告成！


