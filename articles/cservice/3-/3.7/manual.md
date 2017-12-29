# 服务注册中ssl安全配置
## tomcat配置安全证书

把安全证书配置在server.xml文件中，在Connector节点中，需要指定安全证书的路径、使用秘钥库的类型及密码。具体配置如下:
```
<Connector port="8443" protocol="HTTP/1.1" maxThreads="150" SSLEnabled="true" scheme="https" secure="true" 
clientAuth="false" sslProtocol="TLS" keystoreFile="D:/devEvn/ssl/yyuap.keystore" keystoreType="pkcs12" eystorePass="111111" />
```
配置说明：
- SSLEnabled：是否启用ssl配置
- keystoreFile：证书的路径
- keystoreType：证书的类型
- eystorePass：证书的密码
- clientAuth：是否启用双向认证，true为启用双向认证，false为不启用。在RPC框架中使用注册中心时，不需要配置双向认证。
- scheme：访问服务所使用的协议，使用安全的连接，其值必须配置为https。
- SSLEnabled：配置tomcat是否支持SSL连接。
- secure：是否安全访问。使用SSL认证时，此值必须设置为true。

## 服务端的配置
- 服务端在启动时，需要在application.properties中配置是否启用ssl安全服务，如果需要启用的话，只需要配置server.ssl=true就可以了，默认情况下，是不启用ssl安全配置的。

## 客户端的配置
- 客户端在启动时的安全配置，和服务器端的配置是一样。