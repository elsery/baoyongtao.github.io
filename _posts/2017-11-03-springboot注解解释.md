---
layout: post
title: "springboot注解解释"
date: 2017-11-03 
description: "SpringBoot,注解"
tag: java,SpringBoot
--- 

  

### @EnableAutoConfiguration
> 开启自动配置。
 作用：SpringBoot会自动根据你jar包的依赖来自动配置项目。例如当你项目下面有HSQLDB的依赖时，Spring
 Boot会创建默认的内存数据库的数据源DataSource，如果你自己创建了DataSource，Spring
 Boot就不会创建默认的DataSource。
@EnableAutoConfiguration 注解。我们建议你将它添加到主 @Configuration 类上。

### @ComponentScan
>ComponentScan 注解而不需要添加任何参数，Spring Boot会在根包下面搜索注有@Component, @Service,@Repository, @Controller

### @SpringBootApplication
>由于大量项目都会在主要的配置类上添加@Configuration,@EnableAutoConfiguration,@ComponentScan三个注解。因此Spring
Boot提供了@SpringBootApplication注解，该注解可以替代上面三个注解

>###### @RestController 和 @RequestMapping 注解是Spring MVC注解

在启动类我们这样写

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

也可以这样写

```java
@EnableAutoConfiguration 
@ComponentScan 
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
>@Configuration
等价 与XML中配置beans；用@Bean标注方法等价于XML中配置bean

>@Value
   注入Spring boot application.properties配置的属性的值。

>@ImportResource
加载指定的配置文件 如 @ImportResource(locations={"classpath:application-bean.xml"})









