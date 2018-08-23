---
title: Java 日志框架 Logback
description: 
categories:
 - Java
tags: Java Logback
---

## Logback 简介
Logback 是一个稳定、高效、快速的 Java 日志框架，作为 log4j 的改良版，它与 log4j 相比拥有更多特性，也带来了很大的性能提升，具体改进可以查看[官方文档](https://logback.qos.ch/reasonsToSwitch.html)。

<!-- more -->

Logback 主要分为三个模块
- logback-core：核心模块，作为 classic 和 access 模块的基础
- logback-classic：实现了 slf4j API，配合 slf4j 使用，可以方便的切换其他日志框架
- logback-access：​​与Servlet容器（如Tomcat和Jetty）集成，提供了 HTTP 访问日志接口

## Logback 加载
Logback 启动加载时会按一下顺序查找配置文件
1. 在系统配置文件 System Properties 中寻找是否有 logback.configurationFile 对应的 value
2. 在 classpath 下寻找是否有 logback.groovy（logback支持groovy与xml两种配置方式）
3. 在 classpath 下寻找是否有 logback-test.xml
4. 在 classpath 下寻找是否有 logback.xml

当查找到任意一项配置存在后就不进行后续扫描了，会使用该配置文件进行初始化，如果没有查找到配置文件，Logback会创建一个向控制台输出日志的配置。

## Logback 配置

### 根节点 configuration

configuration 是配置文件的根节点，有三个属性：
- debug：默认值为false，设置为true时，将打印出 logback 内部日志信息，实时查看 logback 运行状态。
- scan：默认值为true，设置为true时，配置文件如果发生改变，将会被重新加载。
- scanPeriod：当 scan 为 true 时，此属性才会生效。设置扫描配置文件是否有修改的时间间隔，默认单位是毫秒。默认的时间间隔为1分钟（60 second）

配置代码：

```xml
<configuration scan="true" scanPeriod="60 second" debug="true">
</configuration>
```

### 设置上下文名称 contextName
每个 logger 都关联到 logger 上下文，默认上下文名称为 “default”。但可以使用<contextName\>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。   

配置代码：

```xml
<contextName>new context name</contextName>
```

### 设置变量 property
property 是用来定义变量的标签，设置之后可以用 ${变量名} 访问，有三个属性：   
- name：变量名称
- value：变量值
- file：指定配置文件的路径，它的作用在于，如果有多个配置信息的话，可以直接写在配置文件中，然后通过file引入
- resource：作用与 file 一样，不同的是它可以直接从 classpath 路径下引入配置文件

配置代码：

```xml
<!-- name value 形式 -->
<property name="APP_Name" value="MyApp"/>
<contextName>${APP_Name}</contextName>  
```

```xml
<!-- file 形式 -->
variables.properties:
APP_Name=MyApp
LOG_PATH=logs

<property file="src/main/java/config/variables.properties" />
<contextName>${APP_Name}</contextName> 
```

```xml
<!-- resource 形式 -->
variables.properties:
APP_Name=MyApp
LOG_PATH=logs

<property resource="variables.properties" />
<contextName>${APP_Name}</contextName> 
```

### configuration 子节点 logger、root

logger 用来设置某一个类或者某个包的日志输出级别、以及输出位置（指定 appender），有三个属性：
- name：指定的包名或者类名
- level：输出日志级别，如果未设置此级别，那么当前 logger 会向上继承最近一个非空级别，root 默认有一个 level 为 debug
- additivity：是否将日志向上级传递，默认为 true

logger 通过设置子节点 appender-ref 来指定日志输出位置，一个 logger 可以设置多个 appender-ref   
配置代码：

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <Pattern>[%d{HH:mm:ss.SSS}] [%5level] [%thread] %logger{36} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
</appender>
<logger name="X" level="INFO" additivity="false">
        <appender-ref ref="STDOUT"/>
</logger>
<logger name="X.Y" additivity="false">
    <appender-ref ref="STDOUT"/>
</logger>
```

root 是一个特殊的 logger , 是所有 logger 的根节点，因为已经被命名为 root 同时也没有父级别，所以只有一个属性 level，默认为 DEBUG   
配置代码：

```xml
<root level="DEBUG">
    <appender-ref ref="STDOUT"/>
    <appender-ref ref="ASYNC"/>
</root>
```

level 继承示例1：

| logger name | level | 实际 level |
| :-------- | :--------:| :--: |
| root | DEBUG | DEBUG |
| X | 未设置 | DEBUG |
| X.Y | 未设置 | DEBUG |
| X.Y.Z | 未设置 | DEBUG |

示例1只有 root 设置了一个级别，X，X.Y 和 X.Y.Z 这三个 logger 未设置日志输出级别，因此向上继承 root 的级别，即 DEBUG

level 继承示例2：

| logger name | level | 实际 level |
| :-------- | :--------:| :--: |
| root | ERROR | ERROR |
| X | INFO | INFO |
| X.Y | DEBUG | DEBUG |
| X.Y.Z | WARN | WARN |

示例2所有 logger 都设置了一个日志级别，等级继承不起作用。

level 继承示例3：

| logger name | level | 实际 level |
| :-------- | :--------:| :--: |
| root | DEBUG | DEBUG |
| X | INFO | INFO |
| X.Y | 未设置 | INFO |
| X.Y.Z | WARN | WARN |

示例3 X.Y 没有设置日志级别，向上继承最近一个有日志级别的 logger X 的值。

level 继承示例4：

| logger name | level | 实际 level |
| :-------- | :--------:| :--: |
| root | DEBUG | DEBUG |
| X | INFO | INFO |
| X.Y | 未设置 | INFO |
| X.Y.Z | 未设置 | INFO |

示例4 X.Y 和 X.Y.Z 没有设置日志级别，向上继承最近一个有日志级别的 logger X 的值。


### configuration 子节点 appender

appender 是负责写日志的组件，有两个属性（使用时都必须配置）：
- name：设置 appender 的名称，供后面 logger 设置引用
- class：设置 appender 的全路径类名，例：ch.qos.logback.core.ConsoleAppender

appender 可以包含零个或一个 layout ，零个或多个 encoder 元素以及零个或多个 filter 元素。除了这三个元素之外，还可以包含与 appender 类的 JavaBean 属性相对应的任意数量的元素，如： file 指定日志文件名称。

- layout：对日志进行格式化
- encoder： encoder 是0.9.19版本之后引进的，以前的版本使用 layout ，logback极力推荐的是使用 encoder 而不是 layout 
- filter：对日志进行过滤

appender 常用的有以下几种：
1. ConsoleAppender：输出到控制台，或者更确切地说是输出到 System.out 或 System.err，前者是默认目标。

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
    <target>
        System.err
    </target>
</appender>
```

2. FileAppender：输出到文件，目标文件由 file 指定。如果该文件已存在，则根据 append 属性的值确定追加或者清空文件。

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>testFile.log</file>
    <append>true</append>
    <!-- set immediateFlush to false for much higher logging throughput -->
    <immediateFlush>true</immediateFlush>
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```

3. RollingFileAppender：继承自 FileAppender ,提供了滚动记录的功能，先将日志记录到指定文件，当触发某个条件时，将日志记录到其他文件。有两个重要的子节点 rollingPolicy  和  triggeringPolicy 
- rollingPolicy：指定发生滚动时 RollingFileAppender 的行为，例如可以切换日志文件
- triggeringPolicy：指定 RollingFileAppender 何时触发滚动。

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logFile.log</file>
    <!-- 根据时间来制定滚动策略，既负责滚动也负责触发滚动。 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- 每天生成日志文件 -->
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>

      <!-- 保留最近30天的日志文件 -->
      <maxHistory>30</maxHistory>
      <totalSizeCap>3GB</totalSizeCap>

    </rollingPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender> 

<appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>mylog.txt</file>
    <!-- 基于文件大小和时间的滚动策略 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- 每天生成日志文件 -->
      <fileNamePattern>mylog-%d{yyyy-MM-dd}.%i.txt</fileNamePattern>
       <!-- 每个日志文件最多 100MB, 保留 60 天, 最多 20 GB-->
       <maxFileSize>100MB</maxFileSize>    
       <maxHistory>60</maxHistory>
       <totalSizeCap>20GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
</appender>
```

4. AsyncAppender：异步记录日志，它仅仅作为一个调度者，因此必须引用另一个 appender 来做日志输出。

- discardingThreshold：默认情况下，当阻塞队列剩余容量为20％时，它将丢弃级别为 TRACE，DEBUG 和 INFO 的事件，仅保留级别为 WARN 和 ERROR 的事件。设置为0即可保留所有事件。
- queueSize：阻塞队列的最大容量。默认情况下，queueSize 为 256。
- appender-ref：表示 AsyncAppender 使用哪个具体的 appender 进行日志输出。

```xml
<!-- 异步输出 -->  
<appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">  
        <!-- 不丢失日志 -->  
        <discardingThreshold>0</discardingThreshold>  
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->  
        <queueSize>256</queueSize>  
        <!-- 添加附加的appender,最多只能添加一个 -->  
        <appender-ref ref ="FILE"/>  
</appender>
     
<logger name="X" level="DEBUG">
    <appender-ref ref="ASYNC" />
</logger>
```

