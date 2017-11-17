# EOS异步调用组件使用 #
### EOS简介
- 异步调用的方式解决数据一致性问题
- 业务系统调用事件中心提供的服务发送消息，事件中心接收到消息后将消息放入队列中。
- 事件中心监听器接收队列消息后，调用业务注册的url地址执行事件处理
- 调用成功后，删除队列中的消息。
- 调用失败后，间隔一段时间再次尝试发送。
- 尝试超过一定次数后，记录处理失败日志，删除消息。

### EOS使用方法
- 建立数据库, 并导入eos组件所需的数据库表:
<pre>
    /*Table structure for table `eos_mqerror` */
    
    CREATE TABLE `eos_mqerror` (
      `id` varchar(36) NOT NULL COMMENT '错误日志主键',
      `logtype` int(2) DEFAULT NULL COMMENT '日志类型(0:发送日志,1:接收日志)',
      `mqid` varchar(100) DEFAULT NULL COMMENT '消息id',
      `gtxid` varchar(36) DEFAULT NULL COMMENT '全局事务ID-UUID',
      `queue` varchar(100) DEFAULT NULL COMMENT '队列名称=应用编码-租户ID-环境',
      `mqcontent` varchar(1000) DEFAULT NULL COMMENT '消息内容',
      `errlog` varchar(3000) DEFAULT NULL COMMENT '错误日志消息',
      `ts` bigint(20) DEFAULT NULL COMMENT '当前时间',
      `requeue` int(2) DEFAULT '0' COMMENT '是否重新入队列,0否,1是',
      PRIMARY KEY (`id`),
      KEY `IDX_MQID` (`mqid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
    /*Table structure for table `eos_mqrecv` */
    
    CREATE TABLE `eos_mqrecv` (
      `id` varchar(36) NOT NULL COMMENT 'MQID-UUID-主键',
      `gtxid` varchar(36) DEFAULT NULL COMMENT '全局事务ID-UUID',
      `queue` varchar(100) DEFAULT NULL COMMENT '队列名称',
      `content` varchar(1000) DEFAULT NULL COMMENT '消息内容',
      `createtime` bigint(20) DEFAULT NULL COMMENT '创建时间',
      `updatetime` bigint(20) DEFAULT NULL COMMENT '更新时间',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
    /*Table structure for table `eos_mqsend` */
    
    CREATE TABLE `eos_mqsend` (
      `id` varchar(36) NOT NULL COMMENT '主键-UUID',
      `gtxid` varchar(36) DEFAULT NULL COMMENT '全局事务ID-UUID',
      `queue` varchar(100) DEFAULT NULL COMMENT '队列名称=应用编码-租户ID',
      `content` varchar(1000) DEFAULT NULL COMMENT '消息内容',
      `status` int(2) DEFAULT NULL COMMENT '发送状态(0待发送,1发送中,2发送成功)',
      `createtime` bigint(20) DEFAULT NULL COMMENT '创建时间',
      `updatetime` bigint(20) DEFAULT NULL COMMENT '更新时间',
      `retrycount` int(3) DEFAULT '0' COMMENT '重试次数',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
</pre>

- 在src/main/resources目录下新建applicationContext-eos.xml文件:
<pre>
    &lt;?xml version="1.0" encoding="UTF-8"?&gt;
    &lt;beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context" xmlns:jdbc="http://www.springframework.org/schema/jdbc"  
    	xmlns:rabbit="http://www.springframework.org/schema/rabbit" xmlns:aop="http://www.springframework.org/schema/aop" xmlns:jee="http://www.springframework.org/schema/jee" 
    	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jpa="http://www.springframework.org/schema/data/jpa" 
    	xsi:schemaLocation="
    		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
    		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
    		http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
    		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
    		http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit-1.4.xsd
    		http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
    		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd"
            default-lazy-init="true"&gt;
    
    	&lt;description&gt;EOS配置 &lt;/description&gt;
    
    	&lt;!-- 数据源配置, 使用Tomcat JDBC连接池 --&gt;
    	&lt;bean id="dataSource" class="org.apache.tomcat.jdbc.pool.DataSource"
    		destroy-method="close" lazy-init="false"&gt;
    		&lt;property name="driverClassName" value="${jdbc.driver}" /&gt;
    		&lt;property name="url" value="${jdbc.url}" /&gt;
    		&lt;property name="username" value="${jdbc.username}" /&gt;
    		&lt;property name="password" value="${jdbc.password}" /&gt;
    
    		&lt;property name="defaultAutoCommit" value="true" /&gt;
    		&lt;property name="maxActive" value="${jdbc.pool.maxActive}" /&gt;
    		&lt;property name="maxIdle" value="${jdbc.pool.maxIdle}" /&gt;
    		&lt;property name="minIdle" value="${jdbc.pool.minIdle}" /&gt;
    		&lt;property name="maxWait" value="${jdbc.pool.maxWait}" /&gt;
    		&lt;property name="minEvictableIdleTimeMillis" value="${jdbc.pool.minEvictableIdleTimeMillis}" /&gt;
    		&lt;property name="removeAbandoned" value="${jdbc.pool.removeAbandoned}" /&gt;
    		&lt;property name="removeAbandonedTimeout" value="${jdbc.pool.removeAbandonedTimeout}" /&gt;
    		&lt;property name="testWhileIdle" value="true" /&gt;
    		&lt;property name="validationQuery" value="select 1" /&gt;
    	&lt;/bean&gt;
    	
    	&lt;tx:annotation-driven/&gt;
    	
    	&lt;bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"&gt;
             	&lt;constructor-arg ref="dataSource"/&gt;
        	 &lt;/bean&gt;
         
        	&lt;bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"&gt;
    		&lt;property name="dataSource" ref="dataSource"/&gt;
    	&lt;/bean&gt;
    
    	&lt;bean id="asyncConfig" class="com.yonyou.cloud.config.AsyncConfig"&gt;
        		&lt;property name="jdbcTemplate" ref="jdbcTemplate"/&gt;
        		&lt;property name="transactionManager" ref="transactionManager"/&gt;
    		&lt;--客户端异步调用方设置为true,服务端被调用方设置为false,不设默认为true--&gt;
    		&lt;property name="enableMQSend" value="false"/&gt;
        	&lt;/bean&gt;
    
    &lt;/beans&gt;
</pre>

- 在web.xml中加入新建的applicationContext-eos.xml和eos.xml的引入:
<pre>
&lt;context-param&gt;
	&lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;
	&lt;param-value&gt;
        classpath*:/applicationContext.xml,
        classpath*:/applicationContext-eos.xml,
        classpath*:/eos.xml
	&lt;/param-value&gt;
&lt;/context-param&gt;
&lt;listener&gt;
	&lt;listener-class&gt;com.yonyou.cloud.mwclient.MwClientLoader&lt;/listener-class&gt;
&lt;/listener&gt;
</pre>

- 在pom.xml中引入对eos和mwclient的依赖:
<pre>
&lt;dependency&gt;
	&lt;groupId&gt;com.yonyou.cloud.middleware&lt;/groupId&gt;
	&lt;artifactId&gt;eos&lt;/artifactId&gt;
	&lt;version&gt;2.3.1-SNAPSHOT&lt;/version&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
	&lt;groupId&gt;com.yonyou.cloud.middleware&lt;/groupId&gt;
	&lt;artifactId&gt;mwclient&lt;/artifactId&gt;
	&lt;version&gt;2.3.1-SNAPSHOT&lt;/version&gt;
&lt;/dependency&gt;
</pre>

- 声明对外服务接口, 标注RemoteCall注解, 并在需要异步调用的方法上标注@Async注解(异步调用方法的返回值要求为null):
<pre>
    import com.yonyou.cloud.middleware.rpc.Async;
    import com.yonyou.cloud.middleware.rpc.RemoteCall;
    
    @RemoteCall("appcode@tenantId")
    public interface IAsyncService {
    	@Async
    	public void asyncInvoke(String param);
    
    }
</pre>


- 服务提供方对此接口进行实现后部署启动, 服务调用方就可以使用此接口发起调用了.
	- 服务调用方发起调用的异步方法要在Spring的本地事务中(使用@Transactional注解或使用xml方式配置事务), 使用AsyncConfig中的transactionManager进行事务管理.

<br/>

- 示例代码参见:
	- http://git.yonyou.com/mwclient/demo/tree/coeus/spring-web

