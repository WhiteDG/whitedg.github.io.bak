---

title: Spring Boot Actuator
description: 
categories:
 - SpringBoot
tags: Actuator
---

## 简介

`spring-boot-starter-actuator` 是一个用于监控和管理 Spring Boot 应用的组件。使用 `spring-boot-starter-actuator` 之后，我们可以非常方便的通过 HTTP 接口或者 JMX 获取到服务的运行状态，依赖的数据库状态，JVM 的运行状态，搜集指标等等，从而实现对服务的监控和管理。配合 prometheus 和 grafana 可以快速的实现监控数据可视化。

<!-- more -->

## 使用方式

### 添加依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Actuator 的默认路径是 `/actuator` 也就是说，引入`spring-boot-starter-actuator` 之后，访问 `/actuator` 接口即可判断是否引入成功，例如：

访问`http://localhost:9002/actuator/`

``` json
{
  "_links": {
    "self": {
      "href": "http://localhost:9002/actuator",
      "templated": false
    },
    "health": {
      "href": "http://localhost:9002/actuator/health",
      "templated": false
    },
    "health-component": {
      "href": "http://localhost:9002/actuator/health/{component}",
      "templated": true
    },
    "health-component-instance": {
      "href": "http://localhost:9002/actuator/health/{component}/{instance}",
      "templated": true
    },
    "info": {
      "href": "http://localhost:9002/actuator/info",
      "templated": false
    }
  }
}
```

Actuator 的默认路径可以通过 `management.endpoints.web.base-path` 配置项修改，例如将路径改为 `/manage`：

```properties
management.endpoints.web.base-path = /manage
```

Actuator 的默认端口和路径是继承自 `server` 配置的，我们也可以设置 `actuator` 独立的端口号，将端口号设置为 -1 表示禁用 HTTP EndPoint

```properties
management.server.port=8081
```



## EndPoint(端点)

Actuator 提供了一些默认的 EndPoints，可以通过配置文件自由选择是否启用，各个 EndPoint 的作用如下表

| ID               | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| auditevents      | 当前应用的审计信息（订单数量）                               |
| beans            | 应用中所有 Spring Bean 的完整列表                            |
| caches           | 当前可用的缓存信息                                           |
| conditions       | 在配置和自动配置类上的条件以及它们匹配或不匹配的原因         |
| configprops      | @ConfigurationProperties 列表                                |
| env              | 当前的运行环境信息                                           |
| flyway           |                                                              |
| health           | 当前应用的健康信息                                           |
| httptrace        | HTTP 链路信息（默认情况下，返回最近 100 个 HTTP 请求-响应信息） |
| info             | 自定义的信息（配置文件中的 info 配置项）                     |
| integrationgraph |                                                              |
| loggers          | 日志配置信息，包括各个类的日志等级                           |
| liquibase        |                                                              |
| metrics          | 应用中的所有指标名                                           |
| mappings         | 所有 @RequestMapping 映射的列表                              |
| scheduledtasks   | 定时任务                                                     |
| sessions         |                                                              |
| shutdown         | 停止应用                                                     |
| threaddump       | 所有线程信息                                                 |

Web 应用特有的 EndPoint

| ID         | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| heapdump   | 返回一个 hprof 堆转储文件                                    |
| jolokia    |                                                              |
| logfile    | 返回日志文件的内容（如果已设置logging.file.name或logging.file.path属性）。支持使用 HTTP Range 标头来检索部分日志文件的内容。 |
| prometheus | 返回 Prometheus 服务器可以抓取的格式指标。需要依赖于micrometer-registry-prometheus。 |

### EndPoint 启用/禁用

默认情况下，Actuator 启用了除 shutdown 之外的所有 EndPoint。我们可以通过 EndPoint 的配置项来控制所有端点是否启用。配置项为 `management.endpoint.<endpoint_id>.enabled`例如：启用 **shutdown** 端点

```properties
management.endpoint.shutdown.enabled = true
```

如果只想手动启用某些 EndPoint，可以将 `management.endpoints.enabled-by-default`设为 false，然后在使用前面的单个 EndPoint 配置项启用指定的 EndPoint，例如：

```properties
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

### 暴露 EndPoint

由于 EndPoint 中可能包含部分敏感信息，所以 Actuator 默认只暴露了 health 和 info 两个 EndPoint。其他 EndPoint 的暴露需要通过以下配置实现：

```properties
# 排除哪些 endpoint
management.endpoints.web.exposure.exclude
# 暴露哪些 endpoint, * 表示全部
management.endpoints.web.exposure.include = "*"
```

上面两个配置项分别表示排除和暴露哪些 EndPoint，exclude 的优先级高于 include，也就是说，一个 EndPoint 同时出现在这两个配置项中时，这个 EndPoint 就是被排除的。例如：暴露所有 EndPoint，排除 env 和 beans

```properties
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

暴露所有 EndPoint 是非常不安全的行为，所以如果你的应该暴露了所有 EndPoint，建议同时使用 spring-security 保证接口的安全性。

### 部分 endpoint 介绍

#### health

health 用于检查正在运行的应用程序的运行状况或状态。health 接口会返回当前应用的状态，包括数据库、Redis、磁盘、ES 等等。

health 接口返回的数据可以通过 `management.endpoint.health.show-details`和`management.endpoint.health.show-components`配置，配置项的取值有以下三个：

- never：不显示详细信息(默认)
- when-authorized：仅授权用户访问时返回详细信息，可以使用`management.endpoint.health.roles`配置授权角色。
- always：对所有用户都展示详细信息

```json
{
  "status": "UP",
  "details": {
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 250685575168,
        "free": 66478456832,
        "threshold": 10485760
      }
    },
    "db": {
      "status": "UP",
      "details": {
        "database": "MySQL",
        "hello": 1
      }
    },
    "redis": {
      "status": "UP",
      "details": {
        "version": "5.0.5"
      }
    }
  }
}
```

我们可以通过配置文件开启或者关闭指定的检查项。例如，关闭 Redis 可用性检查：

```properties
management.health.redis.enabled = false
```

还可以实现自己的自定义运行状况指示器，获取自定义的运行状况数据，并通过 health 接口返回

```java
@Component
public class HealthCheck implements HealthIndicator {
    @Override
    public Health health() {
        // 调用自定义的检查方法
        int errorCode = check();
        if (errorCode != 0) {
            return Health.down()
                    .withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

    private int check() {
        // 自定义的健康检查逻辑
        return 0;
    }
}
```

```json
{
  "status": "DOWN",
  "details": {
    "healthCheck": {
      "status": "DOWN",
      "details": {
        "Error Code": 1
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 250685575168,
        "free": 65371209728,
        "threshold": 10485760
      }
    },
    "db": {
      "status": "UP",
      "details": {
        "database": "MySQL",
        "hello": 1
      }
    },
    "redis": {
      "status": "UP",
      "details": {
        "version": "5.0.5"
      }
    }
  }
}
```

自动注入的健康检查监视器

| Name                                                         | Description                                 |
| :----------------------------------------------------------- | :------------------------------------------ |
| [`CassandraHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) | 检查 Cassandra database 是否可用            |
| [`CouchbaseHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) | 检查 Couchbase 节点是否可以                 |
| [`DiskSpaceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) | Checks for low disk space.                  |
| [`ElasticSearchRestHealthContributorAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticSearchRestHealthContributorAutoConfiguration.java) | Checks that an Elasticsearch cluster is up. |
| [`HazelcastHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/hazelcast/HazelcastHealthIndicator.java) | 检查 Hazelcast 是否可用                     |
| [`InfluxDbHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) | 检查 InfluxDB 是否可用                      |
| [`JmsHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) | 检查 JMS broker 是否可用                    |
| [`LdapHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/ldap/LdapHealthIndicator.java) | 检查 LDAP 服务是否可用                      |
| [`MailHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) | Checks that a mail server is up.            |
| [`MongoHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) | 检查 Mongo 是否可用                         |
| [`Neo4jHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) | 检查 Neo4j 数据库是否可用                   |
| [`PingHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/PingHealthIndicator.java) | 一直返回 `UP`.                              |
| [`RabbitHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) | 检查 Rabbit 服务是否可用                    |
| [`RedisHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) | 检查 Redis 是否可用                         |
| [`SolrHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) | 检查 Solr 是否可用                          |

#### info

info 用于显示自定义的应用信息，默认返回一个空的 json，我们可以在配置文件中添加 info 配置项来设置一些属性，例如：

```yaml
info:
  app:
    name: spring-boot-actuator-app
    version: 1.0
```

访问 info 接口时返回：

```json
{
  "app": {
    "name": "spring-boot-actuator-app",
    "version": 1.0
  }
}
```

如果你的项目是使用 git 作为为版本控制工具的话，info 接口还能返回当前应用的 git 信息，包括分支、最新的提交 id、提交时间等等。这个功能依赖于 `GitProperties` 这个类，如果你的 classpath 目录下有 `git.properties` 这个文件的时候，`GitProperties`就会被自动注入，所以你不需要再写任何代码。

默认情况下 info 接口只会返回 `git.branch` `git.commit.id` `git.commit.time`三个属性，如果你需要全部的属性的话，可以通过以下配置实现：

```properties
management.info.git.mode=full
```

info 接口还可以返回当前应用的构建信息，和 git 信息类似，构建信息依赖于 `BuildProperties` 这个类，以及`META-INF/build-info.properties` 这个文件。

##### 如何生成 git.properties

引入插件 `git-commit-id-plugin` 即可，十分简单，具体步骤如下：

1. 在你的 pom 文件中引入插件

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>pl.project13.maven</groupId>
               <artifactId>git-commit-id-plugin</artifactId>
           </plugin>
       </plugins>
   </build>		
   ```

2. 构建你的项目，完成。

`git.properties`长这样：

```properties
#Generated by Git-Commit-Id-Plugin
#Thu Jan 09 11:08:10 CST 2020
git.branch=develop
git.build.host=
git.build.time=2020-01-09T11\:08\:10+0800
git.build.user.email=white.hcj@gmail.com
git.build.user.name=White
git.build.version=1.0-${git.commit.id.abbrev}
git.closest.tag.commit.count=
git.closest.tag.name=
git.commit.id=bfdefa80b76bd7ca37050db0d5b958b915c907be
git.commit.id.abbrev=bfdefa8
git.commit.id.describe=bfdefa8-dirty
git.commit.id.describe-short=bfdefa8-dirty
git.commit.message.full=return request id in error response
git.commit.message.short=return request id in error response
git.commit.time=2020-01-07T21\:43\:16+0800
git.commit.user.email=white.hcj@gmail.com
git.commit.user.name=White
git.dirty=true
git.local.branch.ahead=0
git.local.branch.behind=0
git.remote.origin.url=
git.tags=
git.total.commit.count=94
```

#####  如何生成 build-info.properties

在 springboot 的构建插件中添加以下配置即可

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.2.2.RELEASE</version>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>	
```

#### metrics

metrics 返回的是应用运行过程中的一些动态指标名称列表，比如：内存使用情况，线程信息，GC 信息等等，具体如下：

```json
{
  "names": [
    "jvm.memory.max",
    "jvm.threads.states",
    "process.files.max",
    "jvm.gc.memory.promoted",
    "http.server.requests",
    "system.load.average.1m",
    "jvm.memory.used",
    "jvm.gc.max.data.size",
    "jvm.gc.pause",
    "jvm.memory.committed",
    "system.cpu.count",
    "logback.events",
    "tomcat.global.sent",
    "jvm.buffer.memory.used",
    "tomcat.sessions.created",
    "jvm.threads.daemon",
    "system.cpu.usage",
    "jvm.gc.memory.allocated",
    "tomcat.global.request.max",
    "tomcat.global.request",
    "tomcat.sessions.expired",
    "jvm.threads.live",
    "jvm.threads.peak",
    "tomcat.global.received",
    "process.uptime",
    "tomcat.sessions.rejected",
    "process.cpu.usage",
    "tomcat.threads.config.max",
    "jvm.classes.loaded",
    "jvm.classes.unloaded",
    "tomcat.global.error",
    "tomcat.sessions.active.current",
    "tomcat.sessions.alive.max",
    "jvm.gc.live.data.size",
    "tomcat.threads.current",
    "process.files.open",
    "jvm.buffer.count",
    "jvm.buffer.total.capacity",
    "tomcat.sessions.active.max",
    "tomcat.threads.busy",
    "process.start.time"
  ]
}
```

metrics 接口仅返回指标名称，如果要查看某个指标的详细信息，则要使用 `metrics/{指标名称}` 接口，例如：

`http://localhost:9002/actuator/metrics/jvm.memory.max`


``` json
{
  "name": "jvm.memory.max",
  "description": "The maximum amount of memory in bytes that can be used for memory management",
  "baseUnit": "bytes",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 5583142911
    }
  ],
  "availableTags": [
    {
      "tag": "area",
      "values": [
        "heap",
        "nonheap"
      ]
    },
    {
      "tag": "id",
      "values": [
        "Compressed Class Space",
        "PS Survivor Space",
        "PS Old Gen",
        "Metaspace",
        "PS Eden Space",
        "Code Cache"
      ]
    }
  ]
}
```



### 自定义 EndPoint

除了使用默认的 EndPoint 之外，Actuator 也提供了自定义 EndPoint 的功能，我们可以通过 `@Endpoint` 注解来创建一个新的 EndPoint。`@ReadOperation` `@WriteOperation` `@DeleteOperation` 这三个注解分别表示 get、post、delete 请求到这个 EndPoint 的操作。

```java
@Component
@Endpoint(id = "myEndPoint")
public class MyEndPoint{

    @ReadOperation
    public String read() {
        return "read my endpoint";
    }

    @WriteOperation
    public String write() {
        return "write operation";
    }

    @DeleteOperation
    public String delete() {
        return "delete operation";
    }
}
```