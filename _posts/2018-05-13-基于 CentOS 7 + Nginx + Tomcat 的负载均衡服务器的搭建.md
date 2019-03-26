---
title: 基于 CentOS 7 + Nginx + Tomcat 的负载均衡服务器的搭建
description:
categories:
 - 服务器
tags: Nginx Tomcat
---

## 前言
本文主要介绍如何在 CentOS 上从零开始使用 Nginx + Tomcat 搭建一个负载均衡服务器。在搭建过程中学习 Nginx 的基本使用方式以及 Tomcat 相关知识，进一步理解两者结合后的运行原理。

<!-- more -->

## Nginx 安装
使用源码编译的方式在 CentOS 上安装 Nginx 主要有以下几个步骤：
1. 安装运行环境  

    - Nginx 是使用 C 语言开发的，编译依赖 gcc 环境，因此如果没有 gcc 需要先安装：

    ``` bash  
    yum install -y gcc-c++
    ```

    - 安装 Nginx 依赖库  

    ``` bash  
    # rewrite模块需要 PCRE(Perl Compatible Regular Expressions),pcre-devel 库
    yum install -y pcre pcre-devel
    # gzip模块需要 zlib 库
    yum install -y zlib
    # ssl 功能需要 OpenSSL 库
    yum install -y openssl openssl-devel
    ```

2. 下载 Ngnix

    从 [Ngnix 官网](http://nginx.org/en/download.html)下载最新稳定版，目前是 [1.12.2](http://nginx.org/download/nginx-1.12.2.tar.gz)
    ``` bash  
    # 如果还未安装 wget 请先执行安装命令
    yum install -y wget
    # 使用 wget 命令下载
    wget -c https://nginx.org/download/nginx-1.12.2.tar.gz
    # 解压文件
    tar -zxvf nginx-1.12.2.tar.gz
    # 进入解压后的目录
    cd nginx-1.12.2
    # 使用默认配置
    ./configure
    ```
3. 编译安装 Nginx

    编译安装十分简单，只需执行两个命令即可
    ``` bash  
    # 编译
    make
    # 安装
    make install
    # 安装目录默认为 /usr/local/nginx
    # 可以使用 whereis nginx 查找
    [root@localhost nginx-1.12.2]# whereis nginx
    nginx: /usr/local/nginx
    ```
4. 启动测试

    Ngnix 安装完成后即可启动测试是否可以正常运行
    ``` bash  
    # 进入 ngnix 执行文件目录
    cd /usr/local/ngnix/sbin
    # 启动
    # ./nginx  启动
    # ./nginx -s stop  停止（先查出 nginx 进程 id 再使用 kill 命令强制杀掉进程）
    # ./nginx -s quit  退出（等待 nginx 进程将任务处理完毕再停止）
    # ./nginx -s reload  重新加载，适用于修改了配置文件之后操作
    ./nginx
    # 启动之后可以通过 ps 命令查询 ngnix 进程
    ps aux|grep nginx
    # 访问localhost:80 测试 ngnix 是否正常运行，返回 nginx 欢迎页表示正常运行
    [root@localhost sbin]# curl localhost:80
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

## Tomcat 多实例部署配置

Tomcat 单实例部署，即一个 Tomcat 服务器运行时，不存在负载均衡一说，因此，我们首先要做的就是实现 Tomcat 多实例部署，将我们的同一个应用程序部署在多个 Tomcat 服务器上同时运行。主要步骤如下：

1. 安装 JDK（如果本机已经安装了则跳过第一步）
``` bash  
# 下载、解压 JDK8，下载地址可以从官网获取
wget http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz?AuthParam=1520068673_6f545cf32470b83658219011266e65b8
# 配置 Java 环境变量
vi /etc/profile
# 在文件尾部添加以下内容
export JAVA_HOME=/usr/local/jdk1.8.0_161 (这里是JDK所在目录)
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
# 保存完成后重新加载 /etc/profile 文件
source /etc/profile
# 测试是否配置正确
[root@localhost local]# java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```
2. 下载 Tomcat
``` bash  
# 下载 Tomcat 9
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.5/bin/apache-tomcat-9.0.5.tar.gz
# 解压
tar -zxvf apache-tomcat-9.0.5.tar.gz
# 为接下来的步骤做准备
# 重命名为tomcat-home
mv apache-tomcat-9.0.5 tomcat-home
# 复制第二份，重命名为tomcat-8080
cp -R tomcat-home tomcat-8080
# 复制第三份，重命名为tomcat-9090
cp -R tomcat-home tomcat-9090
```
3. 分离目录

    首先介绍两个 tomcat 中比较重要的概念（通常也是两个系统变量）**CATALINA_HOME** 和 **CATALINA_BASE**  
    - CATALINA_HOME：即指向Tomcat安装路径的系统变量（安装目录）  
    - CATALINA_BASE：即指向活跃配置路径的系统变量（工作目录）     

    通过设置这两个变量，就可以将tomcat的安装目录和工作目录分离，从而实现tomcat多实例的部署。
    下面就是 Tomcat 的基本目录结构，以及对应的作用。


    | 目录 | 作用 |
    | :-------- | :--------:|
    | bin | 存放脚本文件，例如比较常用的启动和关闭脚本 startup.sh、shutdown.sh 等 |
    | conf | 存放配置文件，最重要的是 server.xml，它是 tomcat 的主要配置文件 |
    | lib | Tomcat 运行需要的依赖包 |
    | log | 存放日志文件 |
    | temp | 存放运行时产生的临时文件 |
    | webapps | web 应用的默认目录 |
    | work | 主要存放由JSP文件生成的servlet（java文件以及最终编译生成的class文件） |



    [Tomcat 官方文档](https://tomcat.apache.org/tomcat-9.0-doc/RUNNING.txt) 说明了   CATALINA_HOME 路径下需要包含 bin 和 lib 目录，也就是两个支持 tomcat 运行的目录，而 CATALINA_BASE 可以包含所有目录，但是 bin 和 lib 不是必须的，缺省时会使用 CATALINA_HOME 中的 bin 和 lib。因此我们就可以使用一个 CATALINA_HOME 和多个 CATALINA_BASE 部署多个实例，这样的好处是便于管理和升级 Tomcat。上一步我们已经复制了三个 Tomcat，它们的作用分别是：


    | 目录 | 作用 |
    | :-------- | :--------:|
    | tomcat-home | 作为 CATALINA_HOME，即只需要保留 bin 和 lib 两个文件夹 |
    | tomcat-8080 | 作为 CATALINA_BASE，需要保留除了 bin 和 lib 之外的其他文件夹，使用 8080 端口 |
    | tomcat-9090 | 同 tomcat-8080，使用 9090 端口 |



    基于 CATALINA_HOME 和 CATALINA_BASE 分离目录

    ``` bash  
    # 根据上面表格整理完目录之后，目录结构如下：
    [root@localhost tomcat-home]# ls
    bin  lib  LICENSE  NOTICE  RELEASE-NOTES  RUNNING.txt
    [root@localhost tomcat-8080]# ls
    conf  LICENSE  logs  NOTICE  RELEASE-NOTES  RUNNING.txt  temp  webapps  work
    [root@localhost tomcat-9090]# ls
    conf  LICENSE  logs  NOTICE  RELEASE-NOTES  RUNNING.txt  temp  webapps  work
    ```

4. 修改 Tomcat 配置文件

    这一步主要是修改 server.xml 中端口的配置，在 server.xml 中配置了四个监听端口，分别是：
    - Server port(默认8005): 监听关闭 tomcat 的 shutdown 命令
    - Connector port(默认8080)：监听 http 请求
    - AJP Connector port(默认8009)：监听 AJP 请求
    - redirectPort(默认8443)：重定向端口，出现在Connector配置中，如果该Connector仅支持非SSL的普通http请求，那么该端口会把https的请求转发到这个Redirect Port指定的端口。

    了解了监听的各个端口的作用之后就可以开始修改 server.xml 了，如果不使用 AJP 请求，那么我们只需要保证多实例中的 Server port 和 Connector port 不同即可。

    ```xml
    # tomcat-8080 保持默认配置即可
    # 修改 tomcat-9090 的配置，修改后的 /usr/local/tomcat-9090/conf/server.xml 内容如下：
    ...
    <Server port="9005" shutdown="SHUTDOWN">
    ...
    <Connector port="9090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    ...
    ```

5. 编写脚本

    前面的准备工作做完之后就可以启动 tomcat 了，但是现在只有 tomcat-home/bin 目录下有 startup.sh 和 shutdown.sh，而我们要启动的 8080 和 9090 两个实例，因此我们就需要编写一段脚本，修改 CATALINA_BASE 来达到分别操作两个实例的目的。

    - 启动脚本 start.sh：

    ``` bash  
    #!/bin/sh
    CUR_DIR=`dirname $BASH_SOURCE`
    export CATALINA_BASE=`readlink -f $CUR_DIR`
    export CATALINA_HOME="/usr/local/tomcat-home"

    if [ -f $CATALINA_HOME/bin/startup.sh ]; then
            $CATALINA_HOME/bin/startup.sh
    else
            echo "$CATALINA_HOME/bin/startup.sh not exist"
    fi

    ```

    - 停止脚本 stop.sh：

    ``` bash  
    #!/bin/sh
    CUR_DIR=`dirname $BASH_SOURCE`
    export CATALINA_BASE=`readlink -f $CUR_DIR`
    export CATALINA_HOME="/usr/local/tomcat-home"

    if [ -f $CATALINA_HOME/bin/shutdown.sh ]; then
            $CATALINA_HOME/bin/shutdown.sh
    else
            echo "$CATALINA_HOME/bin/shutdown.sh not exist"
    fi

    ```

    将这两个脚本文件放在 tomcat-8080 和 tomcat-9090 的根目录下即可


    ``` bash  
    [root@localhost tomcat-8080]# ls
    conf     logs    RELEASE-NOTES  start.sh  temp     work
    LICENSE  NOTICE  RUNNING.txt    stop.sh   webapps

    # 启动 8080 端口的 tomcat
    [root@localhost tomcat-8080]# ./start.sh
    Using CATALINA_BASE:   /usr/local/tomcat-8080
    Using CATALINA_HOME:   /usr/local/tomcat-home
    Using CATALINA_TMPDIR: /usr/local/tomcat-8080/temp
    Using JRE_HOME:        /usr/local/jdk1.8.0_161
    Using CLASSPATH:       /usr/local/tomcat-home/bin/bootstrap.jar:/usr/local/tomcat-home/bin/tomcat-juli.jar
    Tomcat started.
    # 停止
    [root@localhost tomcat-8080]# ./stop.sh
    Using CATALINA_BASE:   /usr/local/tomcat-8080
    Using CATALINA_HOME:   /usr/local/tomcat-home
    Using CATALINA_TMPDIR: /usr/local/tomcat-8080/temp
    Using JRE_HOME:        /usr/local/jdk1.8.0_161
    Using CLASSPATH:       /usr/local/tomcat-home/bin/bootstrap.jar:/usr/local/tomcat-home/bin/tomcat-juli.jar

    ```

6. 测试两个 Tomcat 是否同时正常运行

    为了方便测试，在 /usr/local/tomcat-8080/webapps/ROOT 和 /usr/local/tomcat-9090/webapps/ROOT 目录下分别新建一个 index.html 文件，内容分别为 "tomcat-8080" 和 "tomcat-9090" 便于我们区分。完成之后使用 start.sh 启动两个实例，这里我们同样使用 curl 访问来测试。

    ``` bash  
    # 访问 9090 端口，获取到 9090 的数据
    [root@localhost tomcat-9090]# curl localhost:9090
    <h1> tomcat-9090 </h1>
    # 访问 8080 端口，获取到 8080 的数据
    [root@localhost tomcat-8080]# curl localhost:8080
    <h1> tomcat-8080 </h1>
    ```

## Nginx 与 Tomcat 结合

Ngnix 和 Tomcat 结合实现负载均衡的需求大概是这样的：用户访问服务器的 8888 端口，Ngnix 接收到请求之后转发至 8080 端口或者 9090 端口由 Tomcat 处理。两个 Tomcat 部署了同一个应用，这样就可以实现负载均衡，可以由两个 Tomcat 同时处理用户请求。这里我们以 localhost 为例，开始配置 Ngnix
> 这里使用 localhost 作为示例，正式使用时在配置文件中使用域名替换 localhost 即可

``` bash  
# 进入 ngnix 配置文件的目录
# 默认配置文件是 ngnix.conf
/usr/local/nginx/conf
# 这里我们新建一个 localhost.conf 单独配置，内容如下：
upstream localhost {
        server 127.0.0.1:8080;
        server 127.0.0.1:9090;
}
server
        {
                listen       8888;
                server_name localhost;
                index index.html index.htm index.jsp index.php;

                location / {
                        proxy_pass http://localhost;
                        proxy_set_header X-Real-IP $remote_addr;
                        add_header X-Slave $upstream_addr;
                }

        }

# 接着将在 nginx.conf 中引入 localhost.conf
...
http {
    ...
    # 引入 localhost.conf;
    include /usr/local/nginx/conf/localhost.conf;
    ...
}
...

# 完成配置文件修改之后可以通过 ./nginx -t 命令测试配置文件是否正确
cd /usr/local/nginx/sbin/
./nginx -t
# 出现下面提示表示配置正确，如果有误的话检查配置文件
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

# 重载 ngnix 配置文件
./nginx -s reload
```

至此，Nginx + Tomcat 的负责均衡服务器已经搭建完成，现在访问 localhost:8888 就可以看到两个 Tomcat 都能处理来自 8888 端口的请求了

``` bash  
[root@localhost sbin]# curl localhost:8888
<h1> tomcat-8080 </h1>
[root@localhost sbin]# curl localhost:8888
<h1> tomcat-9090 </h1>
```


## 负载均衡策略

上面我们使用的是 Nginx 默认的负载均衡策略，我们也可以根据自己需求配置其他的策略，Nginx 提供的策略主要有以下几种：

1. 轮询（默认）

    每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器挂掉，能自动忽略该服务器。

    ```yml
    upstream localhost {
        server 127.0.0.1:8080;
        server 127.0.0.1:9090;
    }
    ```
2. 权重

    根据配置的权重去分配给不同的服务器处理，适用于服务器性能有差距的情况，可以个高性能的服务器分配高权重

    ```yml
    upstream localhost {
        server 127.0.0.1:8080 weight=1;
        server 127.0.0.1:9090 weight=3;
    }
    ```
3. IP 绑定 ip_hash

    每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

    ```yml
    upstream localhost {
        ip_hash;
        server 127.0.0.1:8080;
        server 127.0.0.1:9090;
    }
    ```
4. 最少连接

    将请求分配给当前连接数最少的服务器

    ```yml
    upstream localhost {
        least_conn;
        server 127.0.0.1:8080;
        server 127.0.0.1:9090;
    }
    ```

## 总结

以上，在 CentOS 上使用 Nginx + Tomcat 搭建负载均衡服务器已经完成，搭建过程中主要是理解如何利用 CATALINA_HOME 和 CATALINA_BASE 实现 Tomcat 的多实例部署，在完成 Tomcat 多实例部署的基础上结合 Nginx 实现负载均衡就很简单了。
