## Sleuth目前也进入维护模式
![[Pasted image 20240706100931.png]]
## 分布式链路追踪概述
### 为什么会出现这个技术？需要解决哪些问题？
在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

### 随着问题的复杂化+微服务的增多+调用链条的变长画面不要太美丽
![[无标题 15.png]]

### 在分布式与微服务场景下需要解决的问题
#### 在分布式与微服务场景下，我们需要解决如下问题：
> 在大规模分布式与微服务集群下，如何实时观测系统的整体调用链路情况。
> 在大规模分布式与微服务集群下，如何快速发现并定位到问题。
> 在大规模分布式与微服务集群下，如何尽可能精确的判断故障对系统的影响范围与影响程度。
> 在大规模分布式与微服务集群下，如何尽可能精确的梳理出服务之间的依赖关系，并判断出服务之间的依赖关系是否合理。
> 在大规模分布式与微服务集群下，如何尽可能精确的分析整个系统调用链路的性能与瓶颈点。
> 在大规模分布式与微服务集群下，如何尽可能精确的分析系统的存储瓶颈与容量规划。

#### 上述问题就是我们的落地议题答案：
> 分布式链路追踪技术要解决的问题，分布式链路追踪（Distributed Tracing），就是将一次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示。比如各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。
## 新一代Spring Cloud Sleuth : Micrometer
### 官网和注意事项
官网地址：[Micrometer Documentation :: Micrometer](https://docs.micrometer.io/micrometer/reference/)
github地址：[micrometer-metrics/micrometer: An application observability facade for the most popular observability tools. Think SLF4J, but for observability. (github.com)](https://github.com/micrometer-metrics/micrometer?tab=readme-ov-file)
#### 说明
版本注意：使用springboot3以上开发必须要使用Micrometer；springboot3以下的项目可以使用Sleuth开发

### Zipkin
Spring Cloud Sleuth(Micrometer)提供了一套完整的分布式链路追踪解决方案并且兼容支持zipkin展示

### 小总结
将一次分布式请求还原成调用链路，进行日志记录和性能监控，并将一次分布式请求的调用情况集中web展示

### 行业内比较成熟的其他分布式链路追踪技术解决方案
![[无标题 16.png]]

## 分布式链路追踪原理
### 假定3个微服务调用的链路：
Service1调用Service2，Service2调用Service3和Service4
![[无标题 17.png]]
### 上一步完整的调用链路
那么一条链路追踪会在每个服务调用的时候加上Trace ID 和 Span ID链路通过TraceId唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来 (Span:表示调用链路来源，通俗的理解span就是一次请求信息)
![[无标题 18.png]]
### 彻底把链路追踪整明白
一条链路通过Trace Id唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来
![[无标题 19.png]]
![[Pasted image 20240706110811.png]]
## Zipkin
### 官网
[OpenZipkin 的 ·分布式跟踪系统](https://zipkin.io/)
### 是什么
#### ZipKin概述
> Zipkin是一种分布式链路跟踪系统图形化的工具，Zipkin 是 Twitter 开源的分布式跟踪系统，能够收集微服务运行过程中的实时调用链路信息，并能够将这些调用链路信息展示到Web图形化界面上供开发人员分析，开发人员能够从ZipKin中分析出调用链路中的性能瓶颈，识别出存在问题的应用程序，进而定位问题和解决问题。
### Zipkin为什么出现
![[无标题 20.png]]
说明：
当没有配置 Sleuth 链路追踪的时候，INFO 信息里面是 [passjava-question,,,]，后面跟着三个空字符串。
当配置了 Sleuth 链路追踪的时候，追踪到的信息是 [passjava-question,504a5360ca906016,e55ff064b3941956,false] ，第一个是 Trace ID，第二个是 Span ID。只有日志没有图，观看不方便，不美观，so，引入图形化Zipkin链路监控让你好看，O(∩_∩)O
### 下载+安装+运行
## Micrometer+ZipKin搭建链路监控案例步骤
### 步骤
#### 总体父工程POM
##### 本案例
###### 引入的jar包分别是什么意思
由于Micrometer Tracing是一个门面工具自身并没有实现完整的链路追踪系统，具体的链路追踪另外需要引入的是第三方链路追踪系统的依赖：

![[Pasted image 20240706144213.png]]
补充包：spring-boot-starter-actuator   SpringBoot框架的一个模块用于监视和管理应用程序
##### POM

```XML
<dependencies>  
    <!--micrometer-tracing-bom导入链路追踪版本中心  1-->    <dependency>  
        <groupId>io.micrometer</groupId>  
        <artifactId>micrometer-tracing-bom</artifactId>  
        <version>${micrometer-tracing.version}</version>  
        <type>pom</type>  
        <scope>import</scope>  
    </dependency>    <!--micrometer-tracing指标追踪  2-->    <dependency>  
        <groupId>io.micrometer</groupId>  
        <artifactId>micrometer-tracing</artifactId>  
        <version>${micrometer-tracing.version}</version>  
    </dependency>    <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 3-->    <dependency>  
        <groupId>io.micrometer</groupId>  
        <artifactId>micrometer-tracing-bridge-brave</artifactId>  
        <version>${micrometer-tracing.version}</version>  
    </dependency>    <!--micrometer-observation 4-->  
    <dependency>  
        <groupId>io.micrometer</groupId>  
        <artifactId>micrometer-observation</artifactId>  
        <version>${micrometer-observation.version}</version>  
    </dependency>    <!--feign-micrometer 5-->  
    <dependency>  
        <groupId>io.github.openfeign</groupId>  
        <artifactId>feign-micrometer</artifactId>  
        <version>${feign-micrometer.version}</version>  
    </dependency>    <!--zipkin-reporter-brave 6-->  
    <dependency>  
        <groupId>io.zipkin.reporter2</groupId>  
        <artifactId>zipkin-reporter-brave</artifactId>  
        <version>${zipkin-reporter-brave.version}</version>  
    </dependency>
```


#### 服务提供者8001
##### POM

```xml
<!--micrometer-tracing指标追踪  1--><dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-tracing</artifactId>  
</dependency>  
<!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2--><dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-tracing-bridge-brave</artifactId>  
</dependency>  
<!--micrometer-observation 3-->  
<dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-observation</artifactId>  
</dependency>  
<!--feign-micrometer 4-->  
<dependency>  
    <groupId>io.github.openfeign</groupId>  
    <artifactId>feign-micrometer</artifactId>  
</dependency>  
<!--zipkin-reporter-brave 5-->  
<dependency>  
    <groupId>io.zipkin.reporter2</groupId>  
    <artifactId>zipkin-reporter-brave</artifactId>  
</dependency>
```

##### YAML

```yaml
# ========================zipkin===================  
management:  
  zipkin:  
    tracing:  
      endpoint: http://localhost:9411/api/v2/spans  
  tracing:  
    sampling:  
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

##### 新建业务类

```java
@RequestMapping("pay")  
@RestController  
public class PayMicrometerController {  
  
    @GetMapping("/micrometer/{id}")  
    public String myMicrometer(@PathVariable("id") Integer id){  
        return "Hello,欢迎来到micrometer inputId：" + id + "\t 服务返回" + IdUtil.simpleUUID();  
    }  
  
}
```

#### API接口PayFeignApi

```java
@GetMapping("/pay/micrometer/{id}")  
public String myMicrometer(@PathVariable("id") Integer id);
```


#### 服务调用者80
##### POM

```xml
<!--micrometer-tracing指标追踪  1--><dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-tracing</artifactId>  
</dependency>  
<!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2--><dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-tracing-bridge-brave</artifactId>  
</dependency>  
<!--micrometer-observation 3-->  
<dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-observation</artifactId>  
</dependency>  
<!--feign-micrometer 4-->  
<dependency>  
    <groupId>io.github.openfeign</groupId>  
    <artifactId>feign-micrometer</artifactId>  
</dependency>  
<!--zipkin-reporter-brave 5-->  
<dependency>  
    <groupId>io.zipkin.reporter2</groupId>  
    <artifactId>zipkin-reporter-brave</artifactId>  
</dependency>
```

##### YAML

```yaml
# zipkin图形展现地址和采样率设置  
management:  
  zipkin:  
    tracing:  
      endpoint: http://localhost:9411/api/v2/spans  
  tracing:  
    sampling:  
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

##### 新建业务类

```java
@RequestMapping("feign")  
@Slf4j  
@RestController  
public class OrderMicrometerController {  
  
    @Resource  
    private PayFeignApi payFeignApi;  
  
    @GetMapping("/pay/micrometer/{id}")  
    public String myMicrometer(@PathVariable("id") Integer id){  
        return payFeignApi.myMicrometer(id);  
    }  
  
}
```
