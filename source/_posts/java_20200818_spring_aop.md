---
layout:     post
title:      Spring AOP 简单实践
date:       2020-08-18 19:54:13
author:     liangtong
catalog: true
categories: Java
tags: note

---

#### 

本文介绍下Spring AOP的简单实例，功能很简单，给类或者方法添加日志功能。



#### 准备

使用`IDEA`新建`Maven`项目，添加SpringBoot相关配置和依赖（或直接新建SpringBoot项目）。

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.16.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>org.example</groupId>
    <artifactId>aspect</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```



注解`LtLog`

```Java
package com.lt.annotation;
import java.lang.annotation.*;

@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.METHOD, ElementType.TYPE})
public @interface LtLog {
    //是否开启日志
    boolean enableLog() default true;
}
```

通知 `Advice`

```Java
package com.lt.annotation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class LtLogService {
    public void printLog(String message){
        log.info(message);
    }
}
```

切面 `Aspect`

```Java
package com.lt.annotation;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LTLogAspect {
    @Autowired
    private LtLogService ltLogService;

    /**
     * 拦截类上的 LtLog 注解
     */
    @Around(value = "@within(ltLog)")
    public Object cutClazz(ProceedingJoinPoint joinPoint, LtLog ltLog) throws Throwable {
        return logging(joinPoint, ltLog);
    }
    
    /**
     * 拦截方法上的 LtLog 注解
     */
    @Around(value = "@annotation(ltLog)")
    public Object cutMethod(ProceedingJoinPoint joinPoint, LtLog ltLog) throws Throwable {
        return logging(joinPoint, ltLog);
    }
    
    /**
     * log
     */
    private Object logging(ProceedingJoinPoint joinPoint, LtLog ltLog) throws Throwable {

        try{

            if (ltLog.enableLog()){
                ltLogService.printLog("参数： " + joinPoint.getArgs());
                ltLogService.printLog("开始执行： " + joinPoint.getSignature().toShortString());
            }

            Object result = joinPoint.proceed(joinPoint.getArgs());

            if (ltLog.enableLog()){
                ltLogService.printLog("结束执行： " + joinPoint.getSignature().toShortString());
            }
            return result;
        }catch (Throwable throwable){
            if (ltLog.enableLog()){
                ltLogService.printLog("异常： " + joinPoint.getSignature().toShortString() + throwable.getMessage());
            }
            throw throwable;
        }
    }
}
```



#### 测试

SpringBoot测试启动类

```Java
package com.lt;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
@SpringBootApplication
public class TestAspectDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestAspectDemoApplication.class, args);
    }
}
```

注解测试类 ： 类上添加注解

```Java
package com.lt;
import com.lt.annotation.LtLog;
import org.springframework.stereotype.Component;

@LtLog
@Component
public class TestClassLog {

    public void aspectOnClass(){
        System.out.println("类 注解");
    }
}
```

注解测试类： 方法上添加注解

```Java
package com.lt;
import com.lt.annotation.LtLog;
import org.springframework.stereotype.Component;

@Component
public class TestMethodLog {
    @LtLog
    public void aspectOnMethodOn(){
        System.out.println("方法 注解(开启)");
    }
  
    @LtLog(enableLog = false)
    public void aspectOnMethodOff(){
        System.out.println("方法 注解（未开启）");
    }
}
```



测试及运行结果

```Java
package com.lt;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@SpringBootTest
@RunWith(SpringRunner.class)
public class TestAspectLtLog {
    @Autowired
    private TestClassLog testClassLog;
    @Autowired
    private TestMethodLog testMethodLog;

    @Test
    public void test_classLog(){
        testClassLog.aspectOnClass();
    }

    @Test
    public void test_methodLog(){
        testMethodLog.aspectOnMethodOn();
        testMethodLog.aspectOnMethodOff();
    }
}
```

运行结果如下：

```bash
# test_classLog 日志
2020-08-18 19:54:38.016  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 参数： [Ljava.lang.Object;@1af05b03
2020-08-18 19:54:38.016  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 开始执行： TestClassLog.aspectOnClass()
类 注解
2020-08-18 19:54:38.021  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 结束执行： TestClassLog.aspectOnClass()

# test_methodLog 日志
2020-08-18 19:54:37.984  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 参数： [Ljava.lang.Object;@c0b41d6
2020-08-18 19:54:37.986  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 开始执行： TestMethodLog.aspectOnMethodOn()
方法 注解(开启)
2020-08-18 19:54:37.993  INFO 8252 --- [           main] com.lt.annotation.LtLogService           : 结束执行： TestMethodLog.aspectOnMethodOn()
方法 注解（未开启）
```












