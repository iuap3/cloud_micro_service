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
#### 4：注解的参数设置请参考[接口开发，注解使用](../2.4/manual.md)。
