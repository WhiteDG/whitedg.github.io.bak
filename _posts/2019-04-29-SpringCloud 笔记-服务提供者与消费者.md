---
title: SpringCloud 笔记-服务提供者与消费者
description: 
categories:
 - SpringCloud
tags: 
---

服务提供者和消费者是微服务架构中两个重要的角色，服务提供者生产服务并注册到服务中心中，消费者从服务中心中获取服务并执行。
<!-- more -->

## 服务提供者 producers

1. 新建一个 SpringBoot 项目并且引入 SpringCloud 依赖
2. 引入 netflix-eureka-client 依赖

    ```
    ...
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    ...
    ```
3. 配置文件

    ```json
    spring.application.name=spring-cloud-producer
    server.port=9000
    # 注册中心地址
    eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
    ```
4. 添加 @EnableDiscoveryClient 注解开启服务发现

    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class EurekaProviderApplication {

        public static void main(String[] args) {
            SpringApplication.run(EurekaProviderApplication.class, args);
        }

    }
    ```
5. 提供一个接口服务

    ```java
    @RestController
    public class TestController {

        @RequestMapping("test")
        public String test(@RequestParam String name) {
            return name + " access test service !";
        }
    }
    ```

## 服务消费者 consumers

### Feign
在 SpringCloud 架构内，服务之间的调用时通过 HTTP 通信完成的，这就意味着在调用消费者在调用提供者的服务时，需要有一个 Web 服务客户端，使用 Feign 能让编写 Web 客户端更加简单，而且 Spring Cloud 也对 Feign进行了封装。

Feign 是一个声明式 Web Service 客户端。它的使用方法是定义一个接口，然后在上面添加注解，同时也支持 JAX-RS 标准的注解。Feign 也支持可拔插式的编码器和解码器。Spring Cloud 对 Feign进行了封装，使其支持了 Spring MVC 标准注解和 HttpMessageConverters。Feign可以与 Eureka 和 Ribbon 组合使用以支持负载均衡。

1. 新建一个 SpringBoot 项目并且引入 SpringCloud 依赖
2. 引入 netflix-eureka-client、openfeign 依赖

    ```
    ...
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
    ...
    ```

3. 配置文件

    ```json
    spring.application.name=spring-cloud-consumer
    server.port=9001
    eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
    ```
4. 添加 @EnableDiscoveryClient 注解开启服务发现、添加 @EnableFeignClients 开启 Feign 进行远程调用

    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableFeignClients
    public class EurekaConsumerApplication {

        public static void main(String[] args) {
            SpringApplication.run(EurekaConsumerApplication.class, args);
        }

    }
    ```
5. Feign 声明远程调用

    @FeignClient 中的参数 name 表示远程服务名，这里就是服务提供者的 spring.application.name

    声明远程调用的接口中的方法和远程服务中 contoller 中的方法名和参数需保持一致。

    ```java
    @FeignClient(name = "spring-cloud-producer")
    public interface TestRemote {
        @RequestMapping(value = "/test")
        String test(@RequestParam(value = "name") String name);
    }
    ```
6. 调用服务提供者提供的远程服务

    ```java
    @RestController
    public class ConsumerController {

        @Autowired
        TestRemote testRemote;

        @RequestMapping("/test")
        public String test(@RequestParam("name") String name) {
            return testRemote.test(name);
        }
    }
    ```

7. 测试

    - 依次启动 [注册中心](https://whitedg.github.io/springcloud/2019/04/28/SpringCloud-%E7%AC%94%E8%AE%B0-%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83Eureka/#){:target="_blank"}。、服务提供者、服务消费者
    - 访问 http://127.0.0.1:9000/test?name=White 测试提供者的服务是否可用
    ![produces_enable](/assets/post_imgs/produces_enable.jpg)
    - 访问 http://127.0.0.1:9001/test?name=White 测试消费者远程调用是否成功
    ![fegin_test](/assets/post_imgs/fegin_test.jpg)