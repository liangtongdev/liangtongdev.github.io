---
layout:     post
title:      RabbitMQ 死信队列
date:       2020-09-16 10:54:13
author:     liangtong
catalog: true
categories: Java
tags: mq

---

​	上一章介绍了`RabbitMQ`在SpringBoot项目中简单队列的使用，介绍`RabbitMQ`的死信队列的使用。

​    为了保证订单业务的消息数据不丢失，需要使用到RabbitMQ的死信队列机制，当消息消费发生异常时，将消息投入死信队列中。`死信`是RabbitMQ中的一种消息机制   ,当消息被拒绝或者超出ttl，队列消息数量达到最大数量时，消息会成为死信。如果配置了死信队列信息，那么该消息将会被丢进死信队列中，如果没有配置，则该消息将会被丢弃。

   通过配置死信队列，可以让未正确处理的消息暂存到另一个队列中。有针对性的分析解决问题（恢复数据）等

### 使用步骤

+ 配置正常业务队列，绑定到业务交换机上
+ 为业务队列配置死信队列和路由key
+ 为死信交换机配置死信队列

死信队列并不是什么特殊的队列，只不过是绑定在死信交换机上的队列。类型可以是Direct、Fanout、Topic等。

### 环境配置

MAVEN依赖不需要更改，如下：

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

需要调整的是Resource配置文件，添加如下配置项：

`spring.rabbitmq.listener.type` 是设置队列`simple`模式。即简单的点对点消息模型，生产者向mq写消息，消费者监听mq，消费消息。
`spring.rabbitmq.listener.simple.default-requeue-rejected`表示被拒绝的消息是否重新入队列。
`spring.rabbitmq.listener.simple.acknowledge-mode` 表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto

```application.yml
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
    listener:
      type: simple
      simple:
        default-requeue-rejected: false
        acknowledge-mode: manual
```

### 项目开发

#### 配置队列

声明常量

```Java
public class MQConstant {
    /** Exchange 交换器/
    public static final String EXCHANGE_IM_NAME = "exchange.demo.im";
    public static final String EXCHANGE_DEAD_NAME = "exchange.demo.dead";
    / 路由定义 /
    public static final String ROUTING_KEY_DEAD_NAME = "routingkey.demo.dead";
    / 队列定义 **/
    public static final String QUEUE_IM_NAME = "queue.demo.im";
    public static final String QUEUE_DEAD_NAME = "queue.demo.dead";
}
```



RabbitMQ 交换机和队列的配置。需要注意的是，因为im队列绑定关系变更了，所以需要手动将之前的im队列删除。
此处需要注意的是即时队列与死信队列的路由绑定

```java
import static org.liangtong.example.rabbit.constant.MQConstant.*;

@Configuration
public class RabbitMQConfig {

    // 声明交换器
    @Bean("imExchange")
    public FanoutExchange imExchange() {
        return new FanoutExchange(EXCHANGE_IM_NAME);
    }
    @Bean("deadExchange")
    public DirectExchange deadExchange() {
        return new DirectExchange(EXCHANGE_DEAD_NAME);
    }

    // 声明队列
    @Bean("imQueue")
    public Queue imQueue() {
        Map<String, Object> args = new HashMap<>(2);
        //绑定死信队列
        args.put("x-dead-letter-exchange", EXCHANGE_DEAD_NAME);
        //当前队列的死信路由Key
        args.put("x-dead-letter-routing-key", ROUTING_KEY_DEAD_NAME);
        return QueueBuilder.durable(QUEUE_IM_NAME).withArguments(args).build();
    }
    @Bean("deadQueue")
    public Queue deadQueue() {
        return new Queue(QUEUE_DEAD_NAME);
    }

    // 绑定关系
    @Bean
    public Binding imBinding(@Qualifier("imQueue") Queue queue,
                             @Qualifier("imExchange") FanoutExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange);
    }
    @Bean
    public Binding deadBinding(@Qualifier("deadQueue") Queue queue,
                               @Qualifier("deadExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(ROUTING_KEY_DEAD_NAME);
    }
}
```

#### 消费者

为了方便测试，我们修改原消费者代码，在即时消息消费过程中，抛出异常

```Java
@Slf4j
@Component
public class ImConsumer {

    @RabbitListener(queues = QUEUE_IM_NAME)
    public void receiveIm(Message message,
                          Channel channel) throws IOException {
        log.info("收到即时推送消息: " + message);
        boolean ack = true;
        Exception exception = null;
        try {
            long deliveryTag = message.getMessageProperties().getDeliveryTag();
            if (deliveryTag % 2 == 1){
                throw new RuntimeException("发生异常！");
            }
        } catch (Exception e) {
            ack = false;
            exception = e;
        } finally {

        }
        if (!ack) {
            log.error("消息消费发生异常，error msg:{}", exception.getMessage(), exception);
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        } else {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        }
    }
}
```

死信消息消费者

```Java
@Slf4j
@Component
public class DeadConsumer {
    @RabbitListener(queues = QUEUE_DEAD_NAME)
    public void receiveDead(Message message,
                            Channel channel) throws IOException {
        log.info("收到死信消息: " + message);
        log.info("死信消息properties：{}", message.getMessageProperties());
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

#### 消息生产者

消息生产者不变，直接放代码

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
2020-09-16 10:22:37.714  INFO 53522 --- [nio-8081-exec-1] o.l.example.rabbit.producer.MsgProducer  : 发送消息：dead msg
2020-09-16 10:22:37.727  INFO 53522 --- [ntContainer#1-1] o.l.example.rabbit.consumer.ImConsumer   : 收到即时推送消息: (Body:'dead msg' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.im, receivedRoutingKey=queue.demo.im, deliveryTag=1, consumerTag=amq.ctag-l1gNT9yx_eVUMIzS4VqYIA, consumerQueue=queue.demo.im])
2020-09-16 10:22:37.731 ERROR 53522 --- [ntContainer#1-1] o.l.example.rabbit.consumer.ImConsumer   : 消息消费发生异常，error msg:发生异常！

java.lang.RuntimeException: 发生异常！
    at org.liangtong.example.rabbit.consumer.ImConsumer.receiveIm(ImConsumer.java:31) ~[classes/:na]
    ......

2020-09-16 10:22:37.732  INFO 53522 --- [ntContainer#0-1] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'dead msg' MessageProperties [headers={x-first-death-exchange=exchange.demo.im, x-death=[{reason=rejected, count=1, exchange=exchange.demo.im, time=Wed Sep 16 10:22:37 CST 2020, routing-keys=[queue.demo.im], queue=queue.demo.im}], x-first-death-reason=rejected, x-first-death-queue=queue.demo.im}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-zzH3g1u0xhhbSFcFtrdNdw, consumerQueue=queue.demo.dead])
2020-09-16 10:22:37.733  INFO 53522 --- [ntContainer#0-1] o.l.e.rabbit.consumer.DeadConsumer       : 死信消息properties：MessageProperties [headers={x-first-death-exchange=exchange.demo.im, x-death=[{reason=rejected, count=1, exchange=exchange.demo.im, time=Wed Sep 16 10:22:37 CST 2020, routing-keys=[queue.demo.im], queue=queue.demo.im}], x-first-death-reason=rejected, x-first-death-queue=queue.demo.im}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-zzH3g1u0xhhbSFcFtrdNdw, consumerQueue=queue.demo.dead]
```



至此，死信队列功能已经实现，下一章节介绍如何使用`RabbitMQ延迟队列`。




