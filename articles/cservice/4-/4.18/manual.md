# SpringCloud适配

---

## Feign迁移
## 配置文件

maven依赖新增

```xml
        <dependency>
            <groupId>com.yonyou.cloud.middleware</groupId>
            <artifactId>middleware</artifactId>
            <version>5.2.1-RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.yonyou.cloud.middleware</groupId>
            <artifactId>iris-springboot-support</artifactId>
            <version>5.2.1-RELEASE</version>
        </dependency>
        
```

配置文件新增

```yml
#公有云可不用此项
registry: http://xxxxxxx

access:
  key: xxxxx
  secret: xxxxxxxxxxx

#删除eureka、Feign相关配置
#eureka:
#  client:
#    serviceUrl:
#      defaultZone: http://127.0.0.1:8761/eureka/
```



## 代码
远程接口

```java
public interface CreditApi {
}

//修改为

@RemoteCall("服务提供方名字@providerid")
public interface CreditApi {
}
```

删除eureka和Feign相关注解和pom依赖

```java
@SpringBootApplication
//@EnableEurekaClient
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}

@SpringBootApplication
//@EnableEurekaClient
//@EnableFeignClients
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

删除被@FeignClient注解的class


## 建议
1. 远程接口的具体实现放在service层
2. 删除RequestMapping、GetMapping、PostMapping、RequestBody、RequestParam等注解


## openFeign适配

## 配置文件

* 修改pom.xml文件,增加如下配置
```xml
		<dependency>
			<groupId>com.yonyou.cloud.middleware</groupId>
			<artifactId>middleware</artifactId>
			<version>5.2.1-RELEASE</version>
		</dependency>
		<dependency>
			<groupId>com.yonyou.cloud.middleware</groupId>
			<artifactId>iris-springboot-support</artifactId>
			<version>5.2.1-RELEASE</version>
		</dependency>
		<!-- 服务调用方需要依赖下方的包，如果是服务提供方则不需要 -->
		<!-- 
		<dependency> 
			<groupId>com.yonyou.cloud.middleware</groupId> 
			<artifactId>iris-springcloud-openfeign</artifactId> 
			<version>5.2.1-SNAPSHOT</version> 
		-->

```

* 在application.properties 件中增加如下配置
```properties
#test会将微服务信息注册到测试环境中,且必须在相同的profile下才能进行通信
spring.profiles.active=test
access.key=XXXXXXX
access.secret=XXXXXX

``` 

* 服务提供方提供的对外接口,这个会作为元数据注入到微服务治理平台中的API列表中.(服务调用方不需要)

```java
@Bean
public RPCBeanFactory factory() {
    return new RPCBeanFactory("${appCode}",
                              Collections.singletonList("${className}"));
}
```