---
title: SpringCloud 笔记-注册中心 Eureka
description: 
categories:
 - SpringCloud
tags: Eureka
---

Eureka 是 Netflix 开发的服务发现框架，本身是一个基于 REST 的服务，主要用于定位运行在 AWS 域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。SpringCloud 将它集成在其子项目 spring-cloud-netflix 中，以实现 SpringCloud 的服务发现功能。    
Eureka包含两个组件：Eureka Server 和 Eureka Client。
<!-- more -->

## 注册中心（服务中心）

注册中心是用来统一管理各个微服务功能的，提供了包括服务的注册、发现、熔断、负载、降级等功能。所有服务之间的相互调用都要通过注册中心来调用。

![服务调用](/assets/post_imgs/service_invoke.jpg)

上图正常的业务流程是需要 服务 A 调用服务 B，服务 B 调用服务 C，有了注册中心之后，调用的步骤就会为变成：服务 A 首先从注册中心请求服务 B 服务器，然后服务 B 再从注册中心请求 服务 C。

注册中心将所有的服务都被统一管理起来的，各个服务之间不需要再关注相互之间的调用地址，只需要从注册中心获取可用的服务调用即可。

## Eureka 的组成
1. Eureka Server: 就是上文说明的注册中心，用来提供服务注册于发现
2. Service Provider: 服务提供方，需要将自己注册到 Eureka Server 供 Service Consumer 发现
3. Service Consumer: 服务消费者，从 Eureka Server 获取服务进行消费

## SpringBoot 集成 Eureka 

> 创建一个 SpringBoot 项目并且引入 SpringCloud 依赖

### Eureka Server
1. 引入 Eureka Server 依赖

```gradle
...
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
}
...
```

2. 添加 @EnableEurekaServer 注解，表示这是一个注册中心    

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

3. 修改注册中心相关配置

```json
spring.application.name=spring-cloud-eureka
server.port=8000
## 表示是否将自己注册到Eureka Server，默认为true。
eureka.client.register-with-eureka=false
## 表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.fetch-registry=false
## 设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka，多个地址可使用 , 分隔。
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```

4. 启动项目，访问 http://localhost:8000/

![注册中心后台](/assets/post_imgs/eureka_server.jpg)

### 注册中心集群
注册中心在微服务中扮演的角色十分重要，如果只有一个注册中心，要是注册中心挂了那么就相当于切断了所有服务之间的联系，几乎就是相当于整个系统挂了，所以保证注册中心的高可用是非常重要的。集群便是保证其高可用的解决方案，Eureka 可以通过互相注册的方式来实现高可用的部署，所以只需要将 Eureke Server 配置其他可用的 serviceUrl.defaultZone 就能实现高可用部署。

1. 新建两个配置文件 application-peer1.properties、application-peer2.properties，配置内容如下：

```json
spring.application.name=spring-cloud-eureka
server.port=8000
eureka.instance.hostname=peer1
# 将 serviceUrl.defaultZone 指向peer2
eureka.client.serviceUrl.defaultZone=http://peer2:8001/eureka/
```

```json
spring.application.name=spring-cloud-eureka
server.port=8001
eureka.instance.hostname=peer2
# 将 serviceUrl.defaultZone 指向peer1
eureka.client.serviceUrl.defaultZone=http://peer1:8000/eureka/
```

2. 上一步用到了 peer1 和 peer2 两个域名，我们本地测试需要修改 host 文件，在 host 文件中增加以下内容：

```
127.0.0.1 peer1  
127.0.0.1 peer2
```

3. 分别使用两个配置文件启动程序，一次执行以下命令：

```
1. 打包
./gradlew clean bootJar
2. 进入打包文件目录
cd build/libs 
3. 分别启动使用 peer1、peer2 配置文件启动程序
java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer1
java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2
```

分别访问  http://pee1:8000/  http://pee2:8001/   

``` http://pee1:8000/ ```
![peer1_8000](/assets/post_imgs/peer1_8000.jpg)

``` http://pee2:8001/ ```
![peer2_8001](/assets/post_imgs/peer2_8001.jpg)

如上图所示，peer 1的 DS Replicas 已经有了 peer2 的相关配置信息，并且出现在 available-replicas 中。如果手动停止 peer2，那么 peer2 就会移动到 unavailable-replicas 一栏中，表示 peer2 不可用。

![peer2_unavailable](/assets/post_imgs/peer2_unavailable.jpg)

