---
title: SpringBoot 整合Druid
date: 2023-03-28 22:17:57
tags: Spring Boot整合Druid
categories: Spring-Boot
description: 结合SpringBoot使用Druid数据库连接池
keywords: Druid,SpringBoot整合Druid，数据库连接池
cover: https://javakk.com/wp-content/uploads/2021/05/druid_8-750x500.png?imageView2/1/w/375/h/250/q/100
---

## 配置pom.xml

```xml
        <!-- 这里mysql配置5版本就可以 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>druid</artifactId>
<version>1.2.14</version>
</dependency>
```

## 配置application.yml

```yaml
  datasource:
    url: jdbc:mysql://localhost:3306/xxx?useSSl=true&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
```

## 测试数据数据库连接池是否生效

```java
package com;

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

{% note blue 'fas fa-bullhorn' simple %}
结果: DruidDataSource
{% endnote %}

## 配置额外参数(可选)

```yaml
  datasource:
    url: jdbc:mysql://localhost:3306/book?useSSl=true&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置

    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    filters: wall,stat,log4j
    #2.连接池配置
    #初始化连接池的连接数量 大小，最小，最大
    initial-size: 5
    min-idle: 5
    max-active: 20
    #配置获取连接等待超时的时间
    max-wait: 60000
    #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    time-between-eviction-runs-millis: 60000
    # 配置一个连接在池中最小生存的时间，单位是毫秒
    min-evictable-idle-time-millis: 30000
    validation-query: SELECT 1 FROM DUAL
    test-while-idle: true
    test-on-borrow: true
    test-on-return: false
    # 是否缓存preparedStatement，也就是PSCache  官方建议MySQL下建议关闭   个人建议如果想用SQL防火墙 建议打开
    pool-prepared-statements: true
    max-pool-prepared-statement-per-connection-size: 20
```

## Druid参数说明

<table style="width:650px;" cellspacing="1" cellpadding="1" border="1">
    <caption>
        DruidDataSource 参数
    </caption>
    <thead>
    <tr>
        <td>配置</td>
        <th>缺省值</th>
        <th>说明</th>
    </tr>
    </thead>
    <tr>
        <td>name</td>
        <td></td>
        <td>配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。&nbsp;<br>
            如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this)
        </td>
    </tr>
    <tbody>
    <tr>
        <td>jdbcUrl</td>
        <td></td>
        <td>连接数据库的url，不同数据库不一样。例如：&nbsp;<br> mysql : jdbc:mysql://10.20.153.104:3306/druid2&nbsp;<br>
            oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto
        </td>
    </tr>
    <tr>
        <td>username</td>
        <td></td>
        <td>连接数据库的用户名</td>
    </tr>
    <tr>
        <td>password</td>
        <td></td>
        <td>连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter</td>
    </tr>
    <tr>
        <td>driverClassName</td>
        <td>根据url自动识别</td>
        <td>这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下)</td>
    </tr>
    <tr>
        <td>initialSize</td>
        <td>0</td>
        <td>初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时</td>
    </tr>
    <tr>
        <td>maxActive</td>
        <td>8</td>
        <td>最大连接池数量</td>
    </tr>
    <tr>
        <td>maxIdle</td>
        <td>8</td>
        <td>已经不再使用，配置了也没效果</td>
    </tr>
    <tr>
        <td>minIdle</td>
        <td></td>
        <td>最小连接池数量</td>
    </tr>
    <tr>
        <td>maxWait</td>
        <td></td>
        <td>
            获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
        </td>
    </tr>
    <tr>
        <td>poolPreparedStatements</td>
        <td>false</td>
        <td>
            是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。
        </td>
    </tr>
    <tr>
        <td>maxOpenPreparedStatements</td>
        <td>-1</td>
        <td>
            要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
        </td>
    </tr>
    <tr>
        <td>validationQuery</td>
        <td></td>
        <td>
            用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。
        </td>
    </tr>
    <tr>
        <td>testOnBorrow</td>
        <td>true</td>
        <td>申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。</td>
    </tr>
    <tr>
        <td>testOnReturn</td>
        <td>false</td>
        <td>归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能</td>
    </tr>
    <tr>
        <td>testWhileIdle</td>
        <td>false</td>
        <td>
            建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
        </td>
    </tr>
    <tr>
        <td>timeBetweenEvictionRunsMillis</td>
        <td></td>
        <td><p>有两个含义：&nbsp;<br> 1) Destroy线程会检测连接的间隔时间</p>
            <p>2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明</p></td>
    </tr>
    <tr>
        <td>numTestsPerEvictionRun</td>
        <td></td>
        <td>不再使用，一个DruidDataSource只支持一个EvictionRun</td>
    </tr>
    <tr>
        <td>minEvictableIdleTimeMillis</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>connectionInitSqls</td>
        <td></td>
        <td>物理连接初始化的时候执行的sql</td>
    </tr>
    <tr>
        <td>exceptionSorter</td>
        <td>根据dbType自动识别</td>
        <td>当数据库抛出一些不可恢复的异常时，抛弃连接</td>
    </tr>
    <tr>
        <td>filters</td>
        <td></td>
        <td>属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：&nbsp;<br>
            监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall
        </td>
    </tr>
    <tr>
        <td>proxyFilters</td>
        <td></td>
        <td>
            类型是List&lt;com.alibaba.druid.filter.Filter&gt;，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系
        </td>
    </tr>
    </tbody>
</table>
