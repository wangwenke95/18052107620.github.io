---
title: Mybatis实现Redis二级缓存
date: 2023-03-28 21:15:06
tags: Mybatis实现Redis二级缓存
categories: MyBatis
description: 对mybatis/mybatis-plus整合spring-boot，实现二级缓存机制,减少对数据的重复查询
keywords: MyBatis,二级缓存，Spring Cache
cover: https://pic4.zhimg.com/v2-7fc793b33a33cef471423a89bf94e0b3_1440w.jpg?source=172ae18b
---

# 简述

## 缓存更新策略

- 利用Redis的缓存淘汰策略被动更新 LRU 、LFU
- 利用TTL被动更新
- 在更新数据库时主动更新 （先更数据库再删缓存----延时双删）
- 异步更新 定时任务 数据不保证时时一致 不穿DB

## 不同策略之间的优缺点

| 策略                            | 一致性 | 维护成本 |
| ------------------------------- | ------ | -------- |
| 利用Redis的缓存淘汰策略被动更新 | 最差   | 最低     |
| 利用TTL被动更新                 | 较差   | 较低     |
| 在更新数据库时主动更新          | 较强   | 最高     |

## Redis与Mybatis整合

- 可以使用Redis做Mybatis的二级缓存，在分布式环境下可以使用。
- 框架采用springboot+Mybatis+Redis。框架的搭建就不赘述了。

## 常用注解
  - @Cacheable ： 触发将数据保存到缓存的操作；
  - @CacheEvict : 触发将数据从缓存删除的操作；
  - @CachePut ： 不影响方法执行更新缓存；
  - @Cacheing： 组合以上多个操作；
  - @CacheConfig： 在类级别共享缓存的相同配置；

## 缓存Key指定
| 属性名称    | 描述                        | 示例                 |
| ----------- | --------------------------- | -------------------- |
| methodName  | 当前方法名                  | #root.methodName     |
| method      | 当前方法                    | #root.method.name    |
| target      | 当前被调用的对象            | #root.target         |
| targetClass | 当前被调用的对象的class     | #root.targetClass    |
| args        | 当前方法参数组成的数组      | #root.args[0]        |
| caches      | 当前被调用的方法使用的Cache | #root.caches[0].name |


# 操作

## 在pom.xml中添加对应的依赖

```xml
    <!--缓存-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <!--redis-->
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```

## 在application.yml中添加配置

```yaml
server:
  port: 8080

mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    cache-enabled: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl


spring:
  redis:
    port: 6379
    host: 192.168.0.124
    database: 0
  datasource:
    url: jdbc:mysql://localhost:3306/book?useSSl=true&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
  cache:
    redis:
      # 缓存过期时间(1800秒)
      time-to-live: 1800000
logging:
  level:
    mapper: debug

```

## 启动类加入缓存注解

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.BeansException;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

import javax.sql.DataSource;

@SpringBootApplication
@MapperScan("com.mapper")
//缓存注解
@EnableCaching
public class SsmApplication implements ApplicationContextAware {
    public static void main(String[] args) {
        SpringApplication.run(SsmApplication.class, args);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println(applicationContext.getBean(DataSource.class).getClass().getSimpleName());
    }
}
```

## 在对应操作数据库的方法上实现二级缓存

```java
import com.pojo.Bookweb;
import com.service.BookwebService;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.List;

@RestController
public class BookWebController {
    @Resource
    private BookwebService bookwebService;

    @RequestMapping("/list")
    //把查询的所有数据都加入到redis的二级缓存
    @Cacheable(value = "BookCache")
    public List<Bookweb> list() {
        return bookwebService.list();
    }

    @RequestMapping("/add")
    //当增加一条数据,就清楚redis里的所有BookCache二级缓存数据
    @CacheEvict(value = "BookCache", allEntries = true)
    public void add(@RequestBody Bookweb bookweb) {
        bookwebService.save(bookweb);

    }
}




```
