# Spring boot 学习笔记

## 数据库配置

```properties
spring.datasource.url=jdbc:mysql:///training_system?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&userSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# mybatis
mybatis.mapper-locations=classpath:/mapper/*Mapper.xml
mybatis.type-aliases-package=com.dgut.liukc.trainingsystem.dao
```

## 缓存数据库 Redis 配置及使用

### 引入依赖包

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 相关配置

```properties

```

