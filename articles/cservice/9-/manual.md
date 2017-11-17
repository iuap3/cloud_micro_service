# EOS异步调用组件使用 #
### Apollo简介
- 数据一致性服务是一种分布式事务在云上的解决方案
- 基于三阶段提交的事务
- 解决海量数据、高并发场景下应用按照业务纵向拆分后数据最终一致性的问题。
- 如果说单数据源应用的两阶段提交是在数据库层面的话，那么TCC就是在应用层面的两阶段提交。
- 提供账号下失败事务的管理，调用失败服务定位及调用过程跟踪。
- 在框架层面保证业务方法的幂等性，让开发者更多的关注核心业务实现。

### Apollo使用方法
- apollo数据库:
<pre>
    CREATE DATABASE `tcc` DEFAULT CHARACTER SET utf8;
    
    USE `tcc`;
    
    CREATE TABLE `tcc_transaction` (
      `TRANSACTION_ID` varchar(36) NOT NULL COMMENT '事务ID(UUID)',
      `DOMAIN` varchar(100) DEFAULT NULL COMMENT '项目名称/业务模块',
      `GLOBAL_TX_ID` varbinary(32) NOT NULL COMMENT '全局事务ID',
      `BRANCH_QUALIFIER` varbinary(32) NOT NULL COMMENT '分支事务ID',
      `CONTENT` varbinary(8000) DEFAULT NULL COMMENT '事务上下文内容',
      `STATUS` int(11) DEFAULT NULL COMMENT '事务状态(1:TRYING:未提交, 2:CONFIRMING:确认中, 3:CANCELLING:取消/回滚中)',
      `TRANSACTION_TYPE` int(11) DEFAULT NULL COMMENT '事务类型:(1:ROOT:根事务, 2:BRANCH:分支事务)',
      `RETRIED_COUNT` int(11) DEFAULT NULL COMMENT '重试次数',
      `CREATE_TIME` datetime DEFAULT NULL COMMENT '创建时间',
      `LAST_UPDATE_TIME` datetime DEFAULT NULL COMMENT '最后更新时间',
      `VERSION` int(11) DEFAULT NULL COMMENT '版本',
      PRIMARY KEY (`TRANSACTION_ID`),
      UNIQUE KEY `UX_TX_BQ` (`GLOBAL_TX_ID`,`BRANCH_QUALIFIER`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
</pre>

- 事务发起方和事务调用方业务数据库加入tcc_idempotent表:
<pre>
CREATE TABLE `tcc_idempotent` (
  `domain` varchar(100) DEFAULT NULL COMMENT '业务模块名称',
  `gtxid` varbinary(32) NOT NULL COMMENT '全局事务ID',
  `btxid` varbinary(32) NOT NULL COMMENT '分支事务ID',
  `method` varchar(100) DEFAULT NULL COMMENT '方法名称',
  `status` int(2) DEFAULT NULL COMMENT '事务阶段(try1/confirm2/cancel3)',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  UNIQUE KEY `UQ_GTXID_BTXID` (`gtxid`,`btxid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='tcc幂等性,全局事务ID和分支事务ID联合唯一索引.' 
</pre>

- 事务发起方和事务调用方pom依赖:
<pre>
&lt;dependency&gt;
    &lt;groupId&gt;com.yonyou.cloud.middleware&lt;/groupId&gt;
    &lt;artifactId&gt;apollo-spring&lt;/artifactId&gt;
	&lt;version&gt;2.3.1-SNAPSHOT&lt;/version&gt;
&lt;/dependency&gt;
</pre>

- 在src/main/resources目录下新建applicationContext-apollo.xml文件:
	- 其中sampleDataSource代表业务数据库
	- 以下所有domain的配置的值得方式必须是:AppCode-租户ID-环境profile的形式, 如:ms-server-c87e2267-1001-4c70-bb2a-ab41f3b81aa3-online
<pre>
        &lt;bean id="sampleDataSource" class="org.apache.tomcat.jdbc.pool.DataSource"
              destroy-method="close" lazy-init="false"&gt;
            &lt;property name="driverClassName" value="#{jdbc['jdbc.driverClassName']}"/&gt;
            &lt;property name="url" value="#{jdbc['jdbc.url']}"/&gt;
            &lt;property name="username" value="#{jdbc['jdbc.username']}"/&gt;
            &lt;property name="password" value="#{jdbc['jdbc.password']}"/&gt;
        &lt;/bean&gt;
        
        &lt;bean id="tccDataSource" class="org.apache.tomcat.jdbc.pool.DataSource"
              destroy-method="close" lazy-init="false"&gt;
            &lt;property name="driverClassName" value="#{jdbc['jdbc.driverClassName']}"/&gt;
            &lt;property name="url" value="#{jdbc['tcc.jdbc.url']}"/&gt;
            &lt;property name="username" value="#{jdbc['jdbc.username']}"/&gt;
            &lt;property name="password" value="#{jdbc['jdbc.password']}"/&gt;
        &lt;/bean&gt;
        
        &lt;bean id="transactionManager"
              class="org.springframework.jdbc.datasource.DataSourceTransactionManager"&gt;
            &lt;property name="dataSource" ref="sampleDataSource"/&gt;
        &lt;/bean&gt;
        
        &lt;!-- 幂等性切面 begin --&gt;
        &lt;bean id="jdbcXidRepository" class="com.yonyou.cloud.apollo.spring.repository.SpringJdbcXidRepository"&gt;
            &lt;property name="dataSource" ref="sampleDataSource"/&gt;
            &lt;property name="domain" value="SAMPLE"/&gt;
        &lt;/bean&gt;
        
        &lt;bean id="configurableIdempotentAspect" class="com.yonyou.cloud.apollo.spring.ConfigurableIdempotentAspect"
              init-method="init"&gt;
            &lt;property name="xidRepository" ref="jdbcXidRepository"/&gt;
        &lt;/bean&gt;
        &lt;!-- 幂等性切面 end --&gt;
        
        &lt;bean class="com.yonyou.cloud.apollo.spring.recover.DefaultRecoverConfig"&gt;
            &lt;property name="maxRetryCount" value="5"/&gt;
            &lt;property name="recoverDuration" value="60"/&gt;
            &lt;property name="cronExpression" value="0/30 * * * * ?"/&gt;
            &lt;!-- 单机应用可以不配置以下分布式锁, 若配置则选择redis或zookeeper其中一个即可, 两个都配置则优先使用redis.
            &lt;property name="zkAddr" value="localhost:2181"/&gt; --&gt;
            &lt;property name="redisAddr" value="localhost:6379"/&gt;
            &lt;property name="domain" value="SAMPLE"/&gt;
        &lt;/bean&gt;
        
        &lt;bean id="transactionRepository" class="com.yonyou.cloud.apollo.spring.repository.SpringJdbcTransactionRepository"&gt;
            &lt;property name="dataSource" ref="tccDataSource"/&gt;
            &lt;property name="domain" value="SAMPLE"/&gt;
        &lt;/bean&gt;
</pre>

- 在web.xml中加入新建的applicationContext-eos.xml和eos.xml的引入:
<pre>
&lt;context-param&gt;
	&lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;
	&lt;param-value&gt;
	        classpath*:/applicationContext.xml,
	        classpath*:/applicationContext-apollo.xml,
	        classpath:tcc-transaction.xml
	&lt;/param-value&gt;
&lt;/context-param&gt;
</pre>

- 主事务发起方的代码逻辑示例:
	- 当主/子事务的发起调用的任意阶段方法失败时会调用主/子事务的cancelMethod进行事务回滚, 所以主/子事务的cancelMethod要根据DB的数据状态判断主/子事务是否执行成功.
<pre>
        // 主事务-发起调用
        @Transactional
        @Compensable(confirmMethod = "confirmDelUserInfo", cancelMethod = "cancelDelUserInfo", transactionContextEditor = MethodTransactionContextEditor.class)
        public void delUserInfo(TransactionContext transactionContext, long userId) {
        	delUserHonour(transactionContext, userId); // 子事务调用
        	// ... 其他子事务调用
        }
        
        // 主事务-确认提交整体事务
        @Transactional
        @Idempotent
        public void confirmDelUserInfo(TransactionContext transactionContext, long userId) {...}
        
        // 主事务-确认回滚整体事务
        @Transactional
        @Idempotent
        public void cancelDelUserInfo(TransactionContext transactionContext, long userId) {...}
        
        // 子事务RPC调用-嵌套在主事务调用逻辑中
        @Compensable(propagation = Propagation.SUPPORTS, confirmMethod = "delUserHonour", cancelMethod = "delUserHonour", transactionContextEditor = MethodTransactionContextEditor.class)
        @Idempotent
        @Transactional
        public void delUserHonour(TransactionContext transactionContext, long userId) {...}
</pre>

- 子事务方被调用方的代码逻辑示例:
<pre>
    // 子事务调用入口
    @Compensable(confirmMethod = "confirmDelUserHonour", cancelMethod = "cancelDelUserHonour", transactionContextEditor = MethodTransactionContextEditor.class)
    @Transactional
    public void delUserHonour(TransactionContext transactionContext, long userId) {...}
    
    // 子事务-提交
    @Idempotent
    @Transactional
    public void confirmDelUserHonour(TransactionContext transactionContext, long userId) {...}
    
    // 子事务-回滚
    @Idempotent
    @Transactional
    public void cancelDelUserHonour(TransactionContext transactionContext, long userId) {...}
</pre>

- 示例代码参见:
	-  http://git.yonyou.com/mwclient/demo/tree/coeus/apollo-sample