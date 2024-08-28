## 总体介绍
![[无标题 26.png]]
## Nacos简介
### 名字由来
前四个字母分别为**Na**ming和**Co**nfiguration的前两个字母，最后的**s**为Service
### 是什么
![[Pasted image 20240709111029.png]]
#### 一句话总结：
Nacos就是注册中心 + 配置中心的组合； 等价于 Eureka + Config +Bus ；等价于 Spring Cloud Consul
### 能干吗
替代Eureka/Consul做服务注册中心
替代（Config+Bus）/Consul做服务配置中心和满足动态刷新广播通知
### 去哪下
[Nacos 快速开始 | Nacos 官网](https://nacos.io/docs/v2/quickstart/quick-start/)
### 各种注册中心比较
## Nacos下载安装
## Nacos Discovery服务注册中心
### 概述
[Nacos 融合 Spring Boot，成为注册配置中心 | Nacos 官网](https://nacos.io/docs/v2/ecology/use-nacos-with-spring-boot/)
![[Pasted image 20240709141958.png]]
### SpringCloud Alibaba参考中文文档
[Spring Cloud Alibaba 参考文档 (spring-cloud-alibaba-group.github.io)](https://spring-cloud-alibaba-group.github.io/github-pages/2022/zh-cn/2022.0.0.0-RC2.html#_%E5%A6%82%E4%BD%95%E5%BC%95%E5%85%A5_nacos_discovery_%E8%BF%9B%E8%A1%8C%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E5%8F%91%E7%8E%B0)
### 基于Nacos的服务提供者
#### 建Module
cloudalibaba-provider-payment9001
#### POM

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <parent>        <artifactId>cloud2024</artifactId>  
        <groupId>com.QingJiu.cloud</groupId>  
        <version>1.0-SNAPSHOT</version>  
    </parent>    <modelVersion>4.0.0</modelVersion>  
  
    <artifactId>cloudalibaba-provider-payment9001</artifactId>  
  
    <properties>        <maven.compiler.source>17</maven.compiler.source>  
        <maven.compiler.target>17</maven.compiler.target>  
    </properties>  
    <dependencies>        <!--nacos-discovery-->  
        <dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
        </dependency>        <!-- 引入自己定义的api通用包 -->  
        <dependency>  
            <groupId>com.QingJiu.cloud</groupId>  
            <artifactId>cloud-ai-commons</artifactId>  
            <version>1.0-SNAPSHOT</version>  
        </dependency>        <!--SpringBoot通用依赖模块-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-actuator</artifactId>  
        </dependency>        <!--hutool-->  
        <dependency>  
            <groupId>cn.hutool</groupId>  
            <artifactId>hutool-all</artifactId>  
        </dependency>        <!--lombok-->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <version>1.18.28</version>  
            <scope>provided</scope>  
        </dependency>        <!--test-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>    </dependencies>  
    <build>        <plugins>            <plugin>                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>        </plugins>    </build>  
  
</project>
```

#### YAML

```YAML
server:  
  port: 9001  
  
spring:  
  application:  
    name: nacos-payment-provider  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848 #配置Nacos地址
```

#### 主启动

```Java
package com.QingJiu.cloud;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;  
  
@EnableDiscoveryClient //只要做服务注册和发现的必要的启动Discovery  
@SpringBootApplication  
public class Main9001 {  
  
    public static void main(String[] args) {  
        SpringApplication.run(Main9001.class,args);  
        System.out.println("payment9001微服务启动成功（づ￣3￣）づ╭❤️～");  
  
    }  
}
```

#### 业务类

```java
package com.QingJiu.cloud.controller;  
  
import lombok.Getter;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.PathVariable;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
  
@RequestMapping("/pay")  
@RestController  
public class PayAlibabaController {  
    @Value("{server.port}")  
    private String serverPort;  
  
    @GetMapping("/nacos/{id}")  
    public String getPayInfo(@PathVariable("id") Integer id){  
        return "nacos registry,serverPort：" + serverPort + "\t id " + id;  
    }  
  
}
```


### 基于Nacos的服务消费者
#### 建Module
cloudalibaba-consumer-nacos-order83
#### POM

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <parent>        <artifactId>cloud2024</artifactId>  
        <groupId>com.QingJiu.cloud</groupId>  
        <version>1.0-SNAPSHOT</version>  
    </parent>    <modelVersion>4.0.0</modelVersion>  
  
    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>  
  
    <properties>        <maven.compiler.source>17</maven.compiler.source>  
        <maven.compiler.target>17</maven.compiler.target>  
    </properties>  
    <dependencies>        <!--nacos-discovery-->  
        <dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
        </dependency>        <!--loadbalancer-->  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
        </dependency>        <!--web + actuator-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-actuator</artifactId>  
        </dependency>        <!--lombok-->  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>    </dependencies>  
    <build>        <plugins>            <plugin>                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>        </plugins>    </build>  
  
  
</project>
```

#### YAML

```yaml
server:  
  port: 83  
  
spring:  
  application:  
    name: nacos-order-consumer  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848  
  
#消费者将要去访问的微服务名称（nacos微服务提供者叫什么就写什么）  
service-url:  
  nacos-user-service: http://nacos-payment-provider
```

#### 主启动

```java
package com.QingJiu.cloud;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;  
  
@EnableDiscoveryClient  
@SpringBootApplication  
public class Main83 {  
    public static void main(String[] args) {  
        SpringApplication.run(Main83.class,args);  
        System.out.println("客户微服务order83启动成功<(￣︶￣)↗[GO!]");  
    }  
}
```

#### 业务类

```java
package com.QingJiu.cloud.controller;  
  
import jakarta.annotation.Resource;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.PathVariable;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
import org.springframework.web.client.RestTemplate;  
  
@RequestMapping("consumer")  
@RestController  
public class OrderNacosController {  
  
    @Resource  
    private RestTemplate restTemplate;  
  
    @Value("${service-url.nacos-user-service}")  
    private String serverURL;  
  
    @GetMapping("/pay/nacos/{id}")  
    public String paymentInfo(@PathVariable("id") Integer id){  
        String forObject = restTemplate.getForObject(serverURL + "/pay/nacos/" + id, String.class);  
        return forObject + "\t" + "    我是OrderNaconstroller83调用者.....(｡･∀･)ﾉﾞ嗨";  
    }  
  
}
```

### 负载均衡
#### 拷贝虚拟端口映射
![[Pasted image 20240709170356.png]]
![[Pasted image 20240709170359.png]]
![[Pasted image 20240709170402.png]]



## Nacos Config服务配置中心
### 作为配置中心配置步骤
#### 建Module
#### POM
#### YAML
Nacos同Consul一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动，为了满足动态刷新和全局广播通知springboot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application
#### 主启动
#### 业务类
#### 添加配置信息
#### 自带动态刷新

## Nacos数据模型之Namespace-Group-Datald
### 问题
实际开发中，通常一个系统会准备
dev开发环境
test测试环境
prod生产环境。

如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢？


一个大型分布式微服务系统会有很多微服务子项目，
每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境......

那怎么对这些微服务配置进行分组和命名空间管理呢？

### 官网
### Namespace+Group+DataId三者关系

### Nacos的图形化管理界面
### 三种方案加载配置
#### DataID方案
指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置
##### 默认空间+默认分组+新建DataID

#### Group方案
##### 默认空间+新建分组+新建DataID

###### 新建prod配置DataID

###### 新建Group

#### Namespace方案
##### 通过Namespace实现命名空间环境区分
##### Prod_Namespace+PROD_DataID
##### 修改YAML
