# 服务启动注册、RPC调用

描述：

## 服务启动

**1：使用Jetty启动**

- 查看pom.xml中jetty-maven-plugin的端口配置,使用maven命令: `mvn jetty:run` 进行启动;
- 在eclipse中可右键Debug As > Maven build... > 在弹出框的Goals输入框中填写jetty:run, 然后点击Debug即可启动.

**2：使用Tomcat方式启动**

- 将应用部署在tomcat中, 启动即可.


**3: 使用main()方法方式启动**

- 引入helix-appstarter的maven依赖:

    `<artifactId>helix-appstarter</artifactId>
    <groupId>com.yonyou.cloud.middleware</groupId>
    <version>2.2.1-SNAPSHOT</version>`

- Run As: `com.yonyou.cloud.middleware.YonyouCloudAppStarter`, 启动端口将以`application.properties`属性文件中的`server.port`配置项为准.

## 服务注册

**1：服务注册**
	
- 服务注册启动后会自动连接[服务注册中心](https://registry.yonyoucloud.com/ "服务注册中心"), 在服务注册中心根据租户ID和应用名称可以看到对应的实例列表, 实例的格式为:"IP:主机名:端口号.部署环境"(例如:`172.17.0.5:mymachine:10080.online`), 一般在启动后两分钟内可注册到服务注册中心.

**2：RPC调用**

- 发起RPC调用时, 服务提供方需要已经启动并注册到服务注册中心, 否则会报错:`can not find active server from eureka! server url is ...`

- 服务消费者和服务提供方都会注册到服务注册中心, 


# 常见问题

## 常见问题1：can not find active server from eureka! server url is ...

**出现此问题的原因有以下几种:**

- 服务提供方未启动或down掉

- 服务提供方因网络原因未注册到服务注册中心

- 服务提供方的部署时的对外服务端口和`application.properties`属性文件中配置的`server.port`属性值不一致.

- 服务提供方访问路径中的`ContextPath`与`application.properties`属性文件中配置的`spring.application.name`属性值不一致.

- 一般情况下, 项目名称、项目maven配置的artifactId、`application.properties`属性文件中配置的`spring.application.name`属性值 及 部署时的`ContextPath`这四项要一致; 项目部署时的对外服务端口和`application.properties`属性文件中配置的`server.port`属性值要一致, 如果使用内置Jetty还需要和`pom.xml`中`jetty-maven-plugin`插件的`port`值要一致.


## 常见问题2：项目往服务注册中心注册或注销操作有延迟.

**几个重要的时间段**

- 注册到服务注册中心:
	- 实例心跳上报时间间隔: 30秒; 第一次心跳在30秒后(参见:[HeartbeatThread](com.netflix.discovery.DiscoveryClient.initScheduledTasks() "HeartbeatThread") 和 [TimedSupervisorTask](com.netflix.discovery.TimedSupervisorTask.TimedSupervisorTask ("TimedSupervisorTask"))
	
- 实例状态刷新
	- 45秒如果服务注册中心未接收到心跳, 将会计划移除此实例.

- 拉取注册列表
	- 每30秒从服务注册中心拉取下最新的实例增量信息, 但服务增量信息将会在服务注册中心中保持3分钟.

- 注销
	- 在正常关闭容器(有执行shutdown操作而不是直接kill进程)的情况下, 客户端示例会向服务注册中心发送注销请求.
	- 服务器移除实例参见:[EvictionTask](com.netflix.eureka.registry.AbstractInstanceRegistry.EvictionTask "EvictionTask")

- 客户端注册/注销的时间延迟
	- 客户端注册/注销反应到所有的实例一般需要两分钟.
		- 注册:30秒注册+30标识为健康状态+30从服务注册中心拉取=90秒内.
		- 注销:直接kill进程情况下,43秒(优化前为60秒)定时移除过期实例和45秒(优化前为90秒)实例过期时间 +30秒被所有实例拉取=约75秒(优化前为120秒)内.

