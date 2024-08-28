## Ribbon目前也进入维护模式
##### 是什么
> Spring Cloud Ribbon是基于Netflix Ribbon实现的一套_客户端_       负载均衡的工具。
> 
> 简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。
##### Ribbon未来替换方案
![[Ribbon替换方案.png]]
## spring-cloud-loadbalancer概述
##### 官网
[Spring Cloud Commons](https://spring.io/projects/spring-cloud-commons#learn)
##### 是什么
官网说明：[Spring Cloud LoadBalancer ：： Spring Cloud Commons](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html)

###### LB负载均衡(Load Balance)是什么
> 简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用），常见的负载均衡有软件Nginx，LVS，硬件 F5等

###### spring-cloud-starter-loadbalancer组件是什么
> Spring Cloud LoadBalancer是由SpringCloud官方提供的一个开源的、简单易用的**客户端负载均衡器**，它包含在SpringCloud-commons中用它来替换了以前的Ribbon组件。相比较于Ribbon，SpringCloud LoadBalancer不仅能够支持RestTemplate，还支持WebClient（WeClient是Spring Web Flux中提供的功能，可以实现响应式异步请求）

##### 面试题
###### loadbalancer本地负载均衡客户端 VS Nginx服务端负载均衡区别
> Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求，即负载均衡是由服务端实现的。
> 
> loadbalancer本地负载均衡，在调用微服务接口时候，会在注册中心（Consul）上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

## spring-cloud-loadbalancer负载均衡解析
##### 负载均衡演示案例-理论
架构说明：80通过轮询负载访问8001/8002/8003
![[LoadBalancer 通过Counsul轮询其他服务提供者.png]]
###### LoadBalancer 在工作时分成两步：

> **第一步**，先选择ConsulServer从服务端查询并拉取服务列表，知道了它有多个服务(上图3个服务)，这3个实现是完全一样的，
> 
> 默认轮询调用谁都可以正常执行。类似生活中求医挂号，某个科室今日出诊的全部医生，客户端你自己选一个。
> 
> **第二步**，按照指定的负载均衡策略从server取到的服务注册列表中由客户端自己选择一个地址，所以LoadBalancer是一个**客户端的**负载均衡器。

##### 负载均衡演示案例-实操
###### 官网参考如何正确使用？
介绍：[Spring Cloud LoadBalancer :: Spring Cloud Commons](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html)只要是抽象的，一定可以有多种实现方法。

###### 官网对使用的演示：

```java
@Configuration public class MyConfiguration { 

	@LoadBalanced 
	@Bean 
	RestTemplate restTemplate() { 
		return new RestTemplate(); 
	} 
}

public class MyClass { 
@Autowired 
private RestTemplate restTemplate; 

public String doOtherStuff() { 
	String result = restTemplate.getForObject("http://stores/stores",String.class); 
	return result; 
	} 
}
```

##### 按照8001拷贝新建8002微服务
##### 启动Consul，将8001/8002启动后注册进微服务
###### 启动后注册进微服务
###### bug
我们之前的配置key/value完全消失没有持久化保存；

Consul数据持久化配置并且注册为Windows服务
> 1、新建文件夹：D:\consul\consul_1.17.1_windows_386
> 	创建文件夹
> 	新建文件consul_start.bat后缀为.bat
##### 订单80模块修改POM并注册进counsul，新增LoadBalancer组件
![[订单80模块POM新增LoadBalancer组件.png]]

```xml
_<!--loadbalancer-->  
_<**dependency**>    <**groupId**>org.springframework.cloud</**groupId**>    <**artifactId**>spring-cloud-starter-loadbalancer</**artifactId**>  
</**dependency**>
```

##### 订单80模块修改Controller并启动80
##### 目前consul上的服务
##### 测试


##### 负载均衡演示案例-小总结
###### 编码使用DiscoveryClient动态获取所有线上的服务列表
官方介绍和网址：[Service Discovery with Consul ：： Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/discovery.html#using-the-discoveryclient)
![[使用DiscoveryClient.png]]
###### 代码解释，修改80微服务的Controller

```java
@Resource  
private DiscoveryClient discoveryClient; //发现客户端  
@GetMapping("/discovery")  
public String discovery()  
{  
    List<String> services = discoveryClient.getServices(); //将服务器上的Service取出来  
    for (String element : services) {  
        System.out.println(element);  
    }  
  
    System.out.println("===================================");  
  
    //找为cloud-payment-service的service  
    List<ServiceInstance> instances = discoveryClient.getInstances("cloud-payment-service");  
    for (ServiceInstance element : instances) {  
        System.out.println(element.getServiceId()+"\t"+element.getHost()+"\t"+element.getPort()+"\t"+element.getUri());  
    }  
  
    //返回第0个的service的id 和 端口号  
    return instances.get(0).getServiceId()+":"+instances.get(0).getPort();  
}
```
结果：

```
cloud-consumer-order
cloud-payment-service
consul
===================================
cloud-payment-service	localhost	8001	http://localhost:8001
cloud-payment-service	localhost	8002	http://localhost:8002
```

###### 负载选择原理小结
> 负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。

```java
List<ServiceInstance> instances = discoveryClient.getInstances("cloud-payment-service");
```

> 8001+ 8002 组合成为集群，它们共计2台机器，集群总数为2， 按照轮询算法原理：
> 当总请求数为1时： 1 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
> 当总请求数位2时： 2 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002
> 
> 当总请求数位3时： 3 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
> 
> 当总请求数位4时： 4 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002

## 负载均衡算法原理
##### 默认算法是什么？有几种？
###### 官网地址：[Spring Cloud LoadBalancer ：： Spring Cloud Commons](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html)
![[负载均衡算法的切换.png]]
###### 默认2种
###### 轮询源码

```java
//  
// Source code recreated from a .class file by IntelliJ IDEA  
// (powered by FernFlower decompiler)  
//  
  
package org.springframework.cloud.loadbalancer.core;  
  
import java.util.List;  
import java.util.Random;  
import java.util.concurrent.atomic.AtomicInteger;  
import org.apache.commons.logging.Log;  
import org.apache.commons.logging.LogFactory;  
import org.springframework.beans.factory.ObjectProvider;  
import org.springframework.cloud.client.ServiceInstance;  
import org.springframework.cloud.client.loadbalancer.DefaultResponse;  
import org.springframework.cloud.client.loadbalancer.EmptyResponse;  
import org.springframework.cloud.client.loadbalancer.Request;  
import org.springframework.cloud.client.loadbalancer.Response;  
import reactor.core.publisher.Mono;  
  
public class RoundRobinLoadBalancer implements ReactorServiceInstanceLoadBalancer {  
    private static final Log log = LogFactory.getLog(RoundRobinLoadBalancer.class);  
    final AtomicInteger position;  
    final String serviceId;  
    ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;  
  
    public RoundRobinLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId) {  
        this(serviceInstanceListSupplierProvider, serviceId, (new Random()).nextInt(1000));  
    }  
  
    public RoundRobinLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId, int seedPosition) {  
        this.serviceId = serviceId;  
        this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;  
        this.position = new AtomicInteger(seedPosition);  
    }  
  
    public Mono<Response<ServiceInstance>> choose(Request request) {  
        ServiceInstanceListSupplier supplier = (ServiceInstanceListSupplier)this.serviceInstanceListSupplierProvider.getIfAvailable(NoopServiceInstanceListSupplier::new);  
        return supplier.get(request).next().map((serviceInstances) -> {  
            return this.processInstanceResponse(supplier, serviceInstances);  
        });  
    }  
  
    private Response<ServiceInstance> processInstanceResponse(ServiceInstanceListSupplier supplier, List<ServiceInstance> serviceInstances) {  
        Response<ServiceInstance> serviceInstanceResponse = this.getInstanceResponse(serviceInstances);  
        if (supplier instanceof SelectedInstanceCallback && serviceInstanceResponse.hasServer()) {  
            ((SelectedInstanceCallback)supplier).selectedServiceInstance((ServiceInstance)serviceInstanceResponse.getServer());  
        }  
  
        return serviceInstanceResponse;  
    }  
  
    private Response<ServiceInstance> getInstanceResponse(List<ServiceInstance> instances) {  
        if (instances.isEmpty()) {  
            if (log.isWarnEnabled()) {  
                log.warn("No servers available for service: " + this.serviceId);  
            }  
  
            return new EmptyResponse();  
        } else if (instances.size() == 1) {  
            return new DefaultResponse((ServiceInstance)instances.get(0));  
        } else {  
            int pos = this.position.incrementAndGet() & Integer.MAX_VALUE;  
            ServiceInstance instance = (ServiceInstance)instances.get(pos % instances.size());  
            return new DefaultResponse(instance);  
        }  
    }  
}
```
###### 随机源码

```java
//  
// Source code recreated from a .class file by IntelliJ IDEA  
// (powered by FernFlower decompiler)  
//  
  
package org.springframework.cloud.loadbalancer.core;  
  
import java.util.List;  
import java.util.concurrent.ThreadLocalRandom;  
import org.apache.commons.logging.Log;  
import org.apache.commons.logging.LogFactory;  
import org.springframework.beans.factory.ObjectProvider;  
import org.springframework.cloud.client.ServiceInstance;  
import org.springframework.cloud.client.loadbalancer.DefaultResponse;  
import org.springframework.cloud.client.loadbalancer.EmptyResponse;  
import org.springframework.cloud.client.loadbalancer.Request;  
import org.springframework.cloud.client.loadbalancer.Response;  
import reactor.core.publisher.Mono;  
  
public class RandomLoadBalancer implements ReactorServiceInstanceLoadBalancer {  
    private static final Log log = LogFactory.getLog(RandomLoadBalancer.class);  
    private final String serviceId;  
    private ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;  
  
    public RandomLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId) {  
        this.serviceId = serviceId;  
        this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;  
    }  
  
    public Mono<Response<ServiceInstance>> choose(Request request) {  
        ServiceInstanceListSupplier supplier = (ServiceInstanceListSupplier)this.serviceInstanceListSupplierProvider.getIfAvailable(NoopServiceInstanceListSupplier::new);  
        return supplier.get(request).next().map((serviceInstances) -> {  
            return this.processInstanceResponse(supplier, serviceInstances);  
        });  
    }  
  
    private Response<ServiceInstance> processInstanceResponse(ServiceInstanceListSupplier supplier, List<ServiceInstance> serviceInstances) {  
        Response<ServiceInstance> serviceInstanceResponse = this.getInstanceResponse(serviceInstances);  
        if (supplier instanceof SelectedInstanceCallback && serviceInstanceResponse.hasServer()) {  
            ((SelectedInstanceCallback)supplier).selectedServiceInstance((ServiceInstance)serviceInstanceResponse.getServer());  
        }  
  
        return serviceInstanceResponse;  
    }  
  
    private Response<ServiceInstance> getInstanceResponse(List<ServiceInstance> instances) {  
        if (instances.isEmpty()) {  
            if (log.isWarnEnabled()) {  
                log.warn("No servers available for service: " + this.serviceId);  
            }  
  
            return new EmptyResponse();  
        } else {  
            int index = ThreadLocalRandom.current().nextInt(instances.size());  
            ServiceInstance instance = (ServiceInstance)instances.get(index);  
            return new DefaultResponse(instance);  
        }  
    }  
}
```

###### 切换的重点是接口ReactorServiceInstanceLoadBalancer

##### 算法切换
###### 从默认的轮询，切换为随机算法，修改RestTemplateConfig
##### 测试