
# 配置说明
<br>

## Maven工程pom.xml引用配置:

	<dependency>
		<groupId>com.yonyou.cloud.middleware</groupId>
		<artifactId>mwclient</artifactId>
		<version>5.2.1-RELEASE</version>
		<type>pom</type>
	</dependency>


## application.properties配置

	# 从开发者中心申请的认证Key
	access.key=xxxxxxxxxxxxxxxx
	# 与上述认证key对应的认证秘钥
	access.secret=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
	
	# 应用名称, 应与开发者中心的应用CODE一致
	spring.application.name=xxxx
	# 应用部署的环境(开发:dev,测试:test,灰度:stage,线上:online)
	spring.profiles.active=dev
	
	#开发调试时指定IP调用的配置(默认为false, yml下值应加上双引号)
	client.usemock=false
	# 启用mock调用时针对某个调用者的具体IP指定
	appCode@providerId@profile=172.20.1.1

	#5.2.1版本新增配置微服务动态配置(默认为false，即不拉取配置文件更新,yml下值应加上双引号)
	app.metainfo.ms_dyn_enable=true

## applicationContext.xml配置

### · 典型最小化配置

	<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	    <property name="locations">
	        <list>
	            <value>classpath*:/application.properties</value>
	        </list>
	    </property>
	    <property name="systemPropertiesMode" value="2"></property>
	</bean>
	
	<!-- 使用annotation 自动注册bean, 并保证@Required、@Autowired的属性被注入 -->
	<context:component-scan base-package="com.yonyou.cloud">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
		<context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
	</context:component-scan>



### · XML方式注册远程调用接口
在没有引用标注@RemoteCall注解的远程调用接口时注册远程调用接口, 适用于服务方没有发布API包的情况.
	
	<bean id="chartTypeService" class="com.yonyou.cloud.middleware.rpc.RPCBeanFactory">
	    <constructor-arg value="${remote-service.appCode}"></constructor-arg>
	    <constructor-arg value="${remote-service.providerId}"></constructor-arg>
	    <constructor-arg>
	    	<list>
	        	<value>${remote-service.fullQualifiedClassName}</value>
	        </list>
	    </constructor-arg>
	</bean>

## app.metainfo

app.metainfo.xxx是微服务的各项配置(所有的value都是字符串)

配置项|默认值|描述
:---:|:---:|:---
ms_dyn_enable| true | 微服务动态控制开启
ribbon.retryOnAllErrors|false|rpc异常是否重试
ribbon.loadBalancer|com.yonyou.cloud.middleware.loadbalancer.ZoneAwareLoadBalancer|负载均衡loadBalancer
ribbon.ping|com.yonyou.cloud.middleware.iris.loadbalancer.check.MetricDiscoveryPing|负载均衡ping实现
ribbon.serverListUpdater|com.yonyou.cloud.middleware.eureka.EurekaNotificationServerListUpdater|服务列表更新updater
ribbon.rule|com.yonyou.cloud.middleware.loadbalancer.AvailabilityFilteringRule|负载均衡rule
ribbon.IClient|com.yonyou.cloud.middleware.rpc.RpcRetryClient|rpc client
ribbon.retryNext|3|rpc重试其他节点次数
ribbon.retrySame|0|rpc重试当前节点次数
ribbon.retryOnConnectErrors|true|网络异常是否重试