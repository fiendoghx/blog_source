---
title: Spring-Cloud基础微服务搭建(springboot1)
date: 2019-10-16 10:09:55
categories:
- Java
tags:
- spring-cloud
- 微服务
- 框架搭建
---

### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体实施](#3-目的)
    * [3.1 事前准备](#3-1事前准备)
    * [3.2 实施并验证](#3-2实施并验证)
		* [3.2.1 设置虚机dns](#3-2-1设置虚机dns)
		* [3.2.2 使用maven打包各服务](#3-2-2使用maven打包各服务)
		* [3.2.3 按照一定顺序执行run.sh](#3-2-3按照一定顺序执行run.sh)
			* [3.2.3.1 启动注册中心](#3-2-3-1启动注册中心)
			* [3.2.3.2 启动Zuul网关](#3-2-3-2启动Zuul网关)
			* [3.2.3.3 启动HelloWorld服务](#3-2-3-3启动HelloWorld服务)
* [4.相关说明](#4-相关说明)
* [5.FQA](#5-FQA)

## 1.缘由


身边有小伙伴需要一套spring-cloud微服务基础服务搭建的方法，自己也需要重新整理一下今年遇到的一些问题

## 2.目的

能够根据文档搭建一套简单的微服务框架，并成功访问具体微服务。

## 3.具体实施

spring-boot版本：1.5.19.RELEASE

## 3.1事前准备

* centos虚拟服务器2台: 192.168.1.150/151 
[centos_download](https://wiki.centos.org/Download)
* centos虚拟服务器 
[Jdk8](https://www.oracle.com/technetwork/java/archive-139210.html)
* 开发环境工具：
[maven3+](https://maven.apache.org/) \ [Git](https://git-scm.com/)
* spring-cloud-frame工程: 
[下载demo工程](https://github.com/fiendoghx/spring-cloud-frame)

服务器以及构建工具的事前准备不在本文讨论范围，请自行搜索准备

## 3.2实施并验证

### 3.2.1设置虚机dns

```
#分别在每个虚机使用root用户执行
cat >> /etc/hosts << EOF
192.168.1.150 spring-cloud-01
192.168.1.151 spring-cloud-02
EOF
```

### 3.2.2使用maven打包各服务

```
#打包注册中心Eureka:
cd <下载的目录>/spring-cloud-frame/base-eureka
mvn install
#打包网关Zuul
cd <下载的目录>/spring-cloud-frame/base-zuul
mvn install
#打包HelloWorld服务
cd <下载的目录>/spring-cloud-frame/service-helloworld
mvn install
```

以上步骤能得到base-eureka-1.0.jar/base-zuul-1.0.jar/service-helloworld-1.0.jar三个jar包

把对应的jar包上传到对应resource目录,工程中的可执行文件run.sh上传到bin目录

```
#分别在每个虚机创建对应目录
mkdir -p /home/fiendo/spring-cloud-frame/eureka/resource
mkdir -p /home/fiendo/spring-cloud-frame/eureka/bin
mkdir -p /home/fiendo/spring-cloud-frame/zuul/resource
mkdir -p /home/fiendo/spring-cloud-frame/zuul/bin
mkdir -p /home/fiendo/spring-cloud-frame/service-helloworld/resource
mkdir -p /home/fiendo/spring-cloud-frame/service-helloworld/bin
```

**采用scp命令把各个基础jar包传到准备好的虚机上，目录保持一致**

```
scp /home/fiendo/spring-cloud-frame/eureka/resource/base-eureka-1.0.jar root@spring-cloud-02:/home/fiendo/spring-cloud-frame/eureka/resource
scp /home/fiendo/spring-cloud-frame/zuul/resource/base-zuul-1.0.jar root@spring-cloud-02:/home/fiendo/spring-cloud-frame/zuul/resource
scp /home/fiendo/spring-cloud-frame/service-helloworld/resource/service-helloworld-1.0.jar root@spring-cloud-02:/home/fiendo/spring-cloud-frame/service-helloworld/resource
```

**用root给各个虚机上run.sh加上可执行权限**

```
chmod +x /home/fiendo/spring-cloud-frame/eureka/bin/run.sh
chmod +x /home/fiendo/spring-cloud-frame/zuul/bin/run.sh
chmod +x /home/fiendo/spring-cloud-frame/service-helloworld/bin/run.sh
```

**解决windows环境下run.sh出现的编码问题**

```
#直接执行./run.sh会由于windows下回车的编码问题报错，采用sed命令替换处理后可正常执行
sed -i 's/\r//' /home/fiendo/spring-cloud-frame/eureka/bin/run.sh
sed -i 's/\r//' /home/fiendo/spring-cloud-frame/zuul/bin/run.sh
sed -i 's/\r//' /home/fiendo/spring-cloud-frame/service-helloworld/bin/run.sh
```

**采用scp命令把处理ok的可执行文件传到其他节点**

```
scp /home/fiendo/spring-cloud-frame/eureka/bin/run.sh root@spring-cloud-02:/home/fiendo/spring-cloud-frame/eureka/bin
scp /home/fiendo/spring-cloud-frame/zuul/bin/run.sh root@spring-cloud-02:/home/fiendo/spring-cloud-frame/zuul/bin
scp /home/fiendo/spring-cloud-frame/service-helloworld/bin/run.sh root@spring-cloud-02:/home/fiendo/spring-cloud-frame/service-helloworld/bin
```

### 3.2.3按照一定顺序执行run.sh

### 3.2.3.1启动注册中心

**对不同虚机的eureka目录run.sh进行编辑**

确定虚机对应的server文件

|参数  | 说明 |
| :----------:| :-----------:| 
|spring-cloud-01   |[server1](https://github.com/fiendoghx/spring-cloud-frame/blob/base-springboot1/base-eureka/src/main/resources/config/application-server1.yml)      |
|spring-cloud-02   |[server2](https://github.com/fiendoghx/spring-cloud-frame/blob/base-springboot1/base-eureka/src/main/resources/config/application-server2.yml)      |

```
#登陆spring-cloud-01启动eureka
cd /home/fiendo/spring-cloud-frame/eureka/bin
./run.sh start --spring.profiles.active=server1
#登陆spring-cloud-02启动eureka
cd /home/fiendo/spring-cloud-frame/eureka/bin
./run.sh start --spring.profiles.active=server2
```

**验证结果**

登陆server1的[Eureka](http://192.168.1.150:12001/)，由于配置了Security，需要用户名密码，可以在配置文件查看 [点击查看](https://github.com/fiendoghx/spring-cloud-frame/blob/base-springboot1/base-eureka/src/main/resources/application.yml)

若无异常会看到有两个Eureka节点已经启动

![Eureka](./img/article/springcloudeureka01.png)

### 3.2.3.2启动Zuul网关

**登陆各虚机启动Zuul**

```
#登陆spring-cloud-01启动zuul
cd /home/fiendo/spring-cloud-frame/zuul/bin
./run.sh start
#登陆spring-cloud-02启动zuul
cd /home/fiendo/spring-cloud-frame/zuul/bin
./run.sh start
```

**验证结果**

若无异常会看到有两个Zuul节点已经启动

![Eureka](./img/article/springcloudzuul01.png)

### 3.2.3.3启动HelloWorld服务

```
#登陆spring-cloud-01启动service-helloworld
cd /home/fiendo/spring-cloud-frame/service-helloworld/bin
./run.sh start
#登陆spring-cloud-02启动service-helloworld
cd /home/fiendo/spring-cloud-frame/service-helloworld/bin
./run.sh start
```

**验证结果**

若无异常会看到有两个service-helloworld节点已经启动

![Eureka](./img/article/springcloudhelloworld01.png)

**通过Zuul网关访问Hello-World微服务**

本文采用Zuul网关来控制各个服务的访问，路由采用默认配置，也就是说访问url为：http://<网关host>/<服务名>/<api>

访问具体服务 http://192.168.1.150:12002/service-helloworld/helloworld

![HelloWorld](./img/article/springcloudhelloworld02.png)

**至此我们就已经搭建完毕一套分布式的微服务基础框架**

## 4.相关说明</h2>

本文基于springboot1搭建的微服务，最初spring-cloud官方给出的方案就是eureka+zuul,本文也是根据最初的方案进行搭建。直到spring-boot2之后迟迟等不到zuul2的更新，才开始出现的gateway，具体springboot2搭建，移步[传送门](https://fiendoghx.github.io/2019/10/20/Spring-Cloud%E5%9F%BA%E7%A1%80%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%90%AD%E5%BB%BA(springboot2)/)

## 5.FQA

### 5.1Eureka在不同虚机为什么要采用不同配置?

由于注册中心本身是相互注册的，不会注册到自己本身，所以不同配置其实是除去了自身，添加其他的注册中心地址而已，类似Zookeeper。

**非常欢迎提问，所有的提问会总结写入FQA之中**