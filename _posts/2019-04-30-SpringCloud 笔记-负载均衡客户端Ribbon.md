---
title: SpringCloud 笔记-负载均衡客户端Ribbon
description: 
categories:
 - SpringCloud
tags: Ribbon
---

Ribbon 是一个基于 HTTP 和 TCP 的负载均衡客户端，Ribbon 默认提供了很多负载均衡算法，例如轮询、随机等。当然，开发者也可为 Ribbon 实现自定义的负载均衡算法。上一篇笔记中提到的 Feign 已经默认使用了 Ribbon。SpringCloud 远程调用还有另一种方式——RestTemplate，这次使用 RestTemplate 作为 Web 客户端。

<!-- more -->

## 服务提供者支持负载均衡

1. spring-cloud-producer 中增加配置文件 application-9000.properties、application-9003.properties，使用不同端口号
    ```
    spring.application.name=spring-cloud-producer
    server.port=9000
    eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
    ```

    ```
    spring.application.name=spring-cloud-producer
    server.port=9003
    eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
    ```

2. 修改 spring-cloud-producer 中的 test 服务的响应内容，增加端口号，便于后面测试

    ```java
    @RestController
    public class HelloController {

        @Value("${spring.profiles.active}")
        String active;

        @RequestMapping("test")
        public String test(@RequestParam String name) {
            return name + " access test service ! port: " + active;
        }
    }
    ```

3. 后面分别使用两个配置文件启动 spring-cloud-producer

## Ribbon + RestTemplate

1. 新建一个 SpringBoot 项目并且引入 SpringCloud 依赖
2. 配置文件
    ```
    spring.application.name=spring-cloud-ribbon
    server.port=10000
    ##设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。
    eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
    ```
3. 在项目中注入一个 RestTemplate 并配置开启负载均衡
    ```java 
    @SpringBootApplication
    @EnableDiscoveryClient
    public class RibbonApplication {

        public static void main(String[] args) {
            SpringApplication.run(RibbonApplication.class, args);
        }

        @Bean
        @LoadBalanced
        RestTemplate restTemplate() {
            return new RestTemplate();
        }

    }
    ```

4. 编写 RibbonController 测试 RestTemplate 的远程调用和 Ribbon 的负载均衡

    ```java 
    @RestController
    public class RibbonController {

        @Autowired
        RestTemplate restTemplate;

        @RequestMapping("ribbon")
        public String ribbon(@RequestParam String name) {
            return restTemplate.getForObject("http://spring-cloud-producer/test?name=" + name, String.class);
        }
    }

    ```
在这里直接用的服务名替代了具体的 url 地址，在 Ribbon 中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的 url 替换掉服务名。

## 测试

1. 分别启动 注册中心、服务提供者(9000/9003)、服务消费者(spring-cloud-ribbon)
![ribbon_produces](/assets/post_imgs/ribbon_produces.jpg)
2. 多次访问 http://127.0.0.1:10000/ribbon?name=White 测试负载均衡是否生效
![9000_resp](/assets/post_imgs/9000_resp.jpg)
![9003_resp](/assets/post_imgs/9003_resp.jpg)

    多次访问分别调用了不同的服务实例，说明负载均衡生效了。