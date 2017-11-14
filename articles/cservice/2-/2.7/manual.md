# 服务的调用


#### 1：调用方需创建一个应用(例如：ms-client)作为服务的调用方，创建方法请参考[微服务应用创建](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/2-%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BA%94%E7%94%A8%E5%88%9B%E5%BB%BA/manual.md)。
#### 2：对创建好的应用进行配置，配置方法请参考[基础工程创建与配置](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/3-%E5%9F%BA%E7%A1%80%E5%B7%A5%E7%A8%8B%E5%88%9B%E5%BB%BA%E4%B8%8E%E9%85%8D%E7%BD%AE/manual.md)。
#### 3：注解调用，通过@RemoteCall将需要调用的应用注解进来
```
@RemoteCall("ms-middle@c87e2267-1001-4c70-bb2a-ab41f3b81aa3")
public interface IMsMiddleService {
    @ApiOperation(value = "获取随机数字", response = String.class)
    public String getDigit();
}
```
#### 4：注解的参数设置请参考[接口开发，注解使用](http://git.yonyou.com/mwclient/ms-docs/blob/develop/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%B9%B3%E5%8F%B0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/%E4%B8%80%EF%BC%9A%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/4-%E6%8E%A5%E5%8F%A3%E5%BC%80%E5%8F%91%E3%80%81%E6%B3%A8%E8%A7%A3%E4%BD%BF%E7%94%A8/manual.md)。