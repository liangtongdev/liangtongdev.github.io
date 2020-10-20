---
layout:     post
title:      RabbitMQ 交换器类型
date:       2020-10-20 11:54:13
author:     liangtong
catalog: true
categories: Java
tags: mq

---

​	在`RabbitMQ`中，生产者的消息都是经过 `Exchange` 来接收，然后再转发到不同的 `Queue` 中。 `RabbitMQ`的交换器类型有 `fanout` 、 `direct` 、 `topic` 、 `header` 四种。
![](/post/java/20201020/hello-world-example-routing.png)


### fanout

一般情况下，交换机分发消息时会先找到绑定的队列，然后判断路由键 `RouteKey` 后决定是否将消息转发到某一个队列中，但如果交换器类型是 `fanout`， 那么交换器就不再判断路由键，直接将消息转发到绑定的队列中。

![fanout](/post/java/20201020/exchange-fanout.png)

```Java
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
        // 注意：此处绑定关系时，不需要指定绑定键
        return BindingBuilder.bind(queue)
                .to(exchange);
    }
}
```

**使用  `fanout` 类型的Exchange，在绑定队列与交换器关系时，不需要指定路由键**
当多个队列与 `fanout` 类型的Exchange绑定时，交换器会把消息分发到所有的队列中。


### direct

在类型是 `direct` 的情况下，交换器在分发消息的时候会先获取绑定的队列，然后再判断消息路由键 `RouteKey` ；当 `RouteKey` 与绑定键 `BindingKey` 完全匹配时，将消息分发到对应的队列中

![fanout](/post/java/20201020/exchange-direct.png)

```Java
@Configuration
public class RabbitMQConfig {
    // 声明交换器
    @Bean("fixDelayExchange")
    public DirectExchange fixDelayExchange() {
        return new DirectExchange(EXCHANGE_DELAY_FIX_NAME);
    }
    // 声明队列
    @Bean("fixDelayQueue1")
    public Queue fixDelayQueue1() {
        Map<String, Object> args = new HashMap<>(1);
        // x-message-ttl  声明队列的TTL
        args.put("x-message-ttl", 60000);
        return QueueBuilder.durable(QUEUE_DELAY_FIX_1_NAME).withArguments(args).build();
    }
    // 绑定关系
    @Bean
    public Binding fixDelay1Binding(@Qualifier("fixDelayQueue1") Queue queue,
                                   @Qualifier("fixDelayExchange") DirectExchange exchange) {
        // 注意： 此处绑定关系时，需要绑定键 BindingKey
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(ROUTING_KEY_DELAY_FIX_1_NAME);
    }
}
```

一个队列可以指定多个路由键，当生产者发送一条消息到交换器后，交换器会根据路由键将消息分发到对应绑定到队列中。
**如果与 `direct` 类型交换器绑定的消息队列中，多个队列绑定了相同的路由键 BindingKey，当发送路由键 RouteKey 类型消息到交换器后，交换器会匹配消息的RouteKey 和 队列的BindingKey ，并把分发到所有绑定该路由键的队列中，即多个队列会收到消息**


### topic

在类型是 `topic` 的情况下，交换器在分发时也需要匹配路由键，与 `direct` 类型的区别是 `topic` 类型时，判断路由键的规则是模糊匹配。

+ 路由键以 `.` 为分隔符，每个分隔符的代表是一个单词。

+ `*` 作为通配符，每个通配符 `*` 可以匹配一个单词，例如 `*.*.com` 。

+ `#` 作为通配符，每个 `#` 可以匹配多个单词。



#### header

类型为 `header` 的交换器与前边的几种完全不一样。不依赖绑定Key和路由Key，而是在绑定队列和交换器的时候指定健值对，当交换器在分发消息时会匹配消息体的header数据，如果匹配成功则将消息转发到对应的队列中。


**从性能上比较 ： fanout > direct > topic > headers**

参考资料：https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges
