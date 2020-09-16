---
layout:     post
title:      RabbitMQ 延时队列
date:       2020-09-16 11:54:13
author:     liangtong
catalog: true
categories: Java
tags: mq

---

​	上一章介绍了`RabbitMQ`中死信队列的使用，接下来介绍RabbitMQ中延时队列的使用，**包括队列TTL，消息TTL和RabbitMQ延迟插件。**


![定时任务](/post/java/20200916/timer.jpg)

### 使用场景

延时队列在需要延时处理的场景下非常有用，队列中的元素都是带有时间属性的， **普通队列的元素按照队列的次序依次取出处理；延时队列中的元素则按照指定时间被取出和处理。** 

+ 订单10分钟内未支付则自动取消。
+ 店铺10天没有更新，发消息提醒。
+ 账单一周内未支付，自动支付。
+ 新用户注册3天内没登录，短信提醒。
+ 预约会议后，在预定的时间开始前15分钟通知参会人员。
+ 发送定时消息。

这些场景都有一个共同点，就是跟时间有关。理想情况下，通过定时任务轮询数据可以实现功能；但随着数据量的增加，轮询会带来巨大的性能问题，同时轮询的方式处理起来很不优雅。这时候延时队列就派上用场了。


### RabbitMQ中的TTL

**TTL是RabbitMQ中一个消息或者队列的属性，表明一条消息或者队列中的所有消息的最大存活时间（毫秒）**。
即如果一条消息设置了TTL，或者进入了设置TTL属性的队列，那么这条消息如果在TTL时限内未被消费，则会成为死信；如果两者同时设置了TTL，那么较小TTL会被使用。

**在消息属性上设置TTL的方式，消息可能并不会按时“死亡“，RabbitMQ只会检查第一个消息是否过期，如果过期则丢到死信队列，索引如果第一个消息的延时时长很长，而第二个消息的延时时长很短，则第二个消息并不会优先得到执行。**

队列TTL的设置，实在创建队列的时候，设置队列的`x-message-ttl`属性，例如：

```Java
    Map<String, Object> args = new HashMap<>(3);
    args.put("x-message-ttl", 60000);//设置60秒ttl
    return QueueBuilder.durable(QUEUE_DELAY_FIX_NAME).withArguments(args).build();
```


### 队列TTL + 死信队列

通过给队列配置TTL，配合死信队列来实现消息的延时。**因为队列的TTL固定，所以这种如果有新的时间需求，则需要增加队列。** 这种适用于固定延时时间的场景。


#### 配置队列

声明常量

```Java
public class MQConstant {
    /** Exchange 交换器/
    public static final String EXCHANGE_DELAY_FIX_NAME = "exchange.demo.fix.delay";
    public static final String EXCHANGE_DEAD_NAME = "exchange.demo.dead";
    / 路由定义 /
    public static final String ROUTING_KEY_DELAY_FIX_1_NAME = "routingkey.demo.fix.1.delay";
    public static final String ROUTING_KEY_DEAD_NAME = "routingkey.demo.dead";
    / 队列定义 **/
    public static final String QUEUE_DELAY_FIX_1_NAME = "queue.demo.fix.1.delay";
    public static final String QUEUE_DEAD_NAME = "queue.demo.dead";
}
```

队列路由与交换器绑定

```java
import static org.liangtong.example.rabbit.constant.MQConstant.*;

@Configuration
public class RabbitMQConfig {

    // 声明交换器
    @Bean("fixDelayExchange")
    public DirectExchange fixDelayExchange() {
        return new DirectExchange(EXCHANGE_DELAY_FIX_NAME);
    }
    @Bean("deadExchange")
    public DirectExchange deadExchange() {
        return new DirectExchange(EXCHANGE_DEAD_NAME);
    }

    // 声明队列
    @Bean("fixDelayQueue1")
    public Queue fixDelayQueue1() {
        Map<String, Object> args = new HashMap<>(3);
        //绑定死信队列
        args.put("x-dead-letter-exchange", EXCHANGE_DEAD_NAME);
        //当前队列的死信路由Key
        args.put("x-dead-letter-routing-key", ROUTING_KEY_DEAD_NAME);
        // x-message-ttl  声明队列的TTL
        args.put("x-message-ttl", 60000);
        return QueueBuilder.durable(QUEUE_DELAY_FIX_1_NAME).withArguments(args).build();
    }
    @Bean("deadQueue")
    public Queue deadQueue() {
        return new Queue(QUEUE_DEAD_NAME);
    }

    // 绑定关系
    @Bean
    public Binding fixDelay1Binding(@Qualifier("fixDelayQueue1") Queue queue,
                                   @Qualifier("fixDelayExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(ROUTING_KEY_DELAY_FIX_1_NAME);
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

#### 死信消费者

此处不再配置延时消费者，而是例如TTL过期直接进消费队列机制，仅配置死信消息消费者

```Java
@Slf4j
@Component
public class DeadConsumer {
    @RabbitListener(queues = QUEUE_DEAD_NAME)
    public void receiveDead(Message message,
                            Channel channel) throws IOException {
        log.info("收到死信消息: " + message);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

#### 消息生产者

消息生产者

```Java
@Slf4j
@Component
public class MsgProducer {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendFixDelayMsg1(String msg){
        log.info("发送延时消息：{}", msg);
        rabbitTemplate.convertAndSend(EXCHANGE_DELAY_FIX_NAME,
                ROUTING_KEY_DELAY_FIX_1_NAME,
                msg,
                new CorrelationData()
        );
    }
}
```

#### 测试

创建Webcontroller来进行功能测试，代码如下：

```Java
@RequestMapping("rabbitmq")
@RestController
public class RabbitMsgController {
    @Autowired
    private MsgProducer sender;

    @RequestMapping("fixDelay1")
    public void sendFixDelay1Msg(String msg){
        sender.sendFixDelayMsg1(msg);
    }
}
```

启动SpringBoot项目，然后浏览器输入`http://localhost:8081/rabbitmq/fixDelay1?msg=Hello`和`http://localhost:8081/rabbitmq/fixDelay1?msg=梁通`测试：

```bash
2020-09-16 12:45:59.686  INFO 84248 --- [nio-8081-exec-5] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：Hello 
2020-09-16 12:46:02.521  INFO 84248 --- [nio-8081-exec-6] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：梁通 
2020-09-16 12:46:59.690  INFO 84248 --- [ntContainer#0-5] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'Hello' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 12:46:59 CST 2020, routing-keys=[queue.demo.fix.delay], queue=queue.demo.fix.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-V5GgFrP46_kt4CxtQXt9_w, consumerQueue=queue.demo.dead])
2020-09-16 12:47:02.524  INFO 84248 --- [ntContainer#0-6] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'梁通' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 12:47:02 CST 2020, routing-keys=[queue.demo.fix.delay], queue=queue.demo.fix.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-dd6HVYjD0xs7voSVu2o-tg, consumerQueue=queue.demo.dead])
```
可以看到，消息在经过60秒后，被消费掉。


### 消息TTL + 死信队列

在队列上设置TTL时，延时时间固定。还有一种方案，就是设置任意延时时长的消息，然后设置消息的TTL。

```Java
public class MQConstant {
    /** Exchange 交换器/
    public static final String EXCHANGE_DELAY_FIX_NAME = "exchange.demo.fix.delay";
    public static final String EXCHANGE_DEAD_NAME = "exchange.demo.dead";
    / 路由定义 /
    public static final String ROUTING_KEY_DELAY_FIX_1_NAME = "routingkey.demo.fix.1.delay";
    public static final String ROUTING_KEY_DELAY_FIX_2_NAME = "routingkey.demo.fix.2.delay";
    public static final String ROUTING_KEY_DEAD_NAME = "routingkey.demo.dead";
    / 队列定义 **/
    public static final String QUEUE_DELAY_FIX_1_NAME = "queue.demo.fix.1.delay";
    public static final String QUEUE_DELAY_FIX_2_NAME = "queue.demo.fix.2.delay";
    public static final String QUEUE_DEAD_NAME = "queue.demo.dead";
}
```
#### 队列配置
不设置队列TTL相关代码

```Java
    // 声明队列
    @Bean("fixDelayQueue2")
    public Queue fixDelayQueue2() {
        Map<String, Object> args = new HashMap<>(3);
        //绑定死信队列
        args.put("x-dead-letter-exchange", EXCHANGE_DEAD_NAME);
        //当前队列的死信路由Key
        args.put("x-dead-letter-routing-key", ROUTING_KEY_DEAD_NAME);
        return QueueBuilder.durable(QUEUE_DELAY_FIX_2_NAME).withArguments(args).build();
    }

    // 绑定关系
    @Bean
    public Binding fixDelay2Binding(@Qualifier("fixDelayQueue2") Queue queue,
                                    @Qualifier("fixDelayExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(ROUTING_KEY_DELAY_FIX_2_NAME);
    }
```

#### 消息生产者

设置消息过期时间

```Java
    public void sendFixDelayMsg2(String msg, Integer delay){
        log.info("发送延时消息：{} 延时{}秒", msg, delay);
        rabbitTemplate.convertAndSend(EXCHANGE_DELAY_FIX_NAME,
                ROUTING_KEY_DELAY_FIX_2_NAME,
                msg,
                message -> {
                    message.getMessageProperties().setExpiration(String.valueOf(delay * 1000) );
                    return message;
                }
        );
    }
```

#### 测试代码
```Java
    @RequestMapping("fixDelay2")
    public void sendFixDelay2Msg(String msg, Integer delay){
        sender.sendFixDelayMsg2(msg, delay);
    }
```

启动SpringBoot项目，然后浏览器输入`http://localhost:8081/rabbitmq/fixDelay2?测试10&delay=10`和`http://localhost:8081/rabbitmq/fixDelay2?测试20&delay=20`和`http://localhost:8081/rabbitmq/fixDelay2?测试30&delay=30`测试：

```bash
2020-09-16 13:38:13.371  INFO 94573 --- [nio-8081-exec-5] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试10 延时10秒
2020-09-16 13:38:18.863  INFO 94573 --- [nio-8081-exec-6] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试20 延时20秒
2020-09-16 13:38:23.374  INFO 94573 --- [ntContainer#0-3] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'测试10' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, original-expiration=10000, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 13:38:23 CST 2020, routing-keys=[routingkey.demo.fix.2.delay], queue=queue.demo.fix.2.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.2.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-xFbTWO_LXJEYVM52clOHjQ, consumerQueue=queue.demo.dead])
2020-09-16 13:38:24.206  INFO 94573 --- [nio-8081-exec-7] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试30 延时30秒
2020-09-16 13:38:24.379  INFO 94573 --- [ntContainer#0-3] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@162a6c67: tags=[[amq.ctag-xFbTWO_LXJEYVM52clOHjQ]], channel=Cached Rabbit Channel: AMQChannel(amqp://liangtong@127.0.0.1:5672/,2), conn: Proxy@1e95b653 Shared Rabbit Connection: SimpleConnection@6d8796c1 [delegate=amqp://liangtong@127.0.0.1:5672/, localPort= 56297], acknowledgeMode=AUTO local queue size=0
2020-09-16 13:38:38.866  INFO 94573 --- [ntContainer#0-4] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'测试20' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, original-expiration=20000, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 13:38:38 CST 2020, routing-keys=[routingkey.demo.fix.2.delay], queue=queue.demo.fix.2.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.2.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-yGbXcSqjA7G5NbSU90ScoQ, consumerQueue=queue.demo.dead])
2020-09-16 13:38:39.870  INFO 94573 --- [ntContainer#0-4] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@7571d97c: tags=[[amq.ctag-yGbXcSqjA7G5NbSU90ScoQ]], channel=Cached Rabbit Channel: AMQChannel(amqp://liangtong@127.0.0.1:5672/,3), conn: Proxy@1e95b653 Shared Rabbit Connection: SimpleConnection@6d8796c1 [delegate=amqp://liangtong@127.0.0.1:5672/, localPort= 56297], acknowledgeMode=AUTO local queue size=0
2020-09-16 13:38:54.209  INFO 94573 --- [ntContainer#0-5] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'测试30' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, original-expiration=30000, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 13:38:54 CST 2020, routing-keys=[routingkey.demo.fix.2.delay], queue=queue.demo.fix.2.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.2.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-1K-YeHnNpqh6utswjyXIuQ, consumerQueue=queue.demo.dead])
2020-09-16 13:38:55.211  INFO 94573 --- [ntContainer#0-5] o.s.a.r.l.SimpleMessageListenerContainer : Restarting Consumer@340b6985: tags=[[amq.ctag-1K-YeHnNpqh6utswjyXIuQ]], channel=Cached Rabbit Channel: AMQChannel(amqp://liangtong@127.0.0.1:5672/,3), conn: Proxy@1e95b653 Shared Rabbit Connection: SimpleConnection@6d8796c1 [delegate=amqp://liangtong@127.0.0.1:5672/, localPort= 56297], acknowledgeMode=AUTO local queue size=0
```

可以看到，发送的消息，再经过特定的ttl时间后，被消费掉。

别高兴太早，前边介绍过，在消息上添加TTL时，消息可能不会按时消亡。RabbitMQ只会检查第一个消息是否过期，如果过期则丢到死信队列，索引如果第一个消息的延时时长很长，而第二个消息的延时时长很短，则第二个消息并不会优先得到执行。然后我们测试下：

浏览器输入`http://localhost:8081/rabbitmq/fixDelay2?测试30&delay=30`和`http://localhost:8081/rabbitmq/fixDelay2?测试10&delay=10`测试：

```bash
2020-09-16 13:44:35.203  INFO 94573 --- [io-8081-exec-10] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试30 延时30秒
2020-09-16 13:44:39.832  INFO 94573 --- [nio-8081-exec-1] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试10 延时10秒
2020-09-16 13:45:05.206  INFO 94573 --- [ntContainer#0-6] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'测试30' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, original-expiration=30000, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 13:45:05 CST 2020, routing-keys=[routingkey.demo.fix.2.delay], queue=queue.demo.fix.2.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.2.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=1, consumerTag=amq.ctag-ZbAeduIZW8Pd_RBTcZPDsg, consumerQueue=queue.demo.dead])
2020-09-16 13:45:05.207  INFO 94573 --- [ntContainer#0-6] o.l.e.rabbit.consumer.DeadConsumer       : 收到死信消息: (Body:'测试10' MessageProperties [headers={x-first-death-exchange=exchange.demo.fix.delay, x-death=[{reason=expired, original-expiration=10000, count=1, exchange=exchange.demo.fix.delay, time=Wed Sep 16 13:45:05 CST 2020, routing-keys=[routingkey.demo.fix.2.delay], queue=queue.demo.fix.2.delay}], x-first-death-reason=expired, x-first-death-queue=queue.demo.fix.2.delay}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.dead, receivedRoutingKey=routingkey.demo.dead, deliveryTag=2, consumerTag=amq.ctag-ZbAeduIZW8Pd_RBTcZPDsg, consumerQueue=queue.demo.dead])
```
结果显示，即使第二条消息的TTL较短（10秒），消息在第一条消息成为死信后才被消费。（**惊不惊喜，意不意外！**）


### 延时插件

从上边测试可以看出，RabbitMQ在消息粒度上的延迟无法满足个性化的延时需求。那么这种消息粒度上的延时问题如何解决呢。
接下来介绍通过安装RabbitMQ插件的形式来完美解决。
插件下载地址：https://www.rabbitmq.com/community-plugins.html

![rabbit_delay_message_exchange.png](/post/java/20200916/rabbit_delay_message_exchange.png)

这里，我们需要找到 `rabbitmq_delayed_message_exchange` 插件，下载后，将`rabbitmq_delayed_message_exchange-3.8.0.ez`文件放到`sbin`同级的`plugins`目录下，然后切换到`sbin`目录，安装插件并重启服务

```bash
> ./rabbitmq-plugins enable rabbitmq_delayed_message_exchange
> ./rabbitmqctl stop
> ./rabbitmq-server
```

![rabbitctl_restart.png](/post/java/20200916/rabbitctl_restart.png)

#### 配置队列

设置常量
```Java
public class MQConstant {

    /** Exchange 交换器**/
    public static final String EXCHANGE_DELAY_NAME = "exchange.demo.delay";
    public static final String EXCHANGE_DEAD_NAME = "exchange.demo.dead";

    /** 路由定义 **/
    public static final String ROUTING_KEY_DELAY_NAME = "routingkey.demo.delay";
    public static final String ROUTING_KEY_DEAD_NAME = "routingkey.demo.dead";

    /** 队列定义 **/
    public static final String QUEUE_DELAY_NAME = "queue.demo.delay";
    public static final String QUEUE_DEAD_NAME = "queue.demo.dead";
}
```

队列路由与交换器绑定

```Java
@Configuration
public class RabbitMQConfig {

    // 声明交换器
    @Bean("delayExchange")
    public CustomExchange delayExchange() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-delayed-type", "direct");
        return new CustomExchange(EXCHANGE_DELAY_NAME, "x-delayed-message", true, false, args);
    }

    @Bean("deadExchange")
    public DirectExchange deadExchange() {
        return new DirectExchange(EXCHANGE_DEAD_NAME);
    }

    // 声明队列
    @Bean("delayQueue")
    public Queue delayQueue() {
        Map<String, Object> args = new HashMap<>(2);
        //绑定死信队列
        args.put("x-dead-letter-exchange", EXCHANGE_DEAD_NAME);
        //当前队列的死信路由Key
        args.put("x-dead-letter-routing-key", ROUTING_KEY_DEAD_NAME);
        return QueueBuilder.durable(QUEUE_DELAY_NAME).withArguments(args).build();
    }

    @Bean("deadQueue")
    public Queue deadQueue() {
        return new Queue(QUEUE_DEAD_NAME);
    }

    // 绑定关系
    @Bean
    public Binding delayBinding(@Qualifier("delayQueue") Queue queue,
                                @Qualifier("delayExchange") CustomExchange exchange) {
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(ROUTING_KEY_DELAY_NAME).noargs();
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
死信消费者不变，此处增加延时队列消费者

```Java
@Slf4j
@Component
public class DelayConsumer {

    @RabbitListener(queues = QUEUE_DELAY_NAME)
    public void receiveDelay(Message message,
                             Channel channel) throws IOException {
        log.info("收到延时消息: " + message);

        boolean ack = true;
        Exception exception = null;
        try {
            //处理延时消息
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

#### 生产者

生产者设置延时时间

```Java
    public void delayMsg(String msg, Integer delay){
        log.info("发送延时消息：{} 延时{}秒", msg, delay);
        rabbitTemplate.convertAndSend(EXCHANGE_DELAY_NAME,
                ROUTING_KEY_DELAY_NAME,
                msg,
                message -> {
                    message.getMessageProperties().setDelay(delay * 1000);
                    return message;
                }
        );
    }
```

#### 测试代码
```Java
    @RequestMapping("delay")
    public void sendDelay(String msg, Integer delay){
        sender.delayMsg(msg, delay);
    }
```

启动SpringBoot项目，然后浏览器输入`http://localhost:8081/rabbitmq/delay?测试30&delay=30`、`http://localhost:8081/rabbitmq/delay?测试20&delay=20`、`http://localhost:8081/rabbitmq/delay?测试50&delay=50`和`http://localhost:8081/rabbitmq/delay?测试10&delay=10`测试：

```bash
2020-09-16 14:09:35.903  INFO 2955 --- [nio-8081-exec-1] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试30 延时30秒
2020-09-16 14:09:43.152  INFO 2955 --- [nio-8081-exec-2] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试20 延时20秒
2020-09-16 14:09:48.832  INFO 2955 --- [nio-8081-exec-3] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试50 延时50秒
2020-09-16 14:09:53.086  INFO 2955 --- [nio-8081-exec-4] o.l.example.rabbit.producer.MsgProducer  : 发送延时消息：测试10 延时10秒
2020-09-16 14:10:03.095  INFO 2955 --- [ntContainer#1-1] o.l.e.rabbit.consumer.DelayConsumer      : 收到延迟推送消息: (Body:'测试10' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.delay, receivedRoutingKey=routingkey.demo.delay, receivedDelay=10000, deliveryTag=1, consumerTag=amq.ctag-rvB4gUM_H9dHVWBy882mUw, consumerQueue=queue.demo.delay])
2020-09-16 14:10:04.103  INFO 2955 --- [ntContainer#1-2] o.l.e.rabbit.consumer.DelayConsumer      : 收到延迟推送消息: (Body:'测试20' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.delay, receivedRoutingKey=routingkey.demo.delay, receivedDelay=20000, deliveryTag=1, consumerTag=amq.ctag-VU9VXQa2OT78oceGFH0xcA, consumerQueue=queue.demo.delay])
2020-09-16 14:10:09.114  INFO 2955 --- [ntContainer#1-3] o.l.e.rabbit.consumer.DelayConsumer      : 收到延迟推送消息: (Body:'测试30' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.delay, receivedRoutingKey=routingkey.demo.delay, receivedDelay=30000, deliveryTag=1, consumerTag=amq.ctag-6bzse22_iQ8XRKBv_Elxqg, consumerQueue=queue.demo.delay])
2020-09-16 14:10:38.836  INFO 2955 --- [ntContainer#1-4] o.l.e.rabbit.consumer.DelayConsumer      : 收到延迟推送消息: (Body:'测试50' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=exchange.demo.delay, receivedRoutingKey=routingkey.demo.delay, receivedDelay=50000, deliveryTag=1, consumerTag=amq.ctag-W9bB-UA0fZsHu0WCGvoPDQ, consumerQueue=queue.demo.delay])
```
可以看到，**借助于RabbitMQ延迟插件，延时队列按照消息粒度进行了消费！（此处应该有掌声👏）**



**至此，RabbitMQ 关于普通消息、死信队列和延迟队列相关内容已结束。**

RabbitMQ相关的Demo代码已上传至Github，有需要的话可自行下载查阅。

地址：https://github.com/liangtongdev/demo-springboot-rabbitmq



后续：接下来将介绍利用RabbitMQ实现即时消息的消费，配合Redis实现延时发送与取消。