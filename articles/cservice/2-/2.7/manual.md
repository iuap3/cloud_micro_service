# 服务的调用


#### 1：调用方需创建一个应用(例如：ms-client)作为服务的调用方，创建方法请参考[微服务应用创建](../2.2/manual.md)。
#### 2：对创建好的应用进行配置，配置方法请参考[基础工程创建与配置](../2.3/manual.md)。
#### 3：注解调用，通过@RemoteCall将需要调用的应用注解进来
```
@RemoteCall("ms-middle@c87e2267-1001-4c70-bb2a-ab41f3b81aa3")
public interface IMsMiddleService {
    @ApiOperation(value = "获取随机数字", response = String.class)
    public String getDigit();
}
```
#### 4：@RemoteCall注解的参数设置请参考[接口开发，注解使用](../2.4/manual.md)。
#### 5：服务调试
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
