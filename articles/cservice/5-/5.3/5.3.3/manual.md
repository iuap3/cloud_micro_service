# 示例工程下载

## 示例下载

可以从微服务治理平台官方网站下载上述文档中提及的示例工程，对应的下载地址如下：

[https://developer.yonyoucloud.com/download/microservice/tcc-test.zip](https://developer.yonyoucloud.com/download/microservice/tcc-test.zip "官方示例5.2.1-RELEASE")

示例下载后，需要修改对应配置文件中的AccessKey和应用编码、租户ID等信息，编译maven工程后运行。

## 常见问题

**1：工程导入错误**

请注意，提供的示例包中包含11个maven工程，tcc-test为其余10个工程的父工程，导入工程后，需要按照配置进行修改后，进行mvn clean install后，再运行示例。

**2：工程编译出错**

示例工程采用Maven进行构建，请保证开发机器包含Java和Maven的环境变量设置。

注意：微服务治理平台提供的SDK的仓库地址为maven.yonyou.com,请保证网络连通。
