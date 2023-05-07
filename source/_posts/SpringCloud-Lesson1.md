#[Spring Cloud](https://spring.io/projects/spring-cloud-alibaba)

## 什么是Spring Cloud

快速构建分布式系统

![Spring Cloud diagram](https://spring.io/images/cloud-diagram-1a4cad7294b4452864b5ff57175dd983.svg)



### [1. Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/3.1.0/reference/html/#configuring-route-predicate-factories-and-gateway-filter-factories)

- 路由功能
- Filter（修改请求参数与返回参数等）
- 负载均衡
- 接口限流与短路（配合断路器Hystrix）

```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org/user/xxx
        predicates:
        - Cookie=mycookie,mycookievalue
```

### [2. Nacos Config](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html#_spring_cloud_alibaba_nacos_config)

- 基于profile实现多环境隔离
- 支持配置的动态实时更新
- 配置历史版本管理

pom.xml

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

Bootstrap.yaml

```
spring:
  cloud:
    nacos:
      config:
        enabled: true
        file-extension: yaml
        namespace: ${spring.cloud.nacos.discovery.namespace}
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        group: ${spring.application.name}
        name: ${spring.application.name}
        
```

```
@SpringBootApplication
@EnableNacosConfig
public class TaskEngineLauncher {

    public static void main(String[] args) {
        SpringApplication.run(TaskEngineLauncher.class, args);
    }
}
```

[Nacos开发环境](http://dev-nacos.iguming.net:8848/nacos/#/configurationManagement?dataId=&group=&appName=&namespace=&pageSize=&pageNo=)

### [3. Nacos Discovery](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html#_spring_cloud_alibaba_nacos_config)

- 服务注册/发现
- Provider/Consumer/ServiceRegistry

![架构图](https://cdn.nlark.com/lark/0/2018/png/15914/1542119181336-b6dc0fc1-ed46-43a7-9e5f-68c9ca344d60.png)

pom.xml

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

bootstrap.yaml

```
spring:
  cloud:
    nacos:
      discovery:
        server-addr: dev-nacos.iguming.net:8848
        namespace: 585c2892-3945-4061-af88-e077fc64f33b
```

```
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(NacosProviderApplication.class, args);
	}
	
	@RestController
	class EchoController {
		@RequestMapping(value = "/echo/{string}", method = RequestMethod.GET)
		public String echo(@PathVariable String string) {
			return "Hello Nacos Discovery " + string;
		}
	}
}
```



###[4. OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/3.1.0/reference/html/)

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

```
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    Page<Store> getStores(Pageable pageable);

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);

    @RequestMapping(method = RequestMethod.DELETE, value = "/stores/{storeId:\\d+}")
    void delete(@PathVariable Long storeId);
}
```



### [5. 分布式链路追踪-ARMS](https://help.aliyun.com/product/34364.html)

### [6. 分布式消息系统-RocketMQ](https://github.com/apache/rocketmq/tree/master/docs/cn)

