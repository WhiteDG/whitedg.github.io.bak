---
title: SpringCloud 笔记-熔断器 Hystrix
description: 
categories:
 - SpringCloud
tags: Hystrix
---

在微服务架构中，一般每个服务都会做集群部署，一是可以做负载均衡，二是需要保证服务的高可用。但是由于网络或服务器或服务自身等原因，都有可能导致某一个服务出现问题，如果某个服务出现响应缓慢的情况，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet 容器的线程资源会被消耗完毕，导致服务瘫痪。由于服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

<!-- more -->

## 熔断器

熔断器就是为了解决上述的问题出现的，其原理就是实现快速失败，如果在一段时间内检测到频繁的出现类似的错误，那么后面的调用将会直接返回失败，而不是继续阻塞线程，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。

## Hystrix

SpringCloud 中整合了 Hystrix 组件，实现了熔断器模式。Hystrix 有以下几个特性：
- 断路器机制
    当请求后端服务失败数量超过一定比例(默认50%), 断路器会切换到开路状态(Open). 这时所有请求会直接失败而不会发送到后端服务。断路器保持在开路状态一段时间后(默认5秒), 自动切换到半开路状态(HALF-OPEN). 这时会判断下一次请求的返回情况，如果请求成功，断路器切回闭路状态(CLOSED)，否则重新切换到开路状态(OPEN)。一旦后端服务不可用，断路器会直接切断请求链，避免发送大量无效请求影响系统吞吐量，并且断路器有自我检测并恢复的能力。
- Fallback
    Fallback 相当于是降级操作。我们可以实现一个 fallback 方法，当服务调用失败的时候，不是返回失败结果，而是返回 fallback 方法的执行结果，一般在查询操作中，可以返回默认值或者缓存数据。


## 示例

### Feign + Hystrix
修改服务消费者项目

1. 开启熔断器(修改配置文件)
    ```
    feign.hystrix.enabled=true
    ```

2. 编写熔断回调类

    ```java
    @Component
    public class TestRemoteHystrix implements TestRemote {
        @Override
        public String test(String name) {
            return name + " access Hystrix!";
        }
    }
    ```

 3. 添加熔断回调声明 fallback

    ```java
    @FeignClient(name = "spring-cloud-producer", fallback = TestRemoteHystrix.class)
    public interface TestRemote {
        @RequestMapping(value = "/test")
        String test(@RequestParam(value = "name") String name);
    }
    ```

4. 测试
    1. 分别启动 注册中心、服务提供者、服务消费者
    ![hystrix](/assets/post_imgs/hystrix.jpg)
    2. 访问 http://127.0.0.1:9001/test?name=White 
    ![9001_success](/assets/post_imgs/9001_success.jpg)
    3. 停掉服务提供者，再次访问
    ![9001_hystrix](/assets/post_imgs/9001_hystrix.jpg)

        如上图所示，说明熔断器已经成功运行。

### Ribbon + Hystrix

1. 引入熔断器
    ```
    implementation 'org.springframework.cloud:spring-cloud-starter-hystrix:1.4.6.RELEASE'
    ```

2. 启用熔断器，添加 @EnableHystrix 注解
    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableHystrix
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

3. 添加熔断回调 @HystrixCommand，fallbackMethod 表示熔断回调方法
    ```java
    @RestController
    public class RibbonController {

        @Autowired
        RestTemplate restTemplate;

        @RequestMapping("ribbon")
        @HystrixCommand(fallbackMethod = "testError")
        public String ribbon(@RequestParam String name) {
            return restTemplate.getForObject("http://spring-cloud-producer/test?name=" + name, String.class);
        }

        public String testError(String name) {
            return name + " access Hystrix!";
        }
    }
    ```

4. 与 Feign + Hystrix 一样的测试步骤。