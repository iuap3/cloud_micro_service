# 服务限流与降级

## 限流参数介绍
### 限流主体
限流服务的主要功能是对限流主体进行流量控制，在限流服务里是以限流唯一标识为限流主体，在限流服务里面配置里面以desc字段表示。

限流唯一标识可以是一个RPC API标识，也可以是用户自定义的埋点字符串，只要保证在一个应用里面是唯一的即可。

### 限流类型
限流类型分为QPS和线程两种。

QPS限流是最直观的限流方式，以流量总量的为基准来做限流控制。

线程限流是以并发执行为度量方式来进行限流控制，限制同一服务或者代码块能够同时并发执行的线程数量。

### 限流阈值
限流阈值分别对限流类型进行限流值的设定，超过限流阈值之后，限流服务会对后续的请求进行拒绝，避免过载。

## 限流设置
在开发者中心的服务提供者应用详情页面，微服务页签下，可以针对不同的服务API进行限流设置，如下图所示:
![](https://ws2.sinaimg.cn/large/006tNc79ly1fknpag3xhtj31an0nyh1a.jpg)

![](https://ws1.sinaimg.cn/large/006tNc79ly1fknpax7m56j31an0nnn6f.jpg)

![](https://ws4.sinaimg.cn/large/006tNc79ly1fknpbbwp3qj31an0nln6d.jpg)

![](https://ws1.sinaimg.cn/large/006tNc79ly1fknpbp9ejej31540netdv.jpg)

从以上路径，找到服务提供的应用，即可方便的对特定的接口方法设置限流阈值。

## 限流配置查看
上节指定的限流配置，实际会同步到配置中心的限流配置文件中，然后下发到应用实例上，进行实际的限流控制。
我们可以在配置中心查看具体的限流配置文件，如下所示：
![](https://ws4.sinaimg.cn/large/006tNc79ly1fknvr8d1jlj31an0o877w.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79ly1fknvrs8ev6j31ak0ot0wl.jpg)

配置的结构包含应用信息、限流唯一标识和限流阈值信息，通过配置文件可以查看我们的限流配置是否生效。还可以在这里配置我们自定义的限流信息。

```
{
  "appName": "micro-service",
  "strategy": [
    {
      "qps": "30",
      "threadCount": "2",
      "desc": "com.yonyoucloud.pt.server.service.IMSService#getRandomText(java.lang.Integer)"
    },
    {
      "qps": "4",
      "threadCount": "0",
      "desc": "com.yonyoucloud.pt.server.service.IptService#ptMethod(java.lang.Integer,java.lang.Integer,java.util.List)"
    },
    {
      "qps": "21",
      "threadCount": "42",
      "desc": "com.yonyoucloud.pt.server.service.IMSService#getList(java.lang.String,java.lang.Class,java.lang.Class,java.util.List)"
    }
  ]
}
```

## 自定义限流
除了对RPC接口进行限流之外，可能我们还有一些非RPC接口或者代码块需要设置限流，这个时候就需要使用到限流服务的API来进行自定义限流设置。

在引入了mwclient之后，会同时加在我们的限流服务客户端yyeye，这个时候我们可以在代码中使用限流API来进行自定义限流。代码如下：
```
Entry entry = null;
		try {
			entry = Entry.enter("user-define-flow-control");
		} catch (SentinelBlockedException se) {
			logger.error("flow control");
		} catch (Exception e) {
			logger.error("exception happens");
		} finally {
			if (entry != null) {
				entry.exit();
			}
		}
```
通过以上代码，我们就在代码中引入了一个“user-define-flow-control”的自定义限流点，这个时候，我们就可以编辑配置文件，对自定义的限流点，进行限流的设置，如下所示：

```
{
  "appName": "micro-service",
  "strategy": [
    {
      "qps": "30",
      "threadCount": "2",
      "desc": "com.yonyoucloud.pt.server.service.IMSService#getRandomText(java.lang.Integer)"
    },
    {
      "qps": "4",
      "threadCount": "0",
      "desc": "com.yonyoucloud.pt.server.service.IptService#ptMethod(java.lang.Integer,java.lang.Integer,java.util.List)"
    },
    {
      "qps": "21",
      "threadCount": "42",
      "desc": "com.yonyoucloud.pt.server.service.IMSService#getList(java.lang.String,java.lang.Class,java.lang.Class,java.util.List)"
    },
    {
      "qps": "30",
      "threadCount": "2",
      "desc": "user-define-flow-control"
    }
  ]
}
```
通过编辑配置文件，我们就可以对自定义限流点进行限流的设置。通过使用API，我们也可以为任何的框架来适配限流功能。

## 服务熔断
限流服务的自动降级功能在服务发生大面积错误时，自动开启服务熔断的功能。发生服务熔断时，客户端会屏蔽掉服务端的调用，进行服务调用的failover，并且会对服务做持续探测，在服务端恢复正常情况下，会恢复服务的调用。
目前服务熔断功能会在RPC框架中自动开启，目前熔断功能还不能进行定制，在下个版本中会支持用户自定义开启和关闭熔断。

## 常见问题
### 限流不生效
#### 唯一标识冲突
此时可能是启用了自定义限流，和系统自动获取的RPC唯一标识冲突，此时我们可以查看对应的限流配置文件，查看时候冲突。
#### 配置中心延迟
由于限流配置的下发依赖与配置中心，由于配置文件更新的延迟，我们设置的限流策略，可能会有一定的生效延时，此时只需要等待一会儿即可生效。


