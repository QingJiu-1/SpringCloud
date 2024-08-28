## 提问
已经有loadbalancer为什么还要学习OpenFeign？
两个都有道理的话，日常用哪个？
## 是什么
##### 官网介绍：[Spring Cloud OpenFeign 功能 ：： Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html)
![[Feign功能翻译介绍.png]]
##### GitHub介绍：[spring-cloud/spring-cloud-openfeign: Support for using OpenFeign in Spring Cloud apps (github.com)](https://github.com/spring-cloud/spring-cloud-openfeign)

##### 一句话介绍：
OpenFeign是一个声明式的Web服务客户端，只需创建一个Rest接口并在该接口上添加注解@FeignClient即可，OpenFeign基本上就是当前微服务之间调用的事实标准


## 能干嘛
###### OpenFeign能干什么

> 前面在使用**SpringCloud LoadBalancer**+RestTemplate时，利用RestTemplate对http请求的封装处理形成了一套模版化的调用方法。**_但是在实际开发中，_**
> 
> 由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，OpenFeign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在OpenFeign的实现下，我们只需创建一个接口并使用注解的方式来配置它(在一个微服务接口上面标注一个**_@FeignClient_**注解即可)，即可完成对服务提供方的接口绑定，统一对外暴露可以被调用的接口方法，大大简化和降低了调用客户端的开发量，也即由服务提供者给出调用接口清单，消费者直接通过OpenFeign调用即可，O(∩_∩)O。
> 
> OpenFeign同时还集成SpringCloud LoadBalancer
> 
> 可以在使用OpenFeign时提供Http客户端的负载均衡，也可以集成阿里巴巴Sentinel来提供熔断、降级等功能。而与SpringCloud LoadBalancer不同的是，通过OpenFeign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。


> 可插拔的注解支持，包括Feign注解和JAX-RS注解
> 支持可插拔的HTTP编码器和解码器
> 支持Sentinel和它的Fallback
> 支持SpringCloudLoadBalancer的负载均衡
> 支持HTTP请求和响应的压缩
## OpenFeign通用步骤(怎么玩)
##### 接口+注解
###### 微服务API接口+@FeignClient注解标签
###### 架构说明图
![[架构说明图.png]]
服务消费者80 -> 调用含有@FeignClient注解的API服务接口->服务提供者（8001/8002）
##### 流程步骤
###### 建Module
cloud-consumer-feign-order80
###### 改POM
只多了一个openfeign的依赖其他与cloud-consumer-order80一致
```xml
<!--openfeign-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-openfeign</artifactId>  
</dependency>
```

###### 写YAML

```YAML
server:  
  port: 80  
  
spring:  
  application:  
    name: cloud-consumer-openfeign-order  
  ####Spring Cloud Consul for Service Discovery  
  cloud:  
    consul:  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true #优先使用服务ip进行注册  
        service-name: ${spring.application.name}
```

###### 主启动

```java
@EnableDiscoveryClient//向consul为注册中心时注册服务  
@EnableFeignClients//开启激活feign客户端，定义服务+绑定接口，以声明式的方式优雅而简单的实现服务调用  
@SpringBootApplication  
public class MainOpenFeign80 {  
  
    public static void main(String[] args) {  
        SpringApplication.run(MainOpenFeign80.class,args);  
        System.out.println("openfeign启动");  
    }  
}
```

###### 业务类
- 修改cloud-api-commons通用模块
	引入openfeign依赖
```xml
_<!--openfeign-->  
_<**dependency**>    <**groupId**>org.springframework.cloud</**groupId**>    <**artifactId**>spring-cloud-starter-openfeign</**artifactId**>  
</**dependency**>
```

- 拷贝之前的80工程到cloud-consumer-feign-order80
- 修改Controller层的调用
##### 测试
##### 小总结
## OpenFeign高级特性
![[feign流程简图.png]]
#### OpenFeign超时控制
##### 最新版与之前的有所不同
> 在Spring Cloud微服务架构中，大部分公司都是利用OpenFeign进行服务间的调用，而比较简单的业务使用默认配置是不会有多大问题的，但是如果是业务比较复杂，服务要进行比较繁杂的业务计算，那后台很有可能会出现Read Timeout这个异常，因此定制化配置超时时间就有必要了。

##### 超时设置，故意设置超时演示出错情况
###### 服务提供方cloud-provider-payment8001故意暂停62秒

```java
//暂停62秒钟线程，故意写bug，测试出feign的默认调用超出时间  
try {  
    TimeUnit.SECONDS.sleep(62);  
}catch (Exception e){  
    e.printStackTrace();  
}
```

###### 服务调用方cloud-consumer-feign-order80写好捕捉超时异常

```java
ResultData resultData = null;  
try{  
    System.out.println("调用开始----：" + DateUtil.now());  
    resultData = payFeignApi.getPayInfo(id);  
}catch (Exception e){  
    e.printStackTrace();  
    System.out.println("调用结束----：" + DateUtil.now());  
    ResultData.fail(ReturnCodeEnum.RC500.getCode(), e.getMessage());  
}
```

###### 测试
![[Pasted image 20240702234516.png]]
###### 结论
OpenFeign默认等待60秒钟，超时后报错

##### 官网解释+配置处理
###### 两个关键参数
![graphic](data:application/octet-stream;base64,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)

> 默认OpenFeign客户端等待60秒钟，但是服务端处理超过规定时间会导致Feign客户端返回报错。
> 
> 为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制，默认60秒太长或者业务时间太短都不好
> 
> yaml文件中开启配置：
> 
> connectTimeout       连接超时时间
> 
> readTimeout             请求处理超时时间

###### 超时配置参考官网要求
![[image/无标题 1.png]]

##### 修改cloud-consumer-feign-order80YAML文件里需要开启OpenFeign客户端超时控制
###### 官网出处
![[无标题 1 1.png]]
###### 全局配置
关键内容
all
3秒测试
##### 指定配置
单个服务配置超时时间
all
5秒测试

#### OpenFeign重试机制
##### 步骤
###### 默认重试机制是关闭的
官网说明：[Spring Cloud OpenFeign 功能 ：： Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html)
![[Pasted image 20240703173752.png]]

###### 默认关闭重试机制
只会调用一次后就结束
![[无标题 2.png]]
###### 开启Retryer功能
新增配置类FeignConfig并修改Retryer配置

```java
@Configuration  
public class FeignConfig {  
  
    @Bean  
    public Retryer myRetryer(){  
        //初始间隔时间为100ms,重试间最大间隔时间为1s，最大请求次数为3（1+2）  
        return new Retryer.Default(100,1,3);  
    }  
  
}
```

#### OpenFeign默认HttpClient修改
##### 是什么
###### OpenFeign中http client
> 如果不做特殊配置，OpenFeign默认使用JDK自带的HttpURLConnection发送HTTP请求，
> 
> 由于默认HttpURLConnection没有连接池、性能和效率比较低，如果采用默认，性能上不是最牛B的，所以加到最大。
##### 替换之前，还是按照超时报错的案例
![[Pasted image 20240703175657.png]]

##### Apache HttpClient5替换
###### why?
地址：[Spring Cloud OpenFeign 功能 ：： Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html#spring-cloud-feign-overriding-defaults)
![[Pasted image 20240703180205.png]]
###### 修改微服务feign80
FeignConfig类里面将Retryer属性修改为默认

```java
@Bean  
public Retryer myRetryer(){  
  
    return Retryer.NEVER_RETRY;//Feign默认配置是不走重试策略的  
  
    //初始间隔时间为100ms,重试间最大间隔时间为1s，最大请求次数为3（1+2）  
    //return new Retryer.Default(100,1,3);  
}
```

POM修改

```xml
_<!-- httpclient5-->  
_<**dependency**>    <**groupId**>org.apache.httpcomponents.client5</**groupId**>    <**artifactId**>httpclient5</**artifactId**>    <**version**>5.3</**version**>  
</**dependency**>  
_<!-- feign-hc5-->  
_<**dependency**>    <**groupId**>io.github.openfeign</**groupId**>    <**artifactId**>feign-hc5</**artifactId**>    <**version**>13.1</**version**>  
</**dependency**>
```

 Apache HttpClient 5 配置开启说明
 YAML修改
##### 替换之前
##### 替换之后
#### OpenFeign请求/响应压缩
###### 官网说明
![[无标题 3.png]]
###### 是什么
对请求和响应进行GZIP压缩

> Spring Cloud OpenFeign支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。
> 
> 通过下面的两个参数设置，就能开启请求与相应的压缩功能：
> 
> spring.cloud.openfeign.compression.request.enabled=true
> 
> spring.cloud.openfeign.compression.response.enabled=true

细粒度化设置

> 对请求压缩做一些更细致的设置，比如下面的配置内容指定压缩的请求数据类型并设置了请求压缩的大小下限，
> 
> 只有超过这个大小的请求才会进行压缩：
> 
> spring.cloud.openfeign.compression.request.enabled=true
> 
> spring.cloud.openfeign.compression.request.mime-types=text/xml,application/xml,application/json #触发压缩数据类型
> 
> spring.cloud.openfeign.compression.request.min-request-size=2048 #最小触发压缩的大小
###### YAML
###### 压缩效果测试在下一章节体验
#### OpenFeign日志打印功能
##### 日志打印功能
##### 是什么
> Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，
> 从而了解 Feign 中 Http 请求的细节，
> 说白了就是对Feign接口的调用情况进行监控和输出
##### 日志级别
> NONE：默认的，不显示任何日志；
> 
> BASIC：仅记录请求方法、URL、响应状态码及执行时间；
> 
> HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
> 
> FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。
![[无标题 4.png]]
##### 配置日志bean
###### 公式(三段)：**logging.level** + 含有@FeignClient注解的完整带包名的接口名+debug

```YAML
logging:  
  level:  
    com:  
      QingJiu:  
        cloud:  
          api:  
            PayFeignApi: debug
```

##### YAML文件里需要开启日志的Feign客户端
##### 后台日志查看
###### 带压缩调用
![[Pasted image 20240703201159.png]]
###### 去掉压缩调用
![[Pasted image 20240703201348.png]]

##### 补充，重试机制控制台看到3次过程
![[无标题 5.png]]

## OpenFeign个Sentinel集成实现fallback服务降级