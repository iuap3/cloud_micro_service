# 服务治理平台常见问题


## 服务启动常见问题

### 1：微服务服务工程启动失败，不能找到app

微服务工程启动过程中，会在开发者中心下检查配置的AccessKey对应的用户是否存在此应用，如果持续集成和应用管理下存在此应用，校验通过后可以启动；如果不存在此应用，请根据前面的文档创建该应用。

当工程中配置的应用和租户下其他用户的应用重名时，可能导致创建应用失败，最终导致服务不能启动。

建议开发者在启动前确认是否存在此应用，或者在开发者中心的持续集成中，明确创建此应用，并选择微服务的类型，避免自动创建工程遇到问题。


### 2：微服务启动报错，没有应用相关权限

开发者启动过程中，控制台打印没有应用的权限，请检查配置的AccessKey所属的用户是否具有应用编码对应的应用的权限，一般造成这种问题的原因是使用了其他人的AccessKey信息或者管理员未给当前用户分配应用权限导致。

开发者可以检查AK信息，并检查应用是否被管理员授权排查问题。

### 3：SDK 组件依赖问题

开发者可以从已有的工程进行改造，来生成新的支持微服务调用的工程，原工程中可能存在旧版本的微服务的SDK和旧版本的IUAP组件，5.2.1-RELEASE版本引用的各个组件的版本如下，可以帮助开发者排查依赖.


- mwclient 5.2.1-RELEASE pom类型
- middleware 5.2.1-RELEASE pom类型
- iuap 3.2.1-SNAPSHOT
- auth-sdk-client 1.0.15-SNAPSHOT
- iris-springboot-support 5.2.1-RELEASE
- iris-iuap-support 5.2.1-RELEASE
- iris-dubbox-support 5.2.1-RELEASE
- eos-spring-support 5.2.1-RELEASE

### 4：工程更新，pom.xml中依赖更新不到

微服务治理平台提供的SDK默认存放的Maven仓库地址为maven.yonyou.com，用友办公网可直接访问，如果不能访问用友办公网，请通过其他渠道购买或者获得相关组件后，推送到私有的Maven仓库中使用。

### 5：提示服务验证失败.请确认accessKey 和application.name的正确性!

- 输入的access.key和access.secret不一致或者错误
- 此AccessKey已被停用或者删除
- application.name即应用编码输入错误
- 此用户下没有应用编码为application.name的应用

### 6：can not find active server from eureka! server url is ...

- 服务提供方未启动或down掉
- 服务提供方因网络原因未注册到服务注册中心
- 服务提供方的部署时的对外服务端口和`application.properties`属性文件中配置的`server.port`属性值不一致.
- 服务提供方访问路径中的`ContextPath`与`application.properties`属性文件中配置的`spring.application.name`属性值不一致.


一般情况下, 项目名称、项目maven配置的artifactId、`application.properties`属性文件中配置的`spring.application.name`属性值、部署时的`ContextPath`及RemoteCall注解的应用编码（RemoteCall注解由应用编码@租户id组成）这五项内容要一致; 项目部署时的对外服务端口和`application.properties`属性文件中配置的`server.port`属性值要一致, 如果使用内置Jetty还需要和`pom.xml`中`jetty-maven-plugin`插件的`port`值要一致.

### 7：部署到开发者中心环境变量覆盖问题

微服务应用部署到开发者中心时，如果使用的是旧版本的微服务治理平台SDK，可能由于环境变量的问题导致配置不生效，请检查属性中的access_key、access_secret、mw_profiles_active等项是否有效或者与配置文件一致。5.1.1-RELEASE版本的SDK已经对此问题进行适配。

如果不匹配，请删除几项环境变量或者修改成与配置中相同的值，保存并重启。修改环境变量的位置为：容器服务->应用管理->具体应用->对应环境->“属性”页签, 详细信息中的环境列表展示了所有的环境变量。

mw_profiles_active的值建议修改成dev、test、stage、online，分别对应开发、测试、灰度、生产。

### 8：关于application.name、ContextPath及RemoteCall注解一致性问题

关于配置一致性问题，参考如下示例：

1.application.properties.name:
>spring.application.name=rpc-provider

2.application.properties.contextPath:
>server.servlet.context-path=/rpc-provider

3.RemoteCall注解：
>@RemoteCall(rpc-provider@租户id)

### 9：关于server端接收不到数据请求问题：

解决方案：/CloudRemoteCall 路径不要被拦截

## 微服务控制台常见问题

### 1：应用工程的创建过程问题

在开发者中心界面上，微服务可以通过从两种方式来创建：

1. 创建普通的java web工程，工程改造成微服务工程，上传war包重新部署
2. 通过持续构建，创建微服务类型应用，开发完成后再进行部署

使用者可以根据自身情况，选择新建和改造工程以达到微服务化的效果。

### 2：微服务应用的实例信息问题

微服务应用启动后，从容器服务菜单下的应用管理下，搜索到相应的应用，展开卡片信息，选择实例页签，可以查看实例信息，如下图所示：

![](images/docker-ins.png)

此实例信息代表Docker容器的实例，Docker容器中运行着微服务工程。

此外，展开微服务的服务管理菜单，查询到对应的微服务，展开监控页签，可以查看微服务注册中心的实例信息，如下图所示：

![](images/registry-ins.png)

此实例信息代表微服务启动后注册到注册中心的实例信息。

### 3：异步调用消息中间件设置问题

使用微服务治理平台的异步调用SDK时，需确保已经在管理界面对应用设置了定制的RabbitMQ中间件的连接地址，此地址可以是服务治理平台提供的公共地址，也可以是用户按照治理平台的要求启动的定制的MQ，设置位置如下图所示：

![](images/mqsetting.png)

### 4：微服务间应用依赖问题

微服务间成功调用后，可以从服务管控界面查看应用或者接口的依赖信息，需要注意的是，微服务后端服务是计算依据为当前时间的前一天的调用记录,会有一定的延时性。

### 5：限流设置不能及时生效问题

微服务治理平台的限流配置和权限控制是通过配置中心推送配置文件生效的，配置文件的推送和生效需要5-10秒的延时，会导致调用时权限和限流生效一定延时，请在一定时间后验证设置的效果。

### 6：接口权限设置问题

服务的接口设置成私有权限后，需要对允许调用的客户端工程进行授权，调用方才能调用通过，设置授权时，请注意选择正确的应用的环境。

开发态和测试态的服务间相互调用，没有权限的验证，请开发者注意。

### 7：服务调用链路查询问题

针对微服务暴露出的接口，可以查看具体方法的调用链路信息，多级调用时，可以进入详细信息页面查看调用的层级，查询条件中支持根据业务关键字查询，业务关键字需要开发者在开发过程中设置，详细设置的API请参考开发手册中的相关章节。

![](images/zipkin.png)


### 8. 本地启动test环境
> 本地启动时会把本地启动的服务注册到注册中心，这样会被调用方发现并调用。

解决方案(任选其一)：

在代码的/src/test/resources/目录中放置一个与应用的application.properties对应的application.yml,里面设置app.version=xxx

或：

加上环境变量  -Deureka.registration.enabled=false不会把本服务注册到注册中心

### 9. can not find active app from registry
>不能找到活跃的实例问题；

现象: 服务重启、构建过程中，本地实例注册到注册中心，客户端轮询拉取，在时间间隔内，未及时拉取到，存在1分钟左右的延迟；

原因: 健康检查配置的是基于端口的检测，端口启动时候新应用还没真正启动完毕，老的实例被提前杀死；

解决方案：

在5.1.x版本及以前版本: 开发者中心配置健康检查时，添加微服务的健康检查，具体如下：

进入开发者中心->应用管理->找到对应服务->属性->右侧编辑按钮->下拉至健康检查->配置路径。注意路径最后以"/"结尾。

![](images/registry.png)

注意：在后续即将发布的5.2.1+版本会增加实时上下线通知的功能, 从根本上解决该问题.

### 10.关于jetty下启动慢的问题

原因：
>jetty启动的情况下，未正确配置jetty-context.xml

解决方案：

第一步：配置jetty版本：

	<jetty.version>9.4.18.v20190429</jetty.version>

第二步：配置jetty依赖：

	<dependency>
	    <groupId>org.eclipse.jetty</groupId>
	    <artifactId>jetty-webapp</artifactId>
	    <version>${jetty.version}</version>
		<scope>test</scope>
		<exclusions>
			<exclusion>
				<groupId>javax.servlet</groupId>
				<artifactId>servlet-api</artifactId>
			</exclusion>
		</exclusions>
	</dependency>

第三步：配置jetty插件：

	<plugin>
   		<groupId>org.eclipse.jetty</groupId>
   		<artifactId>jetty-maven-plugin</artifactId>
   		<version>${jetty.version}</version>
	<configuration>
		<contextXml>${project.basedir}/src/test/resources/jetty-context.xml</contextXml>
		<webAppConfig>
			<contextPath>/${project.artifactId}</contextPath>
			<defaultsDescriptor>${project.basedir}/src/test/resources/webdefault.xml</defaultsDescriptor>
		</webAppConfig>
		<httpConnector>
			<port>8080</port>
			<idleTimeout>60000</idleTimeout>
		</httpConnector>
		<stopPort>9090</stopPort>
		<stopKey>shutdown</stopKey>
	</configuration>
	</plugin>

第四步：配置jetty的配置文件，放在工程的/src/test/resources目录下:

请点击配置文件名称进行下载。

[webdefault.xml](https://developer.yonyoucloud.com/download/microservice/jetty-config/webdefault.xml)

[jetty-context.xml](https://developer.yonyoucloud.com/download/microservice/jetty-config/jetty-context.xml)

### 11.关于升级5.2.1版本后eureka-client参数设置问题

eureka-client的两个参数，一个涉及心跳的，一个涉及拉取变化。

心跳对应的参数在5.2.1版本之前设置为iris.client.cacheRefresh.exponentialBackOffBound，5.2.1版本设置为eureka.client.cacheRefresh.exponentialBackOffBound。5.2.1版本也提供在配置文件中声明app.metainfo.iris.client.heartbeat.exponentialBackOffBound的方式。

拉取变化对应的参数在5.2.1版本之前设置为iris.client.heartbeat.exponentialBackOffBound，5.2.1版本设置为eureka.client.heartbeat.exponentialBackOffBound。

### 12.eos-spring-support、tcc版本与SDK版本不匹配问题

tcc和eos-spring-support版本升级为5.2.1后，SDK版本必须也升级到5.2.1，否则启动时将会出现如下异常：

![](images/tccError.png)
