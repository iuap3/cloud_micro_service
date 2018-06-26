# 应用依赖

应用依赖是指本服务与其他微服务的依赖关系，可以在【服务管理】进入你的服务打开【微服务】页签下面的【依赖关系】进入查看

![](images/dependency.png "应用依赖关系")


**箭头指向方向为被依赖方**

如图所示我们进入的是rpc-provider应用，rpc-provider依赖rpc-server,而rpc-provider被rpc-client依赖