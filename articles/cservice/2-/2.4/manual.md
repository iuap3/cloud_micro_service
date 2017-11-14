# 微服务接口开发，注解使用

描述：

## 接口开发
用例中的test-server为服务提供方，可以定义接口，并实现（接口可以抽取到服务提供方和调用方共同依赖的工程中，详细步骤可以参考demo工程pt-pubapi依赖相关文档），接口的定义和实现可以参考示例工程中pt-pubapi的IMsMiddleService和pt-server的MsMiddleService

**1：IMsMiddleService**

```
@RemoteCall("ms-middle@c87e2267-1001-4c70-bb2a-ab41f3b81aa3")
public interface IMsMiddleService {
    @ApiOperation(value = "获取随机数字", response = String.class)
    public String getDigit();
}

```

**2：MsMiddleService**

```
@Service
public class MsMiddleService implements IMsMiddleService{
    @Override
    public String getDigit() {
        return UUID.randomUUID().toString();
    }
}
```

## 注解使用

**1：@RemoteCall**

- @RemoteCall有三个属性：value，alias，version
- value的使用有两种方式：一种是直接在value属性上填写应用的ID，另一种方式是在value属性上填写应用编码加租户管理员的id即appcode@providerid
- alias的作用是注册服务的别名.当注册的服务没有指定名称时，可以使用别名的方式获取到注册的服务。
- @RemoteCall可以直接在下面红圈内复制：

![](biaoshi.jpg)

**2: @ApiOperation**

- @ApiOperation有一个属性：value
- 方法的别名：value

**3: @ApiParam**

- @ApiParam有5个属性：name，required，exampleValue，moreRestrictions，description
- 请求参数的名称：name，是否必须：required，示例值：exampleValue，更多限制：moreRestrictions，描述：description

**4: @ApiReturnValue**

- @ApiReturnValue有三个属性：name，description，exampleValue
- 响应参数的名称：name，示例值：exampleValue，，描述：description

**附：@ApiParam和@ApiReturnValue请使用yonyou自身开发的包**

- import com.yonyou.cloud.mwclient.servmeta.annotation.ApiParam;
- import com.yonyou.cloud.mwclient.servmeta.annotation.ApiReturnValue;

# 常见问题

## 常见问题1：对应的应用不存在! 

**出现此问题的原因:**

- 此用户下没有相应的应用
- value的值有误即应用编码或者租户id输入错误
