# 微服务工程创建及配置

描述：

## 创建基础Maven工程

**1：创建基础的Maven结构的spring依赖的Web工程**

**2：具体请参考实例提供的test-server工程**


## 配置修改

### maven依赖(pom文件)
1. 增加mwclient的依赖

```
<dependency>
    <groupId>com.yonyou.cloud.middleware</groupId>
    <artifactId>mwclient</artifactId>
    <version>2.1.1-RELEASE</version>
</dependency>
```

 2.artifactId(影响访问路径)，finalname要与应用编码保持一致

### web.xml配置
增加微服务框架需要的listener，对应类为com.yonyou.cloud.mwclient.MwClientLoader

    <listener>
        <listener-class>com.yonyou.cloud.mwclient.MwClientLoader</listener-class>
    </listener>


### 属性文件配置

**application.properties**

1. access.key和access.secret与申请的Access Key中内容保持一致
2. spring.profiles.active为应用的环境(开发：dev，测试：test，灰度：gray，生产：online)
3. spring.application.name为应用的编码

# 常见问题

## 常见问题1：服务验证失败.请确认accessKey 和application.name的正确性! 

**出现此问题的原因:**

- 输入的access.key和access.secret不一致或者错误
- 此AccessKey已被停用或者删除
- application.name即应用编码输入错误
- 此用户下没有应用编码为application.name的应用