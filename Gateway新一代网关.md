## 概述
### 是什么
#### 官网
[Spring Cloud 网关](https://spring.io/projects/spring-cloud-gateway#overview)
#### 体系定位
Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关；
但在2.x版本中，zuul的升级一直跳票，SpringCloud最后自己研发了一个网关SpringCloud Gateway替代Zuul，
那就是SpringCloud Gateway一句话：gateway是原zuul1.x版的替代
![[无标题 21.png]]

### 微服务架构中网关在哪里
![[无标题 22.png]]
### 能干嘛
#### 反向代理
#### 鉴权
#### 流量控制
#### 熔断
#### 日志监控

### 总结
Spring Cloud Gateway组件的核心是一系列的过滤器，通过这些过滤器可以将客户端发送的请求转发(路由)到对应的微服务。 Spring Cloud Gateway是加在整个微服务最前沿的防火墙和代理器，隐藏微服务结点IP端口信息，从而加强安全保护。Spring Cloud Gateway本身也是一个微服务，需要注册进服务注册中心。
![[无标题 23.png]]

## Gateway三大核心
### 总述官网
[Glossary :: Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/glossary.html)
![[Pasted image 20240706161955.png]]

### 分
#### Route(路由)
路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

#### Predicate(断言)
开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言想匹配则进行路由。

#### Filter(过滤)
指的是Sping框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或之后对请求进行修改。

### 总结
![[无标题 24.png]]
web前端请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

predicate就是我们的匹配条件；

filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

## Gateway工作流程
### 官网介绍
![[无标题 25.png]]
客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前(Pre)或之后(Post)执行业务逻辑。

在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等;

在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

### 核心逻辑
路由转发+断言判断+执行过滤器链

## 入门配置
### 建Module
cloud-gateway9527
### 改POM
### 写YAML
### 主启动
### 业务类
无，不写任何业务代码，网关和业务无关
### 测试
## 9527网关如何做路由映射
### 如何做路由映射那？？？？
#### 诉求
我们目前不想暴露8001端口，希望在8001真正的支付微服务外面套一层9527网关
#### 8001新建PayGateWayController

```JAVA
@RestController  
@RequestMapping("pay")  
public class PayGateWayController {  
  
    @Resource  
    PayService payService;  
  
    @GetMapping(value = "/gateway/get/{id}")  
    public ResultData<Pay> getById(@PathVariable("id") Integer id)  
    {  
        Pay pay = payService.getById(id);  
        return ResultData.success(pay);  
    }  
  
    @GetMapping(value = "/gateway/info")  
    public ResultData<String> getGatewayInfo()  
    {  
        return ResultData.success("gateway info test："+ IdUtil.simpleUUID());  
    }  
  
}
```

#### 启动8001支付
#### 8001自测通过
### 9527网关YAML新增配置

```yaml
server:  
  port: 9527  
  
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          uri: http://localhost:8001                #匹配后提供服务的路由地址  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
  
        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          uri: http://localhost:8001              #匹配后提供服务的路由地址  
          predicates:  
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

### 测试
## Gateway高级特性
### Route以微服务名-动态获取服务URI
#### 痛点
uri地址写死
#### 是什么
[全局过滤器 ：： Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/global-filters.html#reactive-loadbalancer-client-filter)
![[Pasted image 20240707092947.png]]
#### 解决uri地址写死问题
将固定地址改为，提供服务的服务名；这样既是更改地址也可以查询的到
```Java
server:  
  port: 9527  
  
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
  
        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001              #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

### Predicate断言
#### 是什么
[路由谓词工厂 ：： Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/request-predicates-factories.html)
![[Pasted image 20240707101440.png]]
#### 启动微服务9527看看IDEA后台的输出

#### 常用的内置Route Predicate
##### 总体概述
两种配置，二选一：[Configuring Route Predicate Factories and Gateway Filter Factories :: Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/configuring-route-predicate-factories-and-filter-factories.html)
There are two ways to configure predicates and filters: shortcuts and fully expanded arguments. Most examples below use the shortcut way.
##### Shortcut Configuration
快捷方式配置由筛选器名称识别，后跟等号 （=），后跟以逗号 （,） 分隔的参数值。
##### Fully Expanded Arguments
完全扩展的参数看起来更像是带有name/value的标准 yaml 配置。通常，会有一个 name 键和一个 args 键。 args 键是用于配置谓词或过滤器的键值对的映射。

##### 常用断言api
id：我们自定义的路由ID，保持唯一
uri：目标服务地址
predicates：路由条件，Predicates接受一个输入参数返回一个布尔值
           该属性包含多种默认方法来将predicates组合成其他复杂的逻辑（如：与，或，非）

##### After Route Predicate Factory
After 路由谓词工厂采用一个参数，即日期时间（这是一种 java ZonedDateTime）。此谓词匹配指定日期时间之后发生的请求。以下示例配置后路由谓词。
###### 如何获得ZonedDataTime

```java
ZonedDateTime zbj = ZonedDateTime._now_(); _//_ _默认时区_              
System.**_out_**.println(zbj);
```
###### YAML
```YAML
server:  
  port: 9527  
  
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问
```
##### Before Route Predicate Factory
Before 路由谓词工厂采用一个参数，即日期时间（这是一种 java ZonedDateTime）。此谓词匹配在指定日期时间之前发生的请求。以下示例配置 before 路由谓词：
###### YAML

```
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由    
            - Before=2024-07-07T11:10:59.094803500+08:00[Asia/Shanghai] #在具体某个时间之前可以访问
```
##### Between Route Predicate Factory
Between 路由谓词工厂采用两个参数：datetime1 和 datetime2，它们是 java ZonedDateTime 对象。此谓词匹配在 datetime1 之后和 datetime2 之前发生的请求。 datetime2 参数必须位于 datetime1 之后。以下示例配置了 Between 路由谓词：
###### YAML

```yaml
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由    
            - Between=2024-07-07T11:16:00.094803500+08:00[Asia/Shanghai],2024-07-07T11:17:00.094803500+08:00[Asia/Shanghai]
```

##### Cookie Route Predicate Factory
Cookie 路由谓词工厂采用两个参数：cookie 名称和正则表达式（这是一个 Java 正则表达式）。此谓词匹配具有给定名称且其值与正则表达式匹配的 cookie。以下示例配置 cookie 路由谓词工厂：
###### YAML

```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            - Cookie=username,QingJiu
```

##### Header Route Predicate Factory
Header 路由谓词工厂采用两个参数，即 header 和 regexp（这是一个 Java 正则表达式）。此谓词与具有给定名称且其值与正则表达式匹配的标头匹配。以下示例配置标头路由谓词：
###### YAML

```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            #- Before=2024-07-07T11:10:59.094803500+08:00[Asia/Shanghai] #在具体某个时间之前可以访问  
            #- Between=2024-07-07T11:16:00.094803500+08:00[Asia/Shanghai],2024-07-07T11:17:00.094803500+08:00[Asia/Shanghai] #在两个具体时间  
            #之间才能访问  
            #- Cookie=username,QingJiu  
            - Header=X-Request-Id,\d+ #请求头要有X-Request-Id属性并且值为整数的正则表达式
```

##### Host Route Predicate Factory
Host 路由谓词工厂采用一个参数：主机名模式列表。该模式是 Ant 风格的模式，带有 .作为分隔符。此谓词与与模式匹配的主机标头相匹配。以下示例配置主机路由谓词：
###### YAML
```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            - Host=**.QingJiu.com
```

##### Query Route Predicate Factory
查询路由谓词工厂采用两个参数：必需的 param 和可选的 regexp（这是 Java 正则表达式）。以下示例配置查询路由谓词：
###### YAML
```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            - Query=username,\d+ #要有参数名username并且值还要是整数才能路由
```

##### RemoteAddr Route Predicate Factory
RemoteAddr 路由谓词工厂采用源列表（最小大小为 1），这些源是 CIDR 表示法（IPv4 或 IPv6）字符串，例如 192.168.0.1/16（其中 192.168.0.1 是 IP 地址，16 是子网掩码） ）。以下示例配置 RemoteAddr 路由谓词：
###### YAML

```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            - RemoteAddr=192.168.34.1/24
```

##### Method Route Predicate Factory
方法路由谓词工厂采用一个方法参数，该参数是一个或多个参数：要匹配的 HTTP 方法。以下示例配置方法路由谓词：
###### YAML
```YAML
spring:  
  application:  
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内  
  cloud:  
    consul: #配置consul地址  
      host: localhost  
      port: 8500  
      discovery:  
        prefer-ip-address: true  
        service-name: ${spring.application.name}  
    gateway:  
      routes:  
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名  
          #uri: http://localhost:8001                #匹配后提供服务的路由地址  
          uri: lb://cloud-payment-service  
          predicates:  
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由  
            - After=2024-07-07T11:05:59.094803500+08:00[Asia/Shanghai] #在什么时间之后才可以访问  
            - Method=POST,GET  
```

##### 小总结：
Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。

#### 自定义断言
##### 痛点
###### 原有的断言配置不满足业务怎么办

###### 模板套路
1. 要么继承AbstractRoutePredicateFactory抽象类
2. 要么实现RoutePredicateFactory接口
3. 开头任意取名字，但必须是以RoutePredicateFactory后缀结尾
##### 自定义路由断言规则步骤套路
1. 新建类名XXX需要以RoutePredicateFactory结尾并继承AbstractRoutePredicateFactory类
	
```java
@Component  
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>  
{  
    
}
```
2. 重写apply方法
	
```java
@Override  
public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config)  
{  
}
```

3. 新建apply方法所需要的静态内部类，这个Config类就是我们的路由断言规则
	
```java
@Validated  
public static class Config{  
    @Setter  
    @Getter    @NotEmpty    private String userType; //钻、金、银等用户等级  
}
```

4. 空参构造方法，内部调用super
	
```java
public MyRoutePredicateFactory()  
{  
    super(MyRoutePredicateFactory.Config.class);  
}
```

5. 重写apply方法第二版
	
```java
@Override  
public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config)  
{  
    return new Predicate<ServerWebExchange>()  
    {  
        @Override  
        public boolean test(ServerWebExchange serverWebExchange)  
        {  
            //检查request的参数里面，userType是否为指定的值，符合配置就通过  
            String userType = serverWebExchange.getRequest().getQueryParams().getFirst("userType");  
  
            if (userType == null) return false;  
  
            //如果说参数存在，就和config的数据进行比较  
            if(userType.equals(config.getUserType())) {  
                return true;  
            }  
  
            return false;  
        }  
    };  
}
```
##### Bug分析
在缺少shortcutFieldOrder方法的实现，所有不支持短格式

##### 补全代码

```java
public List<String> shortcutFieldOrder() {  
    return Collections.singletonList("userType");  
}
```
即可在YAML文件中使用短格式来做路由断言

### Filter过滤
#### 概述
##### 官网：[GatewayFilter Factories ：： Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/gatewayfilter-factories.html)
![[Pasted image 20240708102235.png]]
##### 一句话：
SpringMVC里面的拦截器Interceptor,Servlet的过滤器;"pre"和"post"分别会在请求被执行前调用和被执行后调用，用来修改请求和响应信息。

##### 能干吗
请求鉴权
异常处理
记录接口调用时长统计

##### 类型
###### 全局默认过滤器：Global Filters
> 当请求与路由匹配时，过滤 Web 处理程序会将 GlobalFilter 的所有实例和 GatewayFilter 的所有特定于路由的实例添加到过滤器链中。这个组合的过滤器链由 org.springframework.core.Ordered 接口排序，您可以通过实现 getOrder() 方法来设置。 由于 Spring Cloud Gateway 区分过滤器逻辑执行的“前”和“后”阶段（请参阅工作原理），具有最高优先级的过滤器是“前”阶段的第一个和“后”阶段的最后一个 -阶段。 
> 
> gateway出厂默认已有的，直接用即可，主要作用于所有的路由。
> 
> 不需要在配置文件中配置，作用在所有的路由上，实现Global Filters接口即可。
###### 单一内置过滤器：GatewayFilter
> 路由过滤器允许以某种方式修改传入的 HTTP 请求或传出的 HTTP 响应。路由过滤器的范围仅限于特定路由。 Spring Cloud Gateway 包含许多内置的 GatewayFilter Factory。
> 
> 也可以称为网关过滤器，这种过滤器主要是作用于单一路由或者某个路由分组。
###### 自定义过滤器

#### Gateway内置的过滤器
##### 常用的内置过滤器
###### 请求头相关组RequestHeader
AddRequestHeader GatewayFilter Factory：
> AddRequestHeader GatewayFilter 工厂采用名称和值参数；此列表将 AddRequestHeader=X-Request-QingJiu1,QingJiuValue1标头添加到所有匹配请求的下游请求标头中。
> 指定请求头内容ByName.
> 
```yaml
- id: pay_routh3  
  uri: lb://cloud-payment-service  
  predicates:  
    - Path=/pay/gateway/filter/**  
  filters:  
    - AddRequestHeader=X-Request-QingJiu1,QingJiuValue1 #请求头kv，若一头含有多参则重写一行设置  
    - AddRequestHeader=X-Request-QingJiu2,QingJiuValue2
```

RemoveRequestHeader GatewayFilter：
> RemoveRequestHeader GatewayFilter 工厂采用名称参数。它是要删除的标头的名称。这会在发送到下游之前删除指定标头。

SetRequestHeaderGatewayFilter：
> SetRequestHeader GatewayFilter 工厂采用名称和值参数。此 GatewayFilter 替换（而不是添加）具有给定名称的所有标头。


###### 请求参数相关组RequestParameter
AddRequestParameter GatewayFilter Factory ：
> AddRequestParameter GatewayFilter Factory 采用名称和值参数。 AddRequestParameter 知道用于匹配路径或主机的 URI 变量。 URI 变量可以在值中使用并在运行时扩展。

RemoveRequestParameter GatewayFilter：
> RemoveRequestParameter GatewayFilter 工厂采用名称参数。它是要删除的查询参数的名称。这将在向下游发送之前删除指定参数。


###### 回应头相关组
AddResponseHeader GatewayFilter Factory
> AddResponseHeader GatewayFilter Factory 采用名称和值参数。AddResponseHeader 知道用于匹配路径或主机的 URI 变量。 URI 变量可以在值中使用并在运行时扩展。

RemoveResponseHeader GatewayFilter Factory
>RemoveResponseHeader GatewayFilter 工厂采用名称参数。它是要删除的标头的名称。 要删除任何类型的敏感标头，您应该为您可能想要执行此操作的任何路由配置此过滤器。此外，您可以使用 spring.cloud.gateway.default-filters 配置此过滤器一次，并将其应用于所有路由。

SetResponseHeader GatewayFilter Factory
>SetResponseHeader GatewayFilter 工厂采用名称和值参数。 SetResponseHeader 知道用于匹配路径或主机的 URI 变量。 URI 变量可以在值中使用，并将在运行时扩展。

###### 前缀和路径相关组
PrefixPath GatewayFilter Factory
> PrefixPath GatewayFilter 工厂采用单个前缀参数。

SetPath GatewayFilter Factory
> SetPath GatewayFilter 工厂采用路径模板参数。它提供了一种通过允许路径的模板化段来操作请求路径的简单方法。这使用了 Spring Framework 中的 URI 模板。允许多个匹配段。

RedirectTo GatewayFilter Factory
> RedirectTo GatewayFilter 工厂采用三个参数：status、url 和可选的 includeRequestParams。 status 参数应为 300 系列重定向 HTTP 代码，例如 301。 url 参数应为有效的 URL。这是 Location 标头的值。 includeRequestParams 参数指示 url 中是否应包含请求查询参数。如果不设置，将被视为 false。对于相对重定向，您应该使用 uri: no://op 作为路由定义的 uri。

#### Gateway自定义过滤器
##### 自定义全局Filter
统计接口调用耗时情况，如何落地，谈谈设计思路，通过自定义全局过滤器搞定上述需求。
###### 案例
新建类实现GlobalFiter,Ordered两个接口

```java
package com.QingJiu.cloud.mygateway;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.cloud.gateway.filter.GatewayFilterChain;  
import org.springframework.cloud.gateway.filter.GlobalFilter;  
import org.springframework.core.Ordered;  
import org.springframework.stereotype.Component;  
import org.springframework.web.server.ServerWebExchange;  
import reactor.core.publisher.Mono;  
  
import java.util.Map;  
  
@Slf4j  
@Component  
public class MyGlobalFilter implements GlobalFilter, Ordered {  
  
  
    //开始调用方法的时间  
    public static final String BEGIN_VISIT_TIME = "begin_visit_time";  
  
  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
  
        //1 现记录下访问接口的开始时间  
        exchange.getAttributes().put(BEGIN_VISIT_TIME,System.currentTimeMillis());  
  
        //2 返回统计的各个结果给后台  
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {  
            Long beginVisitTime = exchange.getAttribute(BEGIN_VISIT_TIME);  
            if (beginVisitTime != null){  
                log.info("访问接口主机："+exchange.getRequest().getURI().getHost());  
                log.info("访问接口端口："+exchange.getRequest().getURI().getPort());  
                log.info("访问接口URL："+exchange.getRequest().getURI().getPath());  
                log.info("访问接口参数："+exchange.getRequest().getURI().getRawQuery());  
                log.info("访问接口时间长短："+(System.currentTimeMillis() - beginVisitTime) + "毫秒");  
                log.info("=============");  
                System.out.println();  
            }  
        }));  
    }  
  
    /**  
     * 数字越小优先级越高  
     * @return  
     */  
    @Override  
    public int getOrder() {  
        return 0;  
    }  
}
```


##### 自定义条件Filter
###### 自定义，单一内置过滤器
###### 参考GateWay内置出厂默认

###### 自定义网关过滤器规则步骤套路
新建类MyGatewayFilterFactory并继承AbstractGatewayFilterFactory类

```java
package com.QingJiu.cloud.mygateway;  
  
import lombok.Getter;  
import lombok.Setter;  
import org.springframework.cloud.gateway.filter.GatewayFilter;  
import org.springframework.cloud.gateway.filter.GatewayFilterChain;  
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;  
import org.springframework.http.HttpStatus;  
import org.springframework.http.server.reactive.ServerHttpRequest;  
import org.springframework.stereotype.Component;  
import org.springframework.web.server.ServerWebExchange;  
import reactor.core.publisher.Mono;  
  
import java.util.Arrays;  
import java.util.List;  
  
@Component  
public class MyGatewayFilterFactory extends AbstractGatewayFilterFactory<MyGatewayFilterFactory.Config> {  
  
}
```

新建MyGatewayFilterFactory.Config内部类

```java
public static class Config{  
  
    @Getter@Setter  
    private String status; //设定一个状态值/标志位，它等于多少，匹配后才可以访问  
}
```

重写apply方法

```Java
@Override  
public GatewayFilter apply(MyGatewayFilterFactory.Config config) {  
    return new GatewayFilter() {  
        @Override  
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
            ServerHttpRequest request = exchange.getRequest();//先得到请求头  
            System.out.println(" 进入了自定义网关过滤器MyGatewayFilterFactory，status " + config.getStatus());  
            if (request.getQueryParams().containsKey("QingJiu")){//得到请求路径的参数kay是否包含"QingJiu"  
                return chain.filter(exchange); //存在放行  
            }else {  
                exchange.getResponse().setStatusCode(HttpStatus.BAD_REQUEST);  
                return exchange.getResponse().setComplete();  
            }  
        }  
    };  
}
```

重写shortcutFieldOrder

```java
@Override  
public List<String> shortcutFieldOrder() {  
    return Arrays.asList("status");  
}
```

空参构造方法，内部调用super

```java
public MyGatewayFilterFactory() {  
    super(MyGatewayFilterFactory.Config.class);  
}
```

## Gateway整合阿里巴巴Sentinel实现容错