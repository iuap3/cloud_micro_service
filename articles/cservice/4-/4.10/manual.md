# 熔断机制

## 简介

* 熔断机制是将耗时的方法或代码段保护起来，与当前执行上下文隔离开来执行，通常可用线程池或信号量进行隔离，已达到保护当前调用的目的，避免因耗时操作造成服务堵塞
* 熔断器是位于线程池之前的组件。用户请求某一服务之后，先经过熔断器，此时如果熔断器的状态是打开（跳起），则说明已经熔断，这时将直接进行降级处理，不会继续将请求发到线程池。熔断器相当于在线程池之前的一层屏障。每个熔断器默认维护10个bucket ，
每秒创建一个bucket ，每个blucket记录成功,失败,超时,拒绝的次数等熔断器相关信息。当有新的bucket被创建时，最旧的bucket会被抛弃

## 熔断器状态

* Closed：熔断器关闭状态，调用失败次数积累，到了阈值（或一定比例）则启动熔断机制
* Open：熔断器打开状态，此时对下游的调用都内部直接返回错误，不走网络，但设计了一个时钟选项，默认的时钟达到了一定时间（这个时间一般设置成平均故障处理时间，也就是MTTR），到了这个时间，进入半熔断状态
* Half-Open：半熔断状态，允许定量的服务请求，如果调用都成功（或一定比例）则认为恢复了，关闭熔断器，否则认为还没好，又回到熔断器打开状态

## 熔断器隔离策略
* 1）信号量隔离：使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，请求来先判断计数器的数值，若超过设置的最大线程个数则丢弃改类型的新请求，
   若不超过则执行计数操作请求来计数器+1，请求返回计数器-1。这种方式是严格的控制线程且立即返回模式，无法应对突发流量（流量洪峰来临时，处理的线程超过数量，其他的请求会直接返回，不继续去请求依赖的服务）
* 2）线程池隔离
   使用一个线程池来处理耗时请求，设置任务返回处理超时时间，堆积的请求加入线程池队列。这种方式需要为每个依赖的服务申请线程池，有一定的资源消耗，好处是可以应对突发流量（流量洪峰来临时，处理不完可将数据存储到线程池队里慢慢处理）

## 降级处理
+ 1）当熔断器状态open时，直接执行降级逻辑
+ 2）当执行异常时，执行降级逻辑

## 熔断器相关参数设置
```
参数	                                             作用                 	                            备注
errorThresholdPercentage	                         失败率达到多少百分比后熔断	                        默认值：50
                                                     主要根据依赖重要性进行调整
forceClosed	                                         是否强制关闭熔断，                                 如果是强依赖，应该设置为 true
requestVolumeThreshold	                             熔断触发的最小个数/10s	                            默认值：20
sleepWindowInMilliseconds	                         熔断多少秒后去尝试请求	                            默认值：5000
commandKey	 	                                     熔断器名称                                         默认值：当前执行方法名
coreSize	                                         线程池coreSize	                                    默认值：10，线程池隔离有效
execution.isolation.semaphore.maxConcurrentRequests  信号量最大并发度	                                SEMAPHORE模式有效，默认值：10
execution.isolation.strategy	                     隔离策略，有THREAD和SEMAPHORE	                    默认使用THREAD模式，以下几种可以使用SEMAPHORE模式： 
	                                                                                                    只想控制并发度
	                                                                                                    外部的方法已经做了线程隔离
	                                                                                                    调用的是本地方法或者可靠度非常高、耗时特别小的方法
execution.isolation.thread.interruptOnTimeout	     是否打开超时线程中断	                            THREAD模式有效
execution.isolation.thread.timeoutInMilliseconds     超时时间	                                        默认值：1000
                                                                                                        在THREAD模式下，达到超时时间，可以中断
                                                                                                        在SEMAPHORE模式下，会等待执行完成后，再去判断是否超时
execution.timeout.enabled	                         是否打开超时	 
fallback.isolation.semaphore.maxConcurrentRequests   fallback最大并发度	                                默认值：10
groupKey	                                         表示所属的group，一个group共用线程池	            默认值：getClass().getSimpleName();
maxQueueSize	                                     请求等待队列	                                    默认值：-1
                                                                                                        如果使用正数，队列将从SynchronizeQueue改为LinkedBlockingQueue
hystrix.command.default.metrics.                     设置统计的时间窗口值的，毫秒值                     circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，则rolling window会被分成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
rollingStats.timeInMilliseconds                      	                                
hystrix.command.default.metrics.                     设置一个rolling window被划分的数量
rollingStats.numBuckets 	 
hystrix.commanddefaultmetricshealthSnapshot.
intervalInMilliseconds 	                             记录health 快照（用来统计成功和错误绿）的间隔，   默认500ms
```
## 注意事项
目前只支持信号量隔离策略，使用线程隔离策略会存在threadlocal变量问题，因为被隔离的方法在新的线程中调用，如果被隔离的方法中通过threadlocal来传递参数或上下文的话
会导致被调用方法隔离前后获取threadlocal不对
## 示例
//业务接口
```
public interface ILogService<T, M> {
	public T log(T s);
	public M log(T s,M m);
}
```
//降级逻辑类，实现业务逻辑接口
```
public class Fallback implements ILogService<String, Integer>{

	@Override
	public String log(String s) {
		return "error1";
	}

	@Override
	public Integer log(String s, Integer m) {
		return -m * 2;
	}
}
```
//业务实现类，模拟耗时操作
```
public class LogServiceImpl<T, M> implements ILogService<T, M> {
       @Override
	public T log(T s) {
		return s;
	}
	@Override
	public M log(T s, M m) {
		return m;
	}

}

```
 //业务对象
 ```
 ILogService<Integer> logIntService = new LogServiceImpl<Integer>();
//根据业务对象获取业务对象代理
//降级逻辑，降级逻辑class与业务实现有相同的签名方案，当业务方法出现熔断时，会执行降级逻辑class的相同签名的方法
ILogService<Integer> logIntServiceProxy = 
				 CirCuitBreakerProxyFactory.<ILogService<Integer>>createCircuitBreakerProxy().createProxy(logIntService, Fallback.cass,

//熔断器相关参数设置，递延式参数设置
CCSetter.withGroup("groupKey1").andCommandKey("commandkey1").andThreadPoolKey("threadpoolkey1").andCircuitBreakerForceOpen(true));

// 通过代理对象执行隔离逻辑，即 执行log方法时将被熔断器隔离执行
logIntServiceProxy.log("one");

```
