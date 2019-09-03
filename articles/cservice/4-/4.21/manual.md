# 应用合并

## 合并工程pom添加依赖
```xml
<dependency>
    <groupId>com.yonyou.cloud.middleware</groupId>
    <artifactId>iris-registry-support</artifactId>
    <version>5.2.1-RELEASE</version>
</dependency>
```

## 新增parent配置文件
文件名为合并前的appCode，后缀为parent，多个应用合并应有多个parent文件
resources下新建META-INF文件夹用于存放parent文件
创建${appCode}.parent文件，比如
rpc-client-511.parent
```
  parent=client-provider

  // 如果需要的话
  //租户ID
  nameSpace=xxxxxxxxxx
  //access key
  ak=xxxxxxxxxx
  as=xxxxxxxxxx

```
