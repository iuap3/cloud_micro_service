#配置中心sdk

与配置中心服务器端配套的功能jar包，应用引入该jar包并做相应配置就能通过配置中心管理引用的配置文件。实现实时更新，统一分发，多种环境配置文件管理等功能。

##功能简介
为用户使用配置中心平台的管理功能提供类库，通过SDK封装了应用引入配置中心管理功能所需要做的复杂操作以及配置。引入了配置中心sdk以后使得应用在运行态能够感知配置文件在配置中心的变化，并做到实时响应快速更新变化的配置文件或者配置项。同时该sdk还能够使配置中心感知到应用的运行状态，使用户能够从配置中心的相关页面了解到应用的运行情况，以及单个配置文件在应用中的应用状态。

<img src="/img/confsdk1.png"/>
##主要特征

- 简单的引入方式，单jar包依赖，提供注解式编程和无侵入引入方式
- 本地文件与线上文件的定时对比，保证配置文件一致性
- 强兼容性，通过配置能够开关配置中心得到使用，快速切换使用本地配置文件或者线上配置文件

##接入步骤

###注解式接入

1.@DisconfFile注解，filename值确定sdk管理的配置文件名，copy2TargetDirPath决定配置文件的下载位置

2.@DisconfFileItem,userName指定该配置文件的配置项名称，associateField指向关联的成员变量。


	
	@DisconfFile(filename="simple.properties",copy2TargetDirPath="/etc/config/group1")
	public class DisConfBean {
		
		private static final Logger LOG = LoggerFactory.getLogger(DisConfBean.class);
	
		private String userName;
		
		private String userCode;
		
		private String memo;
	
		@DisconfFileItem(name = "userName", associateField = "userName")
		public String getUserName() {
			return userName;
		}
	
		public void setUserName(String userName) {
			this.userName = userName;
			LOG.info("method setUserName has been called！");
		}
	
		@DisconfFileItem(name = "userCode", associateField = "userCode")
		public String getUserCode() {
			return userCode;
		}
	
		public void setUserCode(String userCode) {
			this.userCode = userCode;
			LOG.info("method setUserCode has been called！");
		}
	
		@DisconfFileItem(name = "memo", associateField = "memo")
		public String getMemo() {
			return memo;
		}
	
		public void setMemo(String memo) {
			this.memo = memo;
			LOG.info("method setMemo has been called！");
		}

3.@DisconfUpdateService注解标识配置更新时需要进行更新的服务,需要指定它影响的配置数据，
可以是配置文件或者是配置项。

	@Service
	@DisconfUpdateService(confFileKeys = {"application.properties"})
	public class DisConfigUpdateCallback implements IDisconfUpdate {
		
		private static final Logger LOG = LoggerFactory.getLogger(DisConfigUpdateCallback.class);
	
	    public void reload(Object data) throws Exception {
			LOG.info("new file content is :" + data);
	    	PropertyUtil.reload();
	    	LOG.info("DisConfigUpdateCallback called! application.properties has changed!");
	    	
	    }
	}
<br>

	@Service
	@DisconfUpdateService(itemKeys = {"testitem"})
	public class DisconfItemUpdateCallback implements IDisconfUpdate {
		
		@Autowired
		private DisconfItemBean item;
		
		private static final Logger LOG = LoggerFactory.getLogger(DisconfItemUpdateCallback.class);
	
	    public void reload(Object data) throws Exception {
	    	LOG.info("DisconfItemUpdateCallback called! testitem has changed!");
	    	System.out.println(item.getTestitem());
	    }
	}

4.@DisconfItem注解，将配置单项单项注入成员变量
	
	public class DisconfItemBean {
		
		private final Logger logger = LoggerFactory.getLogger(getClass());
		
		@Value(value = "2000d")
		private Double testitem;
		
	    @DisconfItem(key = "testitem")
		public Double getTestitem() {
			return testitem;
		}
	
		public void setTestitem(Double testitem) {
			this.testitem = testitem;
			logger.info("dynamic property testitem in disconf has changed! new value is {}!", testitem);
		}
	}

###无侵入接入

	
	    <!-- 使用托管方式的disconf配置(无代码侵入, 配置更改会自动reload)-->
	    <bean id="disconfPropFactoryBean1" class="com.yonyou.iuap.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
	        <!--分组ID，用来隔离不同租户的应用和配置，对应开发者中心的providerID-->
	        <property name="group" value="group1"></property>
	        <property name="locations">
	            <list>
	                <value>classpath:application.properties</value>
	            </list>
	        </property>
	    </bean>
	   
	
	   <bean id="disconfPropertyConfigurer" class="com.yonyou.iuap.disconf.client.addons.properties.ReloadingPropertyPlaceholderConfigurer">
	        <property name="ignoreResourceNotFound" value="true"/>
	        <property name="ignoreUnresolvablePlaceholders" value="true"/>
	        <property name="propertiesArray">
	            <list>
	                <ref bean="disconfPropFactoryBean1"/>
	            </list>
	        </property>
	        <property name="systemPropertiesMode" value="2"></property>
	    </bean>
		
	
###其他

1.disconf.startup.interrupt	读取远程配置文件失败后是否停止容器

2.disconf.value.compare 本地文件是否与远程文件做内容比对

3.disconf.value.compare.robin 文件内容比对的时间间隔

4.disconf.enable.remote.conf 是否启用配置中心

5.disconf.watchable.remote.conf 是否监控远程文件

    <bean id="group1" class="com.yonyou.iuap.disconf.client.config.CustomClientConfig">
		<property name="groupId" value="35568e76-1ef1-4d77-b5cf-8fb66d2c8002"></property>
		<property name="master" value="true"></property>
        <property name="pros">
            <props>
            	<prop key="disconf.startup.interrupt">true</prop>
            	<prop key="disconf.value.compare">true</prop>
            	<prop key="disconf.value.compare.robin">3</prop>
                <prop key="disconf.enable.remote.conf">true</prop>
                <prop key="disconf.watchable.remote.conf">true</prop>
                <prop key="disconf.conf_server_host">http://192.168.32.63/confcenter</prop>
                <prop key="disconf.app">disconf-test2</prop>
                <prop key="disconf.version">1_0_0_0</prop>
                <prop key="disconf.env">dev</prop>
                <prop key="disconf.ignore"></prop>
                <prop key="disconf.conf_server_url_retry_times">1</prop>
                <prop key="disconf.conf_server_url_retry_sleep_seconds">5</prop>
                <prop key="disconf.user_define_download_dir">/etc/config/group1</prop>
                <prop key="disconf.polling_interval">5</prop>
            </props>
        </property>
    </bean>
