---
layout:     post
title:      RabbitMQ消息队列
date:       2020-09-15 20:54:13
author:     liangtong
catalog: true
categories: Java
tags: mq

---

​	上一章介绍了`RabbitMQ`的安装与配置，本章基于SpringBoot项目，介绍`RabbitMQ`的消息队列的简单使用。



### 环境配置

使用 `IDEA` 工具新建Maven项目(或者直接创建SpringBoot项目)，然后在其pom文件中添加依赖如下（主要是`spring-boot-starter-amqp`）：

```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.16.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
            <scope>provided</scope>
        </dependency>
        <!--AMQP Start-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <!--AMQP End-->
    </dependencies>
```



在`resources`目录下，添加资源配置文件`application.yml`。添加服务端口号和rabbitmq相关配置，具体如下：

```xml
# service
server:
  port: 8081

#spring
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
```

### 项目开发

采用直连的方式进行即时队列消息的生产与消费

#### 配置队列

声明常量
```Java
public class MQConstant {
    /** Exchange 交换器**/
    public static final String EXCHANGE_IM_NAME = "exchange.demo.im";
    /** 队列定义 **/
    public static final String QUEUE_IM_NAME = "queue.demo.im";
}
```

RabbitMQ 交换机和队列的配置

```Java
import static org.liangtong.example.rabbit.constant.MQConstant.*;

@Configuration
public class RabbitMQConfig {

    // 声明交换器
    @Bean("imExchange")
    public FanoutExchange imExchange() {
        return new FanoutExchange(EXCHANGE_IM_NAME);
    }

    // 声明队列
    @Bean("imQueue")
    public Queue imQueue() {
        return new Queue(QUEUE_IM_NAME);
    }

    // 绑定关系
    @Bean
    public Binding imBinding(@Qualifier("imQueue") Queue queue,
                             @Qualifier("imExchange") FanoutExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange);
    }
}
```

#### 即时消息消费者

```Java
@Slf4j
@Component
public class ImConsumer {

    @RabbitListener(queues = QUEUE_IM_NAME)
    public void receiveIm(Message message,
                          Channel channel) throws IOException {
        log.info("收到即时推送消息: " + message);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
```

#### 消息生产者

```Java
@Slf4j
@Component
public class MsgProducer {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendMsg(String msg) {
        log.info("发送消息：{}", msg);
        rabbitTemplate.convertAndSend(EXCHANGE_IM_NAME,
                QUEUE_IM_NAME,
                msg,
                new CorrelationData()
        );
    }
}
```

### 测试

创建Webcontroller来进行功能测试，代码如下：

```Java
@RequestMapping("rabbitmq")
@RestController
public class RabbitMsgController {
    @Autowired
    private MsgProducer sender;

    @RequestMapping("send")
    public void sendMsg(String msg){
        sender.sendMsg(msg);
    }
}
```

启动SpringBoot项目，然后浏览器输入`http://localhost:8081/rabbitmq/send?msg=hello`测试：

```bash
2020-09-16 09:54:32.669  INFO 47620 --- [nio-8081-exec-1] o.l.example.rabbit.producer.MsgProducer  : 发送消息：hello
2020-09-16 09:54:32.682  INFO 47620 --- [ntContainer#0-1] o.l.example.rabbit.consumer.ImConsumer   : 收到即时推送消息: (Body:'hello' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.im, receivedRoutingKey=queue.demo.im, deliveryTag=1, consumerTag=amq.ctag-XhooL-hLVgm4a66F8AHqoQ, consumerQueue=queue.demo.im])
```



至此，简单的消息队列功能已经实现，下一章节介绍如何使用`RabbitMQ死信队列`。

