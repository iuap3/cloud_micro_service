# 熔断

## 简介

* 熔断机制是将耗时的方法或代码段保护起来，与当前执行上下文隔离开来执行，以达到保护当前调用的目的，避免因耗时操作造成服务堵塞
* 用户请求某一服务之后，先经过熔断器，此时如果熔断器的状态是打开（跳起），则说明已经熔断，这时将直接进行降级处理，不会继续将请求发到线程池。熔断器相当于在线程池之前的一层屏障。
* 熔断规则可在控制台进行配置（详见：用户指南-熔断），也可以@Fallback注解方式配置，控制台规则优先级高于注解。

## 开发过程

熔断为服务治理平台SDK自带功能，工程搭建方式与RPC和异步服务开发没有差别。在服务调用端新建熔断处理类，实现服务接口。通过@Fallback注解声明本实现类为熔断处理类，并可在@Fallback注解中配置熔断规则:
  @Fallback(window=10, threshold=40, ratio=10, timeout=2000)
  public class FallbackProviderService implements IProviderService {

  	@Override
  	public String testString(String sleep, String error) {
  		// TODO Auto-generated method stub
  		return "fallbackclass";
  	}

  	@Override
  	public String asyn(String name) {
  		// TODO Auto-generated method stub
  		return null;
  	}

  	@Override
  	public User getUserInfo(String sleep, String error) {
  		User u = new User();
  		u.setId("fallbackclass");
  		u.setAge(-10);
  		return u;
  	}

  }

@Fallback注解参数说明：

参数名称|默认值|说明
:---:|:---:|:---
timeout|1000|超时时间，单位：ms，当调用执行时间超过此时间，执行降级逻辑
threshold|20|熔断触发的最小个数/10s
ratio|20|错误率，单位：%，失败率达到多少百分比后熔断
window|10|时间窗口，单位s，熔断多少秒后去变为半开状态，放过一次请求去尝试下游服务的状态，如果成功则关闭熔断，如果失败则继续开启

在熔断实现类中实现的方法，为调用该方法触发熔断时的降级逻辑：

    @Override
    public User getUserInfo(String sleep, String error) {
      User u = new User();
      u.setId("fallbackclass");
      u.setAge(-10);
      return u;
    }

如构造一个虚拟的entity并返回。
