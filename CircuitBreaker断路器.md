## Hystrix目前也进入维护模式
### 是什么
Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。
### Hystrix官宣，停更进维
![[无标题 6.png]]
### Hystrix未来替换方案
Resilience4J
![[无标题-1.png]]
## 概述
### 分布式系统面临的问题
#### 分布式系统面临的问题
> 复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。
 
#### 服务雪崩
>  多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.
> 
> 对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。
> 
> 所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

### 我们的诉求
#### 问题：
> 禁止服务雪崩故障

#### 解决： 
> - 有问题的节点，快速熔断（快速返回失败处理或者返回默认兜底数据【服务降级】）。
> 
> “断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

一句话，**出故障了“保险丝”跳闸，别把整个家给烧了**。
### 如何搞定上述问题避免整个系统大面积故障
#### 搞定
##### 服务熔断
> 类比保险丝，保险丝闭合状态(CLOSE)可以正常使用，当达到最大服务访问后，直接拒绝访问跳闸限电(OREN)，此刻调用方会接受服务降级的处理并返回友好兜底提示。
> 就是家里保险丝，从闭合CLOSE供电状态->跳闸OPEN打开状态。
##### 服务降级
> 服务器忙，请稍后再试。
> 不让客户端等待并立刻返回一个友好提示。
##### 服务限流
> 秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟n个，有序进行。
##### 服务限时
##### 服务预热
##### 接近实时的监控
##### 兜底的处理动作

## Circuit Breaker是什么
### 官网
[Spring Cloud 断路器](https://spring.io/projects/spring-cloud-circuitbreaker)
![[Pasted image 20240703231042.png]]
### 实现原理
CircuitBreaker的目的是保护分布式系统免受故障和异常，提高系统的可用性和健壮性。

当一个组件或服务出现故障时，CircuitBreaker会迅速切换到开放OPEN状态(保险丝跳闸断电)，阻止请求发送到该组件或服务从而避免更多的请求发送到该组件或服务。这可以减少对该组件或服务的负载，防止该组件或服务进一步崩溃，并使整个系统能够继续正常运行。同时，CircuitBreaker还可以提高系统的可用性和健壮性，因为它可以在分布式系统的各个组件之间自动切换，从而避免单点故障的问题。
![[无标题 7.png]]
## Resilience4J
### 是什么
地址：[resilience4j/resilience4j：Resilience4j 是一个专为 Java8 和函数式编程设计的容错库 (github.com)](https://github.com/resilience4j/resilience4j)
![[Pasted image 20240703231444.png]]
### 能干吗？
官网说明：[Examples (readme.io)](https://resilience4j.readme.io/docs/examples)
![[Pasted image 20240703232031.png]]
### 怎么玩？
中文手册介绍：[Resilience4j-Guides-Chinese/index.md at main · lmhmhl/Resilience4j-Guides-Chinese (github.com)](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/index.md)

## 案例实战
### 熔断（CircuitBreaker）（服务熔断+服务降级）
#### 断路器3大状态
![[Pasted image 20240704170705.png]]
#### 断路器3大状态之间的转换
![[无标题 8.png]]
#### 断路器所有配置参数参考
官网说明[CircuitBreaker (readme.io)](https://resilience4j.readme.io/docs/circuitbreaker)
中文手册[Resilience4j-Guides-Chinese/core-modules/CircuitBreaker.md at main · lmhmhl/Resilience4j-Guides-Chinese (github.com)](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/CircuitBreaker.md#%E5%88%9B%E5%BB%BA%E5%92%8C%E9%85%8D%E7%BD%AEcircuitbreaker)

#### 熔断+降级案例说明
>  6__次访问中当执行方法的失败率达到__50%__时__CircuitBreaker__将进入开启__OPEN__状态__(__保险丝跳闸断电__)__拒绝所有请求。  
>   等待__5__秒后，__CircuitBreaker_ _将自动从开启__OPEN__状态过渡到半开__HALF_OPEN__状态，允许一些请求通过以测试服务是否恢复正常。  
>   如还是异常__CircuitBreaker_ _将重新进入开启__OPEN__状态；如正常将进入关闭__CLOSE__闭合状态恢复正常处理请求。_

具体时间和频次等属性见具体实际案例，这里只是作为case举例讲解
![[无标题 9.png]]
#### 按照COUNT_BASED(计数的滑动窗口)
##### 修改8001
###### 新建PayCircuitController

```java
@RequestMapping("pay")  
@RestController  
public class PayCircuitController {  
	//=========Resilience4j CircuitBreaker_ _的例子_
    @GetMapping("/circuit/{id}")  
    public String myCircuit(@PathVariable("id") Integer id){  
        if (id < 0)  
            throw new RuntimeException("-------circuit id 不能为负数");  
  
        if (id == 9999){  
            try {  
                TimeUnit.SECONDS.sleep(5);  
            }catch (Exception e){  
                e.printStackTrace();  
            }  
        }  
        return "Hello, circuit! inputId " + id + "\t" + IdUtil.simpleUUID();  
    }  
  
}
```

##### 修改PayFeignApi接口
###### 添加Resilience4j CircuitBreaker_
```java
//Resilience4j CircuitBreaker_ _的例子_
@GetMapping("/pay/circuit/{id}")  
public String myCircuit(@PathVariable("id") Integer id);
```
##### 修改80
###### 改POM
```xml
_<!--resilience4j-circuitbreaker-->  
_<**dependency**>    <**groupId**>org.springframework.cloud</**groupId**>    <**artifactId**>spring-cloud-starter-circuitbreaker-resilience4j</**artifactId**>  
</**dependency**>  
_<!--_ _由于断路保护等需要__AOP__实现，所以必须导入__AOP__包_ _-->  
_<**dependency**>    <**groupId**>org.springframework.boot</**groupId**>    <**artifactId**>spring-boot-starter-aop</**artifactId**>  
</**dependency**>
```
###### 写YAML

```YAML
spring:   
  cloud:  
    openfeign:  
    # 开启circuitbreaker和分组激活spring.cloud.openfeign.circuitbreaker.enabled
      circuitbreaker:  
		  enabled: true  
		  group:  
		    enabled: true

# Resilience4j CircuitBreaker 按照次数：COUNT_BASED 的例子  
#  6次访问中当执行方法的失败率达到50%时CircuitBreaker将进入开启OPEN状态(保险丝跳闸断电)拒绝所有请求。  
#  等待5秒后，CircuitBreaker 将自动从开启OPEN状态过渡到半开HALF_OPEN状态，允许一些请求通过以测试服务是否恢复正常。  
#  如还是异常CircuitBreaker 将重新进入开启OPEN状态；如正常将进入关闭CLOSE闭合状态恢复正常处理请求。  
resilience4j:  
  circuitbreaker:  
    configs:  
      default:  
        failureRateThreshold: 50 #设置50%的调用失败时打开断路器，超过失败请求百分⽐CircuitBreaker变为OPEN状态。  
        slidingWindowType: COUNT_BASED # 滑动窗口的类型 按计数器来统计  
        slidingWindowSize: 6 #滑动窗⼝的⼤⼩配置COUNT_BASED表示6个请求，配置TIME_BASED表示6秒  
        minimumNumberOfCalls: 6 #断路器计算失败率或慢调用率之前所需的最小样本(每个滑动窗口周期)。如果minimumNumberOfCalls为10，则必须最少记录10个样本，然后才能计算失败率。如果只记录了9次调用，即使所有9次调用都失败，断路器也不会开启。  
        automaticTransitionFromOpenToHalfOpenEnabled: true # 是否启用自动从开启状态过渡到半开状态，默认值为true。如果启用，CircuitBreaker将自动从开启状态过渡到半开状态，并允许一些请求通过以测试服务是否恢复正常  
        waitDurationInOpenState: 5s #从OPEN到HALF_OPEN状态需要等待的时间  
        permittedNumberOfCallsInHalfOpenState: 2 #半开状态允许的最大请求数，默认值为10。在半开状态下，CircuitBreaker将允许最多permittedNumberOfCallsInHalfOpenState个请求通过，如果其中有任何一个请求失败，CircuitBreaker将重新进入开启状态。  
        recordExceptions:  
          - java.lang.Exception  
    instances:  
      cloud-payment-service:  
        baseConfig: default
```

###### 新建OrderCircuitController

```java
@RequestMapping("feign")  
@RestController  
public class OrderCircuitController {  
  
    @Resource  
    private PayFeignApi payFeignApi;  
  
    @GetMapping("/pay/circuit/{id}")  
    @CircuitBreaker(name = "cloud-payment-service",fallbackMethod = "myCircuitFallback")//保护哪个微服务,若服务不可用了要有兜底的方法  
    public String myCircuit(@PathVariable("id") Integer id){  
  
        return payFeignApi.myCircuit(id);  
  
    }  
  
    //myCircuitFallback就是服务降级的兜底方法  
    public String myCircuitFallback(Integer id , Throwable t){  
        //这里是容错处理逻辑，返回备用结果  
        return "myCircuitFallback 系统繁忙，请稍后重试----(っ °Д °;)っ(っ °Д °;)っ";  
    }  
  
}
```

##### 测试
#### 按照TIME_BASED(时间的滑动窗口)
![[无标题 10.png]]
##### 基于时间的滑动窗口

##### 修改80的YAML
##### 为避免测试效果关闭，重试3次
##### 测试（慢查询）
#### 小总结
##### 断路器开启或者关闭的条件
> 当满足一定的峰值和失败率达到一定条件后，断路器将会进入OPEN状态（保险丝跳闸），服务熔断；

> 当OPEN的时候，所有请求都不会调用主页逻辑方法，而是直接走fallbackmetnod兜底被噶方法，服务降级；

> 一段时间只会，这个时候断路器会从OPEN进入到HALF_oPEN半开状态，会放几个请求过去谈谈链路是否通？
> 如果成功，断路器会关闭CLOSE（类似保险丝闭合，恢复可用）；
> 如果失败，继续开启。重复上述过程。
##### 个人建议不要混合用，推荐按照调用次数count_based
### 隔离（BulkHead）
#### 官网
[Resilience4j-Guides-Chinese/core-modules/bulkhead.md at main · lmhmhl/Resilience4j-Guides-Chinese (github.com)](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/bulkhead.md)
#### 是什么
限并发

#### 能干吗
##### 依赖隔离&负载保护
用来限制对于下游服务的最大并发数量的限制

#### Resilience4j提供了两种隔离的是方法，可以限制并发执行的数量
![[Pasted image 20240704230522.png]]
#### 实现信号量舱壁：SemaphoreBulkhead
##### 概述
###### 信号量舱壁（SemaphoreBulkhead）原理
> 当信号量有空闲时，进入系统的请求会直接获取信号量并开始业务处理。
> 当信号量全被占用时，接下来的请求将会进入阻塞状态，SemaphoreBulkhead提供了一个阻塞计时器，
> 如果阻塞状态的请求在阻塞计时内无法获取到信号量则系统会拒绝这些请求。
> 若请求在阻塞计时内获取到了信号量，那将直接获取信号量并执行相应的业务处理。
##### 8001支付微服务修改PayCircuitController

```java
//=========Resilience4j bulkhead 的例子  
@GetMapping(value = "/bulkhead/{id}")  
public String myBulkhead(@PathVariable("id") Integer id)  
{  
    if(id == -4) throw new RuntimeException("----bulkhead id 不能-4");  
  
    if(id == 9999)  
    {  
        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }  
    }  
  
    return "Hello, bulkhead! inputId:  "+id+" \t " + IdUtil.simpleUUID();  
}java
```

##### PayFeignApi接口新增舱壁api方法

```java
@GetMapping(value = "/pay/bulkhead/{id}")  
public String myBulkhead(@PathVariable("id") Integer id);
```

##### 修改80
###### POM

```xml
_<!--resilience4j-bulkhead-->  
_<**dependency**>    <**groupId**>io.github.resilience4j</**groupId**>    <**artifactId**>resilience4j-bulkhead</**artifactId**>  
</**dependency**>
```

###### YAML

```YAML
resilience4j:  
  bulkhead:  
    configs:  
      default:  
        maxConcurrentCalls: 2 # 隔离允许并发线程执行的最大数量  
        maxWaitDuration: 1s # 当达到并发调用数量时，新的线程的阻塞时间，我只愿意等待1秒，过时不候进舱壁兜底fallback  
    instances:  
      cloud-payment-service:  
        baseConfig: default  
  timelimiter:  
    configs:  
      default:  
        timeout-duration: 20s
```

###### 业务类

```java
@RequestMapping("feign")  
@RestController  
public class OrderCircuitController {  
  
    @Resource  
    private PayFeignApi payFeignApi;  
  
    @GetMapping(value = "/pay/bulkhead/{id}")  
    @Bulkhead(name = "cloud-payment-service",fallbackMethod = "myBulkheadFallback",type = Bulkhead.Type.SEMAPHORE)//指定信号量  
    public String myBulkhead(@PathVariable("id") Integer id){  
        return payFeignApi.myBulkhead(id);  
    }  
  
    public String myBulkheadFallback(Throwable t)  
    {  
        return "myBulkheadFallback，隔板超出最大数量限制，系统繁忙，请稍后再试-----o(￣┰￣*)ゞ";  
    }  
  
  
  
}
```

##### 测试
#### 实现固定线程池舱壁：FixedThreadPoolBulkhead
##### 概述
固定线程池舱壁（FixedThreadPoolBulkhead）
> FixedThreadPoolBulkhead的功能与SemaphoreBulkhead一样也是**用于限制并发执行的次数**的，但是二者的实现原理存在差别而且表现效果也存在细微的差别。FixedThreadPoolBulkhead使用一个固定线程池和一个等待队列来实现舱壁。
> 
> 当线程池中存在空闲时，则此时进入系统的请求将直接进入线程池开启新线程或使用空闲线程来处理请求。
> 
> 当线程池中无空闲时时，接下来的请求将进入等待队列，
> 
>    若等待队列仍然无剩余空间时接下来的请求将直接被拒绝，
> 
>    在队列中的请求等待线程池出现空闲时，将进入线程池进行业务处理。
> 
> 另外：ThreadPoolBulkhead只对CompletableFuture方法有效，所以我们必创建返回CompletableFuture类型的方法
##### 修改80
###### POM
###### YAML
![[Pasted image 20240705193012.png]]
###### controller

```java
/**  
 * 舱壁隔离 固定线程池壁垒  
 * @param id  
 * @return  
 */  
@GetMapping(value = "/pay/bulkhead/{id}")  
@Bulkhead(name = "cloud-payment-service",fallbackMethod = "myBulkheadPoolFallback",type = Bulkhead.Type.THREADPOOL)  
public CompletableFuture<String> myBulkheadTHREADPOOL(@PathVariable("id") Integer id){  
  
    System.out.println(Thread.currentThread().getName()+"\t"+"----开始进入ヾ(≧▽≦*)o----");  
    try {  
        TimeUnit.SECONDS.sleep(3);  
    }catch (InterruptedException e){  
        e.printStackTrace();  
    }  
    System.out.println(Thread.currentThread().getName()+"\t"+"----准备离开ヾ(≧▽≦*)o----");  
  
    return CompletableFuture.supplyAsync(() -> payFeignApi.myBulkhead(id) + "\t" + " Bulkhead.Type.THREADPOOL ");  
}  
  
public CompletableFuture<String> myBulkheadPoolFallback(Integer id ,Throwable t) {  
  
    return CompletableFuture.supplyAsync(() -> "myBulkheadPoolFallback，隔板超出最大数量限制，系统繁忙，请稍后再试-----o(￣┰￣*)ゞ" + "Bulkhead.Type.THREADPOOL");  
  
}
```

##### 测试


### 限流(RateLimiter)
#### 官网
官方[RateLimiter (readme.io)](https://resilience4j.readme.io/docs/ratelimiter)
中文[Resilience4j-Guides-Chinese/core-modules/ratelimiter.md at main · lmhmhl/Resilience4j-Guides-Chinese (github.com)](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/ratelimiter.md)
#### 是什么
##### 限流（频率控制）
> 限流 就是限制最大访问流量。系统能提供的最大并发是有限的，同时来的请求又太多，就需要限流。 
> 
> 比如商城秒杀业务，瞬时大量请求涌入，服务器忙不过就只好排队限流了，和去景点排队买票和去医院办理业务排队等号道理相同。
> 
> 所谓限流，就是通过对并发访问/请求进行限速，或者对一个时间窗口内的请求进行限速，以保护应用系统，一旦达到限制速率则可以拒绝服务、排队或等待、降级等处理。

#### 创建限流算法
##### 漏斗算法
> 一个固定容量的漏桶，按照设定常量固定速率流出水滴，类似医院打吊针，不管你源头流量多大，我设定匀速流出。 
> 如果流入水滴超出了桶的容量，则流入的水滴将会溢出了(被丢弃)，而漏桶容量是不变的。
![[无标题 11.png]]
###### 缺点
> 这里有两个变量，一个是桶的大小，支持流量突发增多时可以存多少的水（burst），另一个是水桶漏洞的大小（rate）。因为漏桶的漏出速率是固定的参数，所以，即使网络中不存在资源冲突（没有发生拥塞），漏桶算法也不能使流突发（burst）到端口速率。因此，漏桶算法对于存在突发特性的流量来说缺乏效率。
##### 令牌桶算法
springcloud默认算法
![[无标题 12.png]]
##### 滚动时间窗
> 允许固定数量的请求进入(比如1秒取4个数据相加，超过25值就over)超过数量就拒绝或者排队，等下一个时间段进入。
> 
> 由于是在一个时间间隔内进行限制，如果用户在上个时间间隔结束前请求（但没有超过限制），同时在当前时间间隔刚开始请求（同样没超过限制），在各自的时间间隔内，这些请求都是正常的。下图统计了3次，but......
![[无标题 13.png]]

###### 缺点：
间隔临界的一段时间内的请求就会超过系统限制，可能导致系统被压垮；
由于计数器算法存在时间临界点缺陷，因此在时间临界点左右的极短时间段内容易遭到攻
![[无标题 14.png]]
假如设定1分钟最多可以请求100次某个接口，如12:00:00-12:00:59时间段内没有数据请求但12:00:59-12:01:00时间段内突然并发100次请求，紧接着瞬间跨入下一个计数周期计数器清零；在12:01:00-12:01:01内又有100次请求。那么也就是说在时间临界点左右可能同时有2倍的峰值进行请求，从而造成后台处理请求**加倍过载**的bug，导致系统运营能力不足，甚至导致系统崩溃，/(ㄒoㄒ)/~~

##### 滑动时间窗
> 顾名思义，该时间窗口是滑动的。所以，从概念上讲，这里有两个方面的概念需要理解： 
> 
> - 窗口：需要定义窗口的大小
> 
> - 滑动：需要定义在窗口中滑动的大小，但理论上讲滑动的大小不能超过窗口大小
> 
> 滑动窗口算法是把固定时间片进行划分并且随着时间移动，移动方式为开始时间点变为时间列表中的第2个时间点，结束时间点增加一个时间点，
> 
> 不断重复，通过这种方式可以巧妙的避开计数器的临界点的问题。下图统计了5次
![[无标题 1 2.png]]
#### 8001支付微服务修改PayCircuitController新增myRatelimit方法

```java
@GetMapping("/ratelimit/{id}")  
public String myRatelimit(@PathVariable("id") Integer id){  
    return "Hello,myRatelimit欢迎来到 inputId" + id + "\t" + IdUtil.simpleUUID();  
}
```

#### PayFeignApi接口新增限流api方法

```java
@GetMapping("/pay/ratelimit/{id}")  
public String myRatelimit(@PathVariable("id") Integer id);
```

#### 修改80
##### POM

```xml
<!--resilience4j-ratelimiter-->  
<dependency>  
    <groupId>io.github.resilience4j</groupId>  
    <artifactId>resilience4j-ratelimiter</artifactId>  
</dependency>
```
##### YAML

```YAML
####resilience4j ratelimiter 限流的例子  
resilience4j:  
  ratelimiter:  
    configs:  
      default:  
        limitForPeriod: 2 #在一次刷新周期内，允许执行的最大请求数  
        limitRefreshPeriod: 1s # 限流器每隔limitRefreshPeriod刷新一次，将允许处理的最大请求数量重置为limitForPeriod  
        timeout-duration: 1 # 线程等待权限的默认等待时间  
    instances:  
      cloud-payment-service:  
        baseConfig: default
```

##### controller

```java
@GetMapping("/pay/ratelimit/{id}")  
@RateLimiter(name = "cloud-payment-service",fallbackMethod = "myRatelimitFallback")  
public String myRatelimit(@PathVariable("id") Integer id){  
    return payFeignApi.myRatelimit(id);  
}  
public String myRatelimitFallback(Integer id,Throwable t){  
    return "你被限流了，禁止访问╮(╯-╰)╭";  
}
```
