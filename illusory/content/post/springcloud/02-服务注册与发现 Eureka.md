---
title: "SpringCloud入门系列(二)---服务注册与发现"
description: "服务注册与发现中心Eureka"
date: 2019-03-16 22:00:00
draft: false
tags: ["SpringCloud"]
categories: ["SpringCloud"]
---

本文主要介绍了`SpringCloud-Netflix`系列微服务解决方案之服务注册与发现中心`Eureka`。

<!--more-->

> **[SpringCloud入门系列文章目录](https://www.lixueduan.com/categories/SpringCloud/)**
>
> [SpringCloud入门系列(一)---统一依赖管理 Dependencies](https://www.lixueduan.com/posts/c41e9ac1.html)
>
> [SpringCloud入门系列(二)---服务注册与发现](https://www.lixueduan.com/posts/8735f06d.html)
>
> [SpringCloud入门系列(三)---服务提供者 Provider](https://www.lixueduan.com/posts/cc38b97d.html)
>
> [SpringCloud入门系列(四)---服务消费者 Consumer](https://www.lixueduan.com/posts/bf9c3958.html)
>
> [SpringCloud入门系列(五)---服务熔断 Hystrix](https://www.lixueduan.com/posts/3cc75bae.html)
>
> [SpringCloud入门系列(六)---路由网关 Zuul](https://www.lixueduan.com/posts/abd301c1.html)
>
> [SpringCloud入门系列(七)---分布式配置中心 Spring Cloud Config](https://www.lixueduan.com/posts/68aa18ee.html)
>
> [SpringCloud入门系列(八)---服务链路追踪 ZipKin](https://www.lixueduan.com/posts/c9420366.html)
>
> [SpringCloud入门系列(九)---服务监控 Spring Boot Admin](https://www.lixueduan.com/posts/14927f8f.html)
>
> 
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)
>
> Demo下载：[GItHub](https://github.com/illusorycloud/springboot-learning)

## 1. 概述

在这里，我们需要用的组件是 `Spring Cloud Netflix` 的 `Eureka`，`Eureka` 是一个服务注册和发现模块。

## 2. 创建服务注册于发现项目

创建一个项目`hello-spring-cloud-eureka`作为注册中心。

### 2.1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.lixueduan</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-eureka</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-eureka</name>
    <url>http://www.lixueduan.com</url>
    <inceptionYear>2019-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                   <!--指向启动类 用于jar包方式运行--> <mainClass>com.illusory.hello.spring.cloud.eureka.EurekaApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.2 Application

启动一个服务注册中心，只需要一个注解 `@EnableEurekaServer`

```Java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

### 2.3 application.yml

Eureka 是一个高可用的组件，它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下 Erureka Server 也是一个 Eureka Client ,必须要指定一个 Server。

```yaml
spring:
  application:
    name: hello-spring-cloud-eureka
  boot:
    admin:
      client:
        url: http://localhost:8084
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

通过 `eureka.client.registerWithEureka:false` 和 `fetchRegistry:false` 来表明自己是一个 Eureka Server.

## 3. 测试

Eureka Server 是有界面的，启动工程，打开浏览器访问：

`http://localhost:8761`

## 4. 小结

本章我们主要使用`Eureka`组件建立了我们的注册中心。