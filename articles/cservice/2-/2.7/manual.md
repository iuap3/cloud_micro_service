# 服务的调用


#### 1：调用方需创建一个应用(例如：ms-client)作为服务的调用方，创建方法请参考[微服务应用创建](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/2-%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BA%94%E7%94%A8%E5%88%9B%E5%BB%BA/manual.md)。
#### 2：对创建好的应用进行配置，配置方法请参考[基础工程创建与配置](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/3-%E5%9F%BA%E7%A1%80%E5%B7%A5%E7%A8%8B%E5%88%9B%E5%BB%BA%E4%B8%8E%E9%85%8D%E7%BD%AE/manual.md)。
## 调用方式

#### 1：注解调用，通过@RemoteCall将需要调用的应用注解进来
```
@RemoteCall("ms-middle@c87e2267-1001-4c70-bb2a-ab41f3b81aa3")
public interface IMsMiddleService {
    @ApiOperation(value = "获取随机数字", response = String.class)
    public String getDigit();
}
```
@RemoteCall注解的参数设置请参考[接口开发，注解使用](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/4-%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91%E3%80%81%E6%B3%A8%E8%A7%A3%E4%BD%BF%E7%94%A8/manual.md)。

#### 2：consumer bean
此方式提供在调用应用时，通过在applicationContext.xml注册bean的方式调用
```
<bean id="middleService" class="com.yonyou.cloud.middleware.rpc.RPCBeanFactory">
	<constructor-arg value="ms-middle"></constructor-arg>
	<constructor-arg value="35568e76-1ef1-4d77-b5cf-8fb66d2c8002"></constructor-arg>
	<constructor-arg>
		<list>
			<value>com.yonyou.cloud.ms.service.IMsMiddleService</value>
		</list>
	</constructor-arg>
</bean>
```
constructor-arg 的第一个value为应用的appCode,第二个value为应用的providerId，第三个value为提供服务接口列表，其值必须是以类全名的形式出现。

#### 3：多服务IP调用
在开发阶段，同一应用被多名开发或者测试人员注册到Eureka中心，可以通过指定某一IP来调用服务。
- 在application.properties文件中添加并修改client.usemock=true(false为禁用)。
- 新建mock.properties文件来指定服务的具体IP地址：
- mock.ip=127.0.0.1
- mock.port=8080

#### 4：多服务IP调用指定类
在服务开发阶段，有可能会存在很多不确定的服务，如果想在开发阶段指定调用一个特定的服务实例，可以通过配置指定IP的形式来进行调用
```
<bean id="billCodeService" class="com.yonyou.cloud.middleware.rpc.RPCStubBeanFactory">
    <property name="appCode" value="iuap-saas-billcode-service-cloud"></property>
    <property name="providerId" value="c87e2267-1001-4c70-bb2a-ab41f3b81aa3"></property>
    <property name="serviceInterface" value="com.yonyou.uap.billcode.service.IBillCodeService"></property>
    <property name="serviceIp" value="10.124.1.255" />
</bean>
```
通过进行bean的配置，设置serviceIp为需要调用实例的IP和端口号，可以达到指定调用的目的.
## ZONE调用
- 服务端配置

    服务端在启动时，需要在application.properties中配置zone，添加eureka.instance.metadataMap.zone=middleZone
- 客户端配置

    客户端在启动时，也需要在application.properties中配置zone，其配置方式和服务端的配置一样。

