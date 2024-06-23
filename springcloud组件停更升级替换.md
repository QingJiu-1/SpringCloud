###### Netflix OSS被移除的原因
	在2020年之前Netflix公司就是微服务组件实施标准，开源了诸多的微服务套件，统称为
	Netflix OSS；在2018年之后Netfli宣布核心组件不再进行新特性开发，只修bug。
	所以在2020年将其移除。

###### Cloud2024组件
	服务注册与发现：
		💡Eureka: Netflix公司目前为数不多推荐使用的
		Consul：Java语言时推荐
		Etcd：go语言推荐
		Nacos：由阿里巴巴开发
	
	服务调用和负载均衡：
		❌Ribbon：在Netflix OSS体系中使用，但现在被remove了
		OpenFeign：当今主角，是远程微服务、服务调用、面向接口编程
		LoadBalancer：重要和主流的组件，用于父接口调用的产品
	
	分布式事务
		Seata
		LCN
		Hmily
	
	服务熔断和降级
		❌Hystrix：如今停更
		Circuit Breaker：目前主流，这是一个接口和规范，这只是一个理念
			Resilience4J：由它实现
		Sentinel：阿里巴巴的哨兵
	
	服务链路追踪
		❌Sleuth来做数据的收集 + Zipkin来做图形化的展现：现如今被micrometer tracing替代
		micrometer tracing
	
	服务网关
		❌Zuul：以前使用
		Gate Way
	
	分布式配置管理
		❌Config+Bus：需要访问github，国内并不推荐
		Consul
		Nacos：目前主流的