---
layout:     post
title:      Spring 中 @Component和@Bean的区别
date:       2020-10-17 17:04:13
author:     liangtong
catalog: true
categories: Java
tags: 笔记

---



在Spring中，使用 `@Component` 和 `@Bean` 来创建Bean实例，并让Spring容器管理其生命周期。


`@Component`是对应某一个类，表明该类作为组件类,配合 `@ComponentScan` 使用，然后让 Spring IOC 容器实例化。

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
                message -> {
                    message.getMessageProperties().setDelay(10000);
                    return message;
                }
        );
    }
}
```


**@Component（@Controller、@Service、@Repository）通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中。SpringBoot中，默认扫描全包，如果显式使用 @ComponentScan 的话需要注意**



`@Bean` 对应一个返回对象的方法，表明该对象要注册为Spring上下文中的bean， 配合 `@Confiuration`使用，在配置文件中，返回的是某个对象的实例。


```Java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(5000);//ms
        factory.setConnectTimeout(15000);//ms
        return factory;
    }

}
```



使用`@Component` 和 `@Bean` 配置的类，都可以通过@Autowired装配

```Java


    @Autowired
    private MsgProducer sender;

    @Autowired
    private RestTemplate restTemplate;

```



`@Bean` 比 `@Component` 的自定义性更强，可以实现一些 `@Component` 实现不了的自定义加载类（例如想让第三方类变成组件，没有源码就不能用@Component的形式。这时候使用@Bean就比较合适）。


