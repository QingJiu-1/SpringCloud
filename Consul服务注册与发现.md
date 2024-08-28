## 为什么要引入服务注册中心
##### 为什么引入？
> 微服务所在的IP地址和端口号硬编码到订单微服务中，会存在非常多的问题
> 
> （1）如果订单微服务和支付微服务的IP地址或者端口号发生了变化，则支付微服务将变得不可用，需要同步修改订单微服务中调用支付微服务的IP地址和端口号。
> 
> （2）如果系统中提供了多个订单微服务和支付微服务，则无法实现微服务的负载均衡功能。
> 
> （3）如果系统需要支持更高的并发，需要部署更多的订单微服务和支付微服务，硬编码订单微服务则后续的维护会变得异常复杂。
> 
> 所以，在微服务开发的过程中，需要引入服务治理功能，实现微服务之间的动态注册与发现，从此刻开始我们正式进入SpringCloud实战

## 为什么不再使用传统老牌的Eureka
##### 1、Eureka停更进维
##### 2、Eureka对初学者不友好
##### 3、注册中心独立且和微服务功能解耦
> 目前主流服务中心，希望单独隔离出来而不是作为一个独立微服务嵌入到系统中
> 
> 按照Netflix的之前的思路，注册中心Eureka也是作为一个微服务且需要程序员自己开发部署；
> 
> 实际情况，
> 
> 希望微服务和注册中心分离解耦，注册中心和业务无关的，不要混为一谈。
> 
> 提供类似tomcat一样独立的组件，微服务注册上去使用，是个成品。
##### 4、阿里巴巴Nacos的崛起

## consul简介
##### 是什么
###### ######官网地址：[Consul by HashiCorp](https://www.consul.io/)
###### What is Consul:[What is Consul? | Consul | HashiCorp Developer](https://developer.hashicorp.com/consul/docs/intro)
> Consul <font color="#ff0000">是一套开源的分布式服务发现和配置管理系统</font>，由 HashiCorp 公司用 Go 语言开发。
> 
> 提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows

###### Spring Cloud Consul：[Spring Cloud Consul](https://spring.io/projects/spring-cloud-consul)
##### 能干嘛
###### 服务发现
提供HTT和DNS两种发现方式。
###### 健康检测
支持多种方式，HTTP、TCP、Docker、Shell脚本定制进行监控
###### KV存储
Key、Value的存储方式
###### 多数据中心
Consul支持多数据中心
###### 可视化Web界面
##### 去哪下
下载地址： [Install | Consul | HashiCorp Developer](https://developer.hashicorp.com/consul/install)
##### 怎么玩
![[consul的两大功能.png]]
## 安装并运行consul
##### 安装：
下载完成并解压即可
##### 运行：
安装和登录介绍：[安装 Consul ：： Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/install.html)或者[spring-cloud/spring-cloud-consul: Spring Cloud Consul (github.com)](https://github.com/spring-cloud/spring-cloud-consul)
![[登录consul的介绍.png]]
使用开发者模式启动：consul agent -dev
访问consul的首页：[Services - Consul](http://localhost:8500/ui/dc1/services)
![[consul安装运行成功.png]]


## 服务注册与发现

##### 服务提供者8001

###### 支付服务pay8001注册进consul
###### POM
插件配置来源：[Quick Start :: Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/quickstart.html)

```xml
<!--SpringCloud consul discovery -->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>  
</dependency>
```

###### YAML

```yaml
cloud:  
  consul:  
    host: localhost  
    port: 8500  
    discovery:  
      service-name: ${spring.application.name}
```

###### 主启动

```java
package com.QingJiu.cloud;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;  
import tk.mybatis.spring.annotation.MapperScan;  
  
@EnableDiscoveryClient//开启服务发现  
@SpringBootApplication  
@MapperScan("com.QingJiu.cloud.mapper")  
public class Main8001 {  
    public static void main(String[] args) {  
        SpringApplication.run(Main8001.class,args);  
        System.out.println("Hello world!");  
    }  
}
```

###### 启动8001并查看consul控制台
![[cloud-payment-service注册.png]]

###### 服务消费者80

###### 修改微服务cloud-consumer-order80

```java
package com.QingJiu.cloud.controller;  
  
import com.QingJiu.cloud.entities.PayDTO;  
import com.QingJiu.cloud.resp.ResultData;  
import jakarta.annotation.Resource;  
import org.springframework.web.bind.annotation.*;  
import org.springframework.web.client.RestTemplate;  
  
@RestController  
@RequestMapping("consumer")  
public class OrderController {  
  
    // (url, requestMap, ResponseBean.class)需要这三个参数  
    //url = 请求地址  
    public static final String PaymentSrv_URL = "http://cloud-payment-service";//解决硬编码问题  
  
    @Resource  
    private RestTemplate restTemplate;  
  
    //添加  
    @GetMapping("/pay/add")  
    public ResultData addOrder(PayDTO payDTO){  
        // (url, requestMap, ResponseBean.class)需要这三个参数  
        return restTemplate.postForObject(PaymentSrv_URL+"/pay/add",payDTO,ResultData.class);  
    }  
  
    //查询  
    @GetMapping("/pay/get/{id}")  
    public ResultData getPayInfo(@PathVariable("id") Integer id ){  
        return restTemplate.getForObject(PaymentSrv_URL+"/pay/get/"+id,ResultData.class,id);  
    }  
  
    //删除  
    @DeleteMapping("/pay/del/{id}")  
    public ResultData deletePay(@PathVariable("id") Integer id){  
        restTemplate.delete(PaymentSrv_URL+"/pay/del/"+id,id);  
        return null;    }  
  
    //修改  
    @PutMapping("/pay/update")  
    public ResultData updatePay(@RequestBody PayDTO payDTO){  
        restTemplate.put(PaymentSrv_URL+"/pay/update",payDTO,ResultData.class);  
        return null;    }  
  
    //查询全部  
    @GetMapping("/pay/get/all")  
    public ResultData getAll(){  
        return restTemplate.getForObject(PaymentSrv_URL+"/pay/get/all",ResultData.class);  
    }  
  
  
}
```


###### POM
插件配置来源：[Quick Start :: Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/quickstart.html)

```xml
<!--SpringCloud consul discovery -->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>  
</dependency>
```

###### YAML

```java
spring:  
  application:  
    name: cloud-consumer-order  
  
  cloud:  
    consul:  
      host: localhost  
      port: 8500  
      discovery:  
        service-name: ${spring.application.name}
```

###### 启动80并查看consul控制台
![[80的consul控制台.png]]

###### 配置修改RestTemplateConfig

```java
package com.QingJiu.cloud.config;  
  
  
import org.springframework.cloud.client.loadbalancer.LoadBalanced;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.client.RestTemplate;  
  
@Configuration  
public class RestTemplateConfig {  
  
    //使用http连接，访问第三方网络接口  
    @Bean  
    @LoadBalanced//就能让这个给restTemplate在请求时拥有客户端负载均衡的能力  
    public RestTemplate restTemplate(){  
        return new RestTemplate();  
    }  
  
}
```

###### 访问地址测试
![[Consul注册连接测试.png]]
## 服务配置与刷新
##### 分布式系统面临的 配置问题
![[问题介绍.png]]
> 微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。比如某些配置文件中的内容大部分都是相同的，只有个别的配置项不同。就拿数据库配置来说吧，如果每个微服务使用的技术栈都是相同的，则每个微服务中关于数据库的配置几乎都是相同的，有时候主机迁移了，我希望一次修改，处处生效。
> 
> 当下我们每一个微服务自己带着一个application.yml，上百个配置文件的管理....../(ㄒoㄒ)/~~
##### 官网说明
官网地址：[Distributed Configuration with Consul :: Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/config.html)
![[官网说明.png]]

##### 服务配置案例步骤
###### 需求
> 通用全局配置信息，之间注册进Consul服务器，从Consul获取
> 既然从Consul获取自然要遵守Consul的配置规则要求
###### 修改8001
###### POM
###### YAML
配置说明：
![[实现所有微服务下发广播说明.png]]
新增配置文件bootstrap.yaml
> applicaiton.yml是用户级的资源配置项
> 
> bootstrap.yml是系统级的，优先级更加高
> 
> Spring Cloud会创建一个“Bootstrap Context”，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。
> 
> `Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。
> 
>  application.yml文件改为bootstrap.yml,这是很关键的或者两者共存
> 
> 因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml
###### consul服务器key/value配置填写
###### controller
###### 测试
##### 动态刷新案例步骤
###### 问题
我们在consul的dev配置分支修改了内容马上访问，结果没有更改；没有做到及时响应和动态刷新
###### 步骤
@RefreshScope 主启动类添加

```yaml
spring:  
  application:  
    name: cloud-payment-service  
    ####Spring Cloud Consul for Service Discovery  
  cloud:  
    consul:  
      host: localhost  
      port: 8500  
      discovery:  
        service-name: ${spring.application.name}  
      config:  
        profile-separator: '-' # 分隔符默认围为 ","，我们将其修改为 '-'        format: YAML  
        watch:  
          wait-time: 1
```

controller

```java
@Value("${server.port}")//获取配置文件中server.port的属性值  
private String port;  
  
@GetMapping("/get/info")  
public String getInfoByConsul(@Value("${QingJiu.info}") String qingJiuInfo){  
    return "qingJiuInfp："+qingJiuInfo+"\t"+"port："+port;  
}
```

##### 思考
##### 截至到这，服务器配置和动态刷新全部通过，假设重启Consul，之前配置还在吗？
> 不在

###### 引出问题
Consul配置持久化